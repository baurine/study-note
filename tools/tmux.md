# tmux

tmux 与 terminal 相比有一个很大的不同，terminal 关闭后原来的信息就清除了，而 tmux 却不会。我猜想 tmux 是采用了进程间内存共享的技术，不同的 tmux 进程可以共享相同的内存数据，而 terminal 的内存是进程独立的。这样就使得 tmux 可以实现很多 terminal 无法实现的功能。比如：

1. 在公司 ssh 到远程服务器开了一个 tmux session 做了一些任务，回到家后可以继续使用这个 session。
1. 远程 pair programing，双方登录到同一台远程服务器，使用同一个 tmux session。

(我原以为 tmux 是将数据持久化到硬盘里了，但我尝试重启电脑后，原来的 session 被消除了，所以 tmux 的数据也只是在内存里，如果重启机器，session 也是会丢失的。)

tmux 管理多个 session，每个 session 都有多个 window，每个 window 可以分成多个 pane。

## References

1. [Wrangle your terminal with tmux](https://egghead.io/courses/wrangle-your-terminal-with-tmux)

## Note 1

Note for [Wrangle your terminal with tmux](https://egghead.io/courses/wrangle-your-terminal-with-tmux)

**0.install**

    $ brew install tmux

**1. pane**

以下用 `^` 表示 mac 上的 `ctrl` (control) 键。

- `^b"`：左右分 pane
- `^b%`：上面分 pane
- `^b + 方向键`：切换 pane
- `^d` 或 `exit` shell 命令：关闭 pane
- `^b?`：帮助

`^b"` 表示，先同时按下 `^b` 两个按键，然后松开，再同时按下 shift + `"|'` 键。

**2. window**

- `^bc`：新建 window
- `^bn`：前一个 window
- `^bp`：后一个 window

**3. session**

    $ tmux ls             // list session
    $ tmux attach         // attach 到最近的 session 上
    $ tmux attach -t [n]  // 指定 attach 到某个 session 上

`^bd`：detach 当前 session

tmux 管理多个 session，每个 session 里有多个 window，每个 window 里有多个 pane。

**4. rename session**

- `^bs`：在 session 内切换 session，然后用方向键选择
- `^b$`：在 session 内 rename session

从命令行 rename session

    $ tmux rename-session -t old_name new_name

从命令行新建具名的 session

    $ tmux new -s session_name

从命令行关闭 session

    $ tmux kill-session -t session_name

**5. resize pane**

- `^bz`：toggle fullscreen a pane
- `^b:` 进入 tmux 的命令模式，然后运行 `resize-pane -U/D/L/R [num]` 命令

`^b:` 用于进入 tmux 命令模式。

**6. tmux conf**

    $ vim ~/.tmux.conf

    # Rebind prefix
    unbind C-b
    set-option -g prefix C-a
    bind-key C-a send-prefix

    # Pretty color
    set -g status-bg blue
    set -g status-fg white

    # Easy reloading
    bind R source-file ~/.tmux.conf

在已运行的 tmux session 中重新读取配置，先用 `^b:` 进入 tmux 命令模式，然后执行 `source-file ~/.tmux.conf` 命令，在上面的 conf 中为此操作设置了一个快捷键 `R`，因此也可以在进入命令模式后执行 `R` (?? 还用回车吗，待试验)。

**7. copy and past**

略。直接用系统的 cmd+c cmd+v 就行。

- `^b[`：进入滚动模式，q 退出
- `^b: set-window-option mode-keys vi`：设置复制粘贴模式为 vi

**8. mouse enable**

- `^b: set mouse on`
- `^b: set mouse off`

在配置文件里为它们设置快捷键：

    bind m set -g mouse on
    bind M set -g mouse off

**9. script and workflow**

    $ vim tmuxlaunch.sh

    SESSION=dev
    tmux new-session -d -s $SESSION
    tmux new-window -t $SESSION:1 -n ‘webserver’
    tmux select-window -t $SESSINO:1
    tmux split-window -h
    tmux send-keys ‘cd examples/react; python -m SimpleHTTPServer’ C-m
    tmux attach -t $SESSION

**10. share a session for pair programing**

super cool~!

**11. handle history in tmux session**

配置 `~/.bashrc`：

    shopt -s histappend     // share option -s histroy append
    shopt -s hsitreedit
    shopt -s histverify
    HISTCONTROL='ignoreboth'
    PROMPT_COMMAND="history -a;history -c;history -r; $PROMPT_COMMAND"

DONE!

## Note 2

使用 iTerm2 的 tmux mode，tmux 的一个 window 可以独享一个 iTerm2 的 tab。

- [使用 iTerm2 管理 Tmux 会话](http://ponder.work/2021/08/02/iterm2-tmux-integration/)
