# Gitlab CI Note

## References

1.[用 GitLab CI 进行持续集成](https://scarletsky.github.io/2016/07/29/use-gitlab-ci-for-continuous-integration/)

## Note 1

Note for [用 GitLab CI 进行持续集成](https://scarletsky.github.io/2016/07/29/use-gitlab-ci-for-continuous-integration/)

从 GitLab 8.0 开始，GitLab CI 就已经集成在 GitLab 中，我们只要在项目中添加一个 `.gitlab-ci.yml` 文件，然后添加一个 Runner，即可进行持续集成。

### 关键概念

Pipeline，Stages，Jobs。

**Pipeline**

一次 Pipeline 其实相当于一次构建任务，里面可以包含多个流程，如安装依赖、运行测试、编译、部署测试服务器、部署生产服务器等流程。

任何提交或者 Merge Request 的合并都可以触发 Pipeline，我们用 `.gitlab-ci.yml` 来定义 Pipeline。

**Stages**

Stages 表示构建阶段，说白了就是上面提到的流程。

我们可以在一次 Pipeline 中定义多个 Stages，这些 Stages 会有以下特点：

- 所有 Stages 会按照顺序运行，即当一个 Stage 完成后，下一个 Stage 才会开始
- 只有当所有 Stages 完成后，该构建任务 (Pipeline) 才会成功
- 如果任何一个 Stage 失败，那么后面的 Stages 不会执行，该构建任务 (Pipeline) 失败

**Jobs**

Jobs 表示构建工作，表示某个 Stage 里面执行的工作。

我们可以在 Stages 里面定义多个 Jobs，这些 Jobs 会有以下特点：

- 相同 Stage 中的 Jobs 会并行执行
- 相同 Stage 中的 Jobs 都执行成功时，该 Stage 才会成功
- 如果任何一个 Job 失败，那么该 Stage 失败，即该构建任务 (Pipeline) 失败

### Gitlab Runner

Gitlab Runner 用来执行 Pipeline，它可以和 Gitlab 安装在不同的机器上，从而不影响 Gitlab 的性能。

### `.gitlab-ci.yml`

基本写法：

    # 定义 stages
    stages:
      - build
      - test

    # 定义 job
    job1:
      stage: test
      script:
        - echo "I am job1"
        - echo "I am in test stage"

    # 定义 job
    job2:
      stage: build
      script:
        - echo "I am job2"
        - echo "I am in build stage"

**常用关键字**

1. `stages`：定义 Stages，默认有三个 Stages - `build`、`test`、`deploy`
1. `before_script`：定义任何 Jobs 运行前都会执行的命令
1. `after_script`：定义任何 Jobs 运行后都会执行的命令
1. `variables` && `Job.variables`：定义环境变量
1. `cache` && `Job.cache`：定义需要缓存的文件

   每个 Job 开始的时候，Runner 都会删除 `.gitignore` 里的文件。如果有些文件 (如 `node_modules/`) 需要多个 Jobs 共用的话，我们只能让每个 Job 都先执行一遍 `npm install`。这样很不方便，因此我们需要对这些文件进行缓存。缓存了的文件除了可以跨 Jobs 使用外，还可以跨 Pipeline 使用。

1. `Job.script`：定义 Job 要运行的命令
1. `Job.stage`：定义 Job 的 stage，默认是 `test`
1. `Job.artifacts`：定义 Job 中生成的附件

   当该 Job 运行成功后，生成的文件可以作为附件 (如生成的二进制文件) 保留下来，打包发送到 GitLab，之后我们可以在 GitLab 的项目页面下下载该附件。

1. `Job.only`：如果只在某些分支上执行此 Job，则定义此选项
1. `Job.except`：如果不在某些分支上执行此 Job，则定义上选项

## Note 2

- [Gitlab Official CI Document](https://docs.gitlab.com/ce/ci/README.html)
- [Setting up GitLab CI for Android projects](https://about.gitlab.com/2016/11/30/setting-up-gitlab-ci-for-android-projects/)
- [Setting up GitLab CI for iOS projects](https://about.gitlab.com/2016/03/10/setting-up-gitlab-ci-for-ios-projects/)

看完官方文档，大致明白了怎么用。但还有一个疑问，当 Gitlab Runner 通过镜像启动一个 container 后，它应该是要把当前这个 git repo 作为数据卷挂载到 container 里的，是怎么挂载的，挂载到哪个路径，没有详细说明。

另外，Runner 在 container 里执行了某个 job 后的所有输出日志，肯定也是存在数据卷里的，而不是直接放在 container 里，否则 container 删除后，这些日志也消失了。

在这篇文档里有所描述：[The Docker executor](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/blob/master/docs/executors/docker.md)

> The Docker executor by default stores all builds in `/builds/<namespace>/<project-name>` and all caches in `/cache` (inside the container).
