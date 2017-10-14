# Resque

[关于 resque](http://xiajian.github.io/2015/01/30/about-resque)

Resque 是一个用来执行后台任务的 gem，支持一次性的后台任务和周期性的后台任务 (以下称任务为 job)。

Resque 依赖于 redis。

启动一个 Resque worker：

    $ QUEUE=* bin/rake resque:work

`QUEUE=*` 表示这个 worker 支持所有类型的 queue。可以启动多个 resque worker，每个 worker 支持多种 queue。

每个 job 从属于一种 queue，当把 job 入队到 resque 中时，resque 会先从所有 worker 中筛选出支持此 job 属于的 queue 的 worker，从这些 worker 中选出一个空闲的，然后把它放到这个 worker 中执行，如果此时没有空闲的 worker，那么此 job 就会留在 resque 的等待列队中。每当 worker 变成空闲时，resque 就从等待列队中挑选合适的 job 交给 worker 执行。worker 之间是并行执行的。

所以，如果你有很多种 job，有些紧急度比较高，要优先执行，或者有些 job 比较耗时，如果只设置一个 worker，那些耗时的 job 就会阻塞后面一些需要紧急执行的 job。此时，建议多设置一些 worker，而且可以让不同的 worker 支持不同的 queue。比如一个 woker 专门用来执行耗时的 job，一个 worker 专门用来执行紧急的 job。

启动两个 worker，一个支持 `queue_a`，`queue_b` 队列，另一个支持 `queue_c`，`queue_d` 队列：

    $ QUEUE=queue_a,queue_b bin/rake resuqe:work
    $ QUEUE=queue_c,queue_d bin/rake resuqe:work

**Resque 的配置**

1. Resque 依赖 redis，所以你要为它配置好 redis 的地址和端口。我们把这些配置写在 config/resque.yml 中，然后在 initializer (initializers/resque.rb) 中读取这些配置，设置给 Resque.redis。

        # config/resque.yml
        development: localhost:6379
        test: localhost:6379
        staging: localhost:6379
        production: localhost:6379

        # initializers/resque.rb
        ...
        resque_config = YAML.load_file(rails_root + '/config/resque.yml')
        Resque.redis = resque_config[rails_env]

1. Redis 命名空间的隔离。如果运行了多个单独的 Resque 实例，比如在一台服务器上部署了多个 Rails App，每个 App 都有一个 Resque 实例，但这些 Resque 实例都依赖同一个 Redis 服务器，那么就可能需要使用命名空间让 Redis 隔离这个不同的 Resque 实例，从而避免彼此覆盖。该特性由 redis-namespace 库提供，Resque 默认使用该库在 Redis 服务器中隔离。

        Resque.redis.namespace = "project_name_#{Rails.env}"

1. 如果我们想用 Resque 执行一些周期性任务，比如每天自动更新数据库信息，那么我们要使用一个 resque-scheduler 的 gem，它是基于 resque 的，用来执行计划任务。要执行的任务写在 `config/resque_scheduler_env.yml` 中，我们在 initializer 中读取这个文件的配置，设置给 Resque.schedule。

        # config/resque_scheduler_development.yml
        update_podcast_info:
          cron: "0 0 * * *"
          class: UpdatePodcastJob
          queue: update_podcast
          args: ...
          description: "This job updates the episode for existing podcasts"

        # initializers/resque.rb
        require 'resque-scheduler'
        require 'resque/scheduler/server'

        ...
        Resque.schedule = YAML.load_file(Rails.root + "config/resque_scheduler_#{Rails.env}.yml")
    
   计划任务的配置，包括执行周期 (cron)、执行的 job (class)、属于的 queue (有点疑问，这个 queue 不是已经在 job 中指定了吗? 为什么这里还要再指定一次，难道可以通过这里的设置来改变原有的 queue?)、job 的参数 (args) 以及这个计划任务的描述 (description)。

最后，一个完整的 initializers/resque.rb：

    require 'resque'
    require 'resque-scheduler'
    require 'resque/scheduler/server'

    rails_root = ENV['RAILS_ROOT'] || File.dirname(__FILE__) + '/../..'
    rails_env = ENV['RAILS_ENV'] || 'development'

    # step 1: setup redis
    resque_config = YAML.load_file(rails_root + '/config/resque.yml')
    Resque.redis = resque_config[rails_env]

    # step 2: setup redis namespace
    Resque.redis.namespace = "podknife_#{Rails.env}"

    # step 3: load schedule jobs
    Resque.schedule = YAML.load_file(Rails.root + "config/resque_scheduler_#{Rails.env}.yml")

**job 的实现**

resque 中的 job，必要条件是实现 self.perform 方法，可以接受参数。并且要指定 @queue=xxx。

示例：

    class XlsxProcessJob
      @queue = :xlsx_process

      def self.perform(xlsx_file_path, xlsx_file_name)
        xlsx = XlsxParser.new(xlsx_file_path)
        xlsx.parse

        if xlsx.err_msg.present?
          AdminMailer.xlsx_process_result(xlsx_file_name, xlsx.err_msg).deliver
          return
        end

        @err_msg = []
        ...
        AdminMailer.xlsx_process_result(xlsx_file_name, @err_msg).deliver
      end
    end

**执行一次性 job**

上面我们说到了用 resque 执行周期性任务，如果要在代码中手动执行一次 job，用 Resque.enqueue 方法，将 job 放到待执行的队列中，比如执行上面的  XlsxProcessJob。

    Resque.enqueue(XlsxProcessJob, file_path, file_name)

**Resque 的其它 API**

检查 job 是否正在执行中：

    pry(main)> Resque.workers.each { |worker| ... }
    ...

    pry(main)> worker = Resque.workers.first
    pry(main)> worker.queues
    => ['podcast', 'update_podcast', 'xlsx_process']

    pry(main)> worker.processing
    => {"queue"=>"update_podcast", "run_at"=>"2017-09-26T06:54:39Z", "payload"=>{"class"=>"UpdatePodcastJob", "args"=>[]}}

    pry(main)> worker.working?
    => true

检查 job 是否处于待执行的列队中：

    #peek(queue, start = 0, count = 1) ⇒ Object
    Returns an array of items currently queued.

    pry(main)> Resque.queue_from_class(UpdatePodcastJob)
    => :update_podcast

    pry(main)> Resque.peek("update_hashtags")
    => nil
    # 处于 queue 中待命，还没有扔到 worker 中执行
    pry(main)> Resque.peek(:update_podcast)
    => {"class"=>"UpdatePodcastJob", "args"=>[]}

一个例子，在手动启动一个 job 时，先检查它有没有正在执行中或是已处于队列中，只有不满足前两者的情况下，才将此 job 入队。

    # http://www.rubydoc.info/gems/resque/Resque
    def job_status
      updation_job_queue = Resque.queue_from_class(UpdatePodcastJob)

      # check whether the job is processing by worker
      processing_worker = Resque.workers.select { |worker| worker.processing && worker.processing['queue'] == updation_job_queue.to_s }
      return 'A job is already being processed in the background. Only one job can be processed at a time.' if processing_worker.present?

      # check whether the job is in the queue
      job = Resque.peek(updation_job_queue)
      return 'There is already a pending job in the queue.' if job.present?

      nil
    end

    def updation_job_new
      @content = job_status
      render 'updation_job'
    end

    def updation_job_create
      Resque.enqueue(UpdatePodcastJob, true) if job_status.nil?
      @content = I18n.t('messages.podcast.job_enqueued')
      render 'updation_job'
    end
