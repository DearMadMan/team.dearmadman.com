title: 使用 Supervisor 进行进程管理
date: 2016-07-14 10:34:53
tags: [php, laravel]
---

# 使用 Supervisor 进行进程管理

## 简介

> **Supervisor**
> 
> Supervisor 是可以在类 UNIX 系统中进行管理和监控各种进程的小型系统。它自带了客户端和服务端工具。它可以通过用户所定义的配置文件来管理和监控单个或多个进程，并且它可以根据配置来对异常崩溃的进程进行重启操作。

我们可以使用 Supervisor 来轻松的管理和维护 nginx redis node 的进程，当这些进程因为某些原因出现异常而崩溃时，Supervisor 会自动的帮助它重新启动。

## 安装

如果你安装了 Python 的 [setuptools](http://peak.telecommunity.com/DevCenter/EasyInstall#installation-instructions) 模块，那么你就可以使用 `easy_install` 来进行安装，这是首选的安装方式：

```bash
easy_install supervisor
```

或者你也可以通过 `pip`:

```bash
pip install supervisor
```

## 生成配置文件

当你安装完成 Supervisor 之后，你就可以使用 `echo_supervisord_conf` 命令来输出一个配置文件：

```bash
echo_supervisord_conf > /etc/supervisord.conf
```

当然你可以将配置文件生成到任意位置，但是你需要在启动时使用 `-c` 选项来指定配置文件的路径。

## 配置

配置选项:

```bash
; Sample supervisor config file.
;
; For more information on the config file, please see:
; http://supervisord.org/configuration.html
;
; Notes:
;  - Shell expansion ("~" or "$HOME") is not supported.  Environment
;    variables can be expanded using this syntax: "%(ENV_HOME)s".
;  - Comments must have a leading space: "a=b ;comment" not "a=b;comment".

[unix_http_server]
file=/tmp/supervisor.sock   ; (the path to the socket file)
;chmod=0700                 ; socket file mode (default 0700)
;chown=nobody:nogroup       ; socket file uid:gid owner 
;username=user              ; (default is no username (open server)) 
;password=123               ; (default is no password (open server))

;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

[supervisord]
logfile=/tmp/supervisord.log ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB        ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10           ; (num of main logfile rotation backups;default 10)
loglevel=info                ; (log level;default info; others: debug,warn,trace)
pidfile=/tmp/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=false               ; (start in foreground if true;default false)
minfds=1024                  ; (min. avail startup file descriptors;default 1024)
minprocs=200                 ; (min. avail process descriptors;default 200)
;umask=022                   ; (process file creation umask;default 022)
;user=chrism                 ; (default is current user, required if root)
;identifier=supervisor       ; (supervisord identifier, default is 'supervisor')
;directory=/tmp              ; (default is not to cd during start)
;nocleanup=true              ; (don't clean up tempfiles at start;default false)
;childlogdir=/tmp            ; ('AUTO' child log dir, default $TEMP)
;environment=KEY="value"     ; (key value pairs to add to environment)
;strip_ansi=false            ; (strip ansi escape codes in logs; def. false)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as http_username if set
;password=123                ; should be same as http_password if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")
;history_file=~/.sc_history  ; use readline history if available

; The below sample program section shows all possible program subsection values,
; create one or more 'real' program: sections to be able to control them under
; supervisor.

;[program:theprogramname]
;command=/bin/cat              ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=999                  ; the relative start priority (default 999)
;autostart=true                ; start at supervisord start (default: true)
;startsecs=1                   ; # of secs prog must stay up to be running (def. 1)
;startretries=3                ; max # of serial start failures when starting (default 3)
;autorestart=unexpected        ; when to restart if exited after running (def: unexpected)
;exitcodes=0,2                 ; 'expected' exit codes used with autorestart (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;stopasgroup=false             ; send stop signal to the UNIX process group (default false)
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A="1",B="2"       ; process environment additions (def no adds)
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample eventlistener section shows all possible
; eventlistener subsection values, create one or more 'real'
; eventlistener: sections to be able to handle event notifications
; sent by supervisor.

;[eventlistener:theeventlistenername]
;command=/bin/eventlistener    ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;events=EVENT                  ; event notif. types to subscribe to (req'd)
;buffer_size=10                ; event buffer queue size (default 10)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=-1                   ; the relative start priority (default -1)
;autostart=true                ; start at supervisord start (default: true)
;startsecs=1                   ; # of secs prog must stay up to be running (def. 1)
;startretries=3                ; max # of serial start failures when starting (default 3)
;autorestart=unexpected        ; autorestart if exited after running (def: unexpected)
;exitcodes=0,2                 ; 'expected' exit codes used with autorestart (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;stopasgroup=false             ; send stop signal to the UNIX process group (default false)
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=false         ; redirect_stderr=true is not allowed for eventlisteners
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A="1",B="2"       ; process environment additions
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample group section shows all possible group values,
; create one or more 'real' group: sections to create "heterogeneous"
; process groups.

;[group:thegroupname]
;programs=progname1,progname2  ; each refers to 'x' in [program:x] definitions
;priority=999                  ; the relative start priority (default 999)

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

;[include]
;files = relative/directory/*.ini
```

我们只需要关注 `program` 配置项，每个 `program` 代表了一个要进行管理的进程，我们可以在这里进行进程的相关配置：

```bash
; * 为必选项
;[program:theprogramname]
;command=/bin/cat              ; * 要执行的命令路径（可以是相对与 PATH 的路径）也可带参数
;process_name=%(program_name)s ; 当进程数为 1 时 为 %(program_name)s 当进程数 >1 时 应配置为 %(program_name)s_%(process_num)02d
;numprocs=1                    ; 进程数量 (默认 1)
;directory=/tmp                ; 在执行运行前切换到目录 (def no cwd)
;umask=022                     ; 掩码 (default None)
;priority=999                  ; 优先级，数值越低越先启动而越后关闭 (default 999)
;autostart=true                ; 在 supervisord 启动时即启动 (default: true)
;startsecs=1                   ; 需要考虑进程启动成功的时间, 当 running 状态超过该值时，表明启动成功 (def. 1)
;startretries=3                ; 在启动时状态为失败时的最大尝试重启次数 (default 3)
;autorestart=unexpected        ; 当进程退出时是否应该重启，可选值为 false true unexpected ，为 false 时表示不重启，为 true 表示重启，为 unexpected 时，如果退出状态码不是 exitcodes 中之一时进行重启 (def: unexpected)
;exitcodes=0,2                 ; 用来重启的状态码 (default 0,2)
;stopsignal=QUIT               ; 进程停止信号，当用设定的信号去干掉进程，退出码会被认为是 expected (default TERM)
;stopwaitsecs=10               ; 这个是当我们向子进程发送 stopsignal 信号后，到系统返回信息给 supervisord，所等待的最大时间。超过这个时间，supervisord会向该子进程发送一个强制 kill 的信号。 (default 10)
;stopasgroup=false             ; 如果设置为 true 那么将会终止该进程下的所有子进程 (default false)
;killasgroup=false             ; 当进程关闭时向该进程的子进程发送的是 kill 信号 (def false)
;user=chrism                   ; 可以管理该 program 的用户
;redirect_stderr=true          ; 如果为 true，那么 stderr 将会被写入 stdout 日志文件中 (default false)
;stdout_logfile=/a/path        ; 日志文件路径， NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; 日志文件的最大大小 (default 50MB)
;stdout_logfile_backups=10     ; 日志文件的最大数量 (default 10)
;stdout_capture_maxbytes=1MB   ; capture 管道的大小，当值不为 0 时，子进程可以从 stdout 发送信息，而 supervisor 可以根据信息，发送相应的 event (default 0)
;stdout_events_enabled=false   ; 当设置为 ture 的时候，当子进程由 stdout 向文件描述符中写日志的时候，将触发 supervisord 发送 PROCESS_LOG_STDOUT 类型的 event (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A="1",B="2"       ; 子进程共享的环境变量 (def no adds)
;serverurl=AUTO                ; override serverurl computation (childutils)
```

常用配置可以像下面一样：

```bash
[program:node]
command=node server.js
process_name=%(program_name)s
numprocs=1
directory=%(here)s/server
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=%(here)s/node-server.log
priority=999
```

## 启动 

配置完成之后使用 `supervisord` 命令来启动服务端监控：

```bash
supervisord -c /etc/supervisord.conf
```

如果不指定 `-c` 选项，那么会使用以下顺序路径进行检索 `supervisord.conf` 文件
- $CWD/supervisord.conf
- $CWD/etc/supervisord.conf
- /etc/supervisord.conf
- /etc/supervisor/supervisord.conf
- ../etc/supervisord.conf
- ../supervisord.conf

## 使用 supervisorctl 管理

启动 supervisorctl 时需要指定配置文件路径，否则它将和 supervisord 使用相同的配置文件检索方式。

```bash
supervisorctl -c /etc/supervisord.conf
```

在 supervisorctl 的 shell 命令中可以使用以下命令：

```bash
> status       # 查询进程状态
> stop node    # 关闭 [program:node] 的进程
> start node   # 启动 [program:node] 的进程
> restart node # 重启 [program:node] 的进程
> stop all     # 关闭所有进程
> start all    # 启动所有进程
> reread       # 重新读取配置文件,读取有更新（增加）的配置文件，不会启动新添加的程序
> update       # 重启配置文件修改过的程序
```

除了在 shell 中使用命令，你也可以在命令行工具中直接输入 supervisorctl 命令:

```php
supervisorctl stop all
supervisorctl reload
```

