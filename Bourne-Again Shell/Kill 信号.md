#### 说明
``` txt
kill -l                  # 查看linux提供的信号

SIGHUP  1          A     # 终端挂起或者控制进程终止
SIGINT  2          A     # 键盘终端进程(如control+c)
SIGQUIT 3          C     # 键盘的退出键被按下
SIGILL  4          C     # 非法指令
SIGABRT 6          C     # 由abort(3)发出的退出指令
SIGFPE  8          C     # 浮点异常
SIGKILL 9          AEF   # Kill信号  立刻停止
SIGSEGV 11         C     # 无效的内存引用
SIGPIPE 13         A     # 管道破裂: 写一个没有读端口的管道
SIGALRM 14         A     # 闹钟信号 由alarm(2)发出的信号
SIGTERM 15         A     # 终止信号,可让程序安全退出 kill -15
SIGUSR1 30,10,16   A     # 用户自定义信号1
SIGUSR2 31,12,17   A     # 用户自定义信号2
SIGCHLD 20,17,18   B     # 子进程结束自动向父进程发送SIGCHLD信号
SIGCONT 19,18,25         # 进程继续（曾被停止的进程）
SIGSTOP 17,19,23   DEF   # 终止进程
SIGTSTP 18,20,24   D     # 控制终端（tty）上按下停止键
SIGTTIN 21,21,26   D     # 后台进程企图从控制终端读
SIGTTOU 22,22,27   D     # 后台进程企图从控制终端写

缺省处理动作一项中的字母含义如下:
    A  缺省的动作是终止进程
    B  缺省的动作是忽略此信号，将该信号丢弃，不做处理
    C  缺省的动作是终止进程并进行内核映像转储(dump core)
       内核映像转储是指将进程数据在内存的映像和进程在内核结构中的部分内容以一定格式转储到文件系统
       并且进程退出执行，这样做的好处是为程序员提供了方便，使得他们可以得到进程当时执行时的数据值，允许他们确定转储的原因
       并且可以调试他们的程序。
    D  缺省的动作是停止进程，进入停止状况以后还能重新进行下去，一般是在调试的过程中（例如ptrace系统调用）
    E  信号不能被捕获
    F  信号不能被忽略
```
