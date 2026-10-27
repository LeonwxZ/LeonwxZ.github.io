---
title: "Linux System Programming"
date: 2026-09-20
math: true
draft: true
---

> 适合：已经安装好 WSL2，但没系统学过 Linux、嵌入式或计算机系统的初学者。  
> 环境：Windows + WSL2 + Ubuntu。  
> 目标：掌握文件IO、系统调用、进程/线程、互斥锁、信号量、共享内存、条件变量、死锁、TCP/UDP Socket、`select` 和 `epoll`。

---

# 1. 学习路线
```text
→ Linux 基本命令
→ C + GCC
→ 文件描述符 fd
→ 标准 IO / 系统调用
→ open/read/write/close
→ 进程
→ 线程
→ mutex
→ 条件变量
→ semaphore
→ 共享内存
→ 死锁
→ TCP/UDP
→ socket
→ select
→ epoll
```
---

# 2. 准备 WSL2 开发环境

打开 Ubuntu，或者 PowerShell 输入：

```powershell
wsl
```

进入 WSL 后：

```bash
sudo apt update
sudo apt install -y build-essential gdb vim nano tree strace lsof net-tools
```

检查：

```bash
gcc --version
gdb --version
```

建立学习目录：

```bash
mkdir -p ~/linux_system_study
cd ~/linux_system_study

mkdir 01_basic
mkdir 02_file_io
mkdir 03_process
mkdir 04_thread
mkdir 05_mutex
mkdir 06_condition
mkdir 07_semaphore
mkdir 08_shared_memory
mkdir 09_tcp
mkdir 10_udp
mkdir 11_select
mkdir 12_epoll
```

查看：

```bash
tree
```

---

# 3. 先掌握最基本的 Linux 命令

```bash
pwd                 # 当前目录
ls                  # 查看文件
ls -l               # 详细查看
cd dir              # 进入目录
cd ..               # 返回上一级
cd ~                # 回到用户目录

mkdir test          # 创建目录
touch a.txt         # 创建文件
cp a.txt b.txt      # 复制
mv a.txt b.txt      # 移动/重命名
rm a.txt            # 删除文件
rm -r dir           # 删除目录

cat file.txt        # 查看文件
grep "abc" file.txt # 搜索文本
ps aux              # 查看进程
top                 # 查看 CPU/内存
nano file.c         # 编辑文件
```

---

# 4. 第一个 Linux C 程序

进入：

```bash
cd ~/linux_system_study/01_basic
nano hello.c
```

输入：

```c
#include <stdio.h>

int main()
{
    printf("Hello Linux!\n");
    return 0;
}
```

编译：

```bash
gcc -Wall -Wextra -g hello.c -o hello
```
其中：
```text
-Wall   打开常见警告
-Wextra 打开更多警告
-g      加入 GDB 调试信息
```

运行：

```bash
./hello
```



---

# 5. Linux 最重要的概念之一：文件描述符 fd

Linux 中很多资源都可以用一个整数表示：

```text
文件
Socket
管道
设备
    ↓
文件描述符 fd
```

例如：

```c
int fd = 3;
```

可以理解为：

```text
3 → 某个已经打开的资源
```

程序启动后默认有：

```text
0 → stdin
1 → stdout
2 → stderr
```

也就是：

```text
标准输入
标准输出
标准错误
```

---

# 6. 标准 IO 与系统调用 IO

## 6.1 标准 IO

属于 C 标准库：

```c
fopen()
fread()
fwrite()
fclose()
printf()
fprintf()
```

主要使用：

```c
FILE *
```

例如：

```c
FILE *fp = fopen("test.txt", "w");
```

## 6.2 系统调用 IO

Linux 常用系统调用：

```c
open()
read()
write()
close()
```

主要使用：

```c
int fd
```

例如：

```c
int fd = open("test.txt", O_RDONLY);
```

## 6.3 二者关系

可以粗略理解：

```text
你的程序
   ↓
fwrite()
   ↓
C 标准库缓冲区
   ↓
write()
   ↓
Linux 内核
```

标准 IO 往往有缓冲区，因此可以减少系统调用次数。

---

# 7. open / read / write / close

进入：

```bash
cd ~/linux_system_study/02_file_io
```

## 7.1 open

头文件：

```c
#include <fcntl.h>
```

例如：

```c
int fd = open("test.txt", O_RDONLY);
```

常见标志：

```text
O_RDONLY  只读
O_WRONLY  只写
O_RDWR    读写
O_CREAT   不存在就创建
O_TRUNC   打开后清空
O_APPEND  追加
```

例如：

```c
int fd = open(
    "test.txt",
    O_WRONLY | O_CREAT | O_TRUNC,
    0644
);
```

---

## 7.2 write

```c
ssize_t write(
    int fd,
    const void *buf,
    size_t count
);
```

例如：

```c
char msg[] = "hello\n";
write(fd, msg, 6);
```

---

## 7.3 read

```c
ssize_t read(
    int fd,
    void *buf,
    size_t count
);
```

返回值：

```text
> 0  读取到的数据字节数
= 0  到达 EOF
< 0  出错
```

---

## 7.4 close

```c
close(fd);
```

用完文件描述符后要关闭。

---

# 8. 文件写入实验

```c
#include <stdio.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>

int main()
{
    int fd = open(
        "test.txt",
        O_WRONLY | O_CREAT | O_TRUNC,
        0644
    );

    if(fd < 0)
    {
        perror("open");
        return 1;
    }

    const char *msg = "Hello Linux System Call!\n";

    ssize_t n = write(fd, msg, strlen(msg));

    if(n < 0)
    {
        perror("write");
        close(fd);
        return 1;
    }

    close(fd);
    return 0;
}
```

编译运行：

```bash
gcc -Wall -Wextra -g write_demo.c -o write_demo
./write_demo
cat test.txt
```

用 `strace` 观察系统调用：

```bash
strace -e trace=openat,write,close ./write_demo
```

---

# 9. 文件读取实验

```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>

int main()
{
    int fd = open("test.txt", O_RDONLY);

    if(fd < 0)
    {
        perror("open");
        return 1;
    }

    char buf[128];

    ssize_t n = read(fd, buf, sizeof(buf) - 1);

    if(n < 0)
    {
        perror("read");
        close(fd);
        return 1;
    }

    buf[n] = '\0';

    printf("Read %zd bytes:\n%s", n, buf);

    close(fd);
    return 0;
}
```

---

# 10. 文件复制器

目标：

```bash
./copy source.txt target.txt
```

只能使用：

```text
open
read
write
close
```

核心流程：

```text
open(src)
open(dst)
   ↓
while(read(src))
   ↓
write(dst)
   ↓
close
```

```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>

int main(int argc, char *argv[]){

        char *s1 = argv[1];
        char *s2 = argv[2];
        printf("%s\n%s", s1, s2);

        int fd_1 = open(s1, O_RDONLY);
        int fd_2 = open(s2, O_WRONLY|O_TRUNC);

        char buf[128];
        ssize_t n;
        while(n =  read(fd_1, buf, sizeof(buf) - 1))
        {
                write(fd_2, buf, n);
        };

        return 0;
}
```
---

# 11. 进程

程序和进程不同：

```text
程序 = 磁盘上的文件
进程 = 正在运行的程序实例
```

查看进程：

```bash
ps aux
```

每个进程都有：

```text
PID
```

获取 PID：

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    printf("PID = %d\n", getpid());
    return 0;
}
```

---

# 12. fork 创建子进程

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    pid_t pid = fork();

    if(pid < 0)
    {
        perror("fork");
        return 1;
    }

    if(pid == 0)
    {
        printf("child, PID=%d\n", getpid());
    }
    else
    {
        printf(
            "parent, PID=%d, child=%d\n",
            getpid(),
            pid
        );
    }

    return 0;
}
```

记住：

```text
pid < 0  创建失败
pid = 0  当前是子进程
pid > 0  当前是父进程
```

---

# 13. wait 等待子进程

```c
#include <sys/wait.h>
```

父进程：

```c
wait(NULL);
```

表示：

```text
等子进程结束后再继续
```

---

# 14. 线程

一个进程内部可以有多个线程：

```text
进程
├── 线程1
├── 线程2
└── 线程3
```

线程共享：

```text
代码
全局变量
堆
文件描述符
```

每个线程有自己的：

```text
栈
寄存器
线程状态
```

核心区别：

```text
进程之间地址空间通常独立
同一进程内线程共享大量资源
```

---

# 15. pthread 创建线程

```c
#include <stdio.h>
#include <pthread.h>

void *worker(void *arg)
{
    (void)arg;
    printf("worker running\n");
    return NULL;
}

int main()
{
    pthread_t tid;

    pthread_create(
        &tid,
        NULL,
        worker,
        NULL
    );

    pthread_join(tid, NULL);

    return 0;
}
```

编译：

```bash
gcc thread_demo.c -o thread_demo -pthread
```

---

# 16. Race Condition

假设：

```c
int count = 0;
```

两个线程都执行：

```c
count++;
```

`count++` 可以粗略理解成：

```text
读取 count
加 1
写回 count
```

可能出现：

```text
A 读取 0
B 读取 0
A 写入 1
B 写入 1
```

最后结果：

```text
1
```

而不是 2。

这叫：

```text
Race Condition
```

---

# 17. Mutex 互斥锁

定义：

```c
pthread_mutex_t mutex =
    PTHREAD_MUTEX_INITIALIZER;
```

加锁：

```c
pthread_mutex_lock(&mutex);
```

解锁：

```c
pthread_mutex_unlock(&mutex);
```

例如：

```c
pthread_mutex_lock(&mutex);

count++;

pthread_mutex_unlock(&mutex);
```

这段被保护的代码叫：

```text
临界区
```

---

# 18. Mutex 完整例子

```c
#include <stdio.h>
#include <pthread.h>

long long count = 0;

pthread_mutex_t mutex =
    PTHREAD_MUTEX_INITIALIZER;

void *worker(void *arg)
{
    (void)arg;

    for(int i = 0; i < 1000000; i++)
    {
        pthread_mutex_lock(&mutex);

        count++;

        pthread_mutex_unlock(&mutex);
    }

    return NULL;
}

int main()
{
    pthread_t t1, t2;

    pthread_create(&t1, NULL, worker, NULL);
    pthread_create(&t2, NULL, worker, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    printf("count = %lld\n", count);

    pthread_mutex_destroy(&mutex);

    return 0;
}
```

---

# 19. 死锁

经典情况：

```text
线程 A：
拿 lock1
等 lock2

线程 B：
拿 lock2
等 lock1
```

双方永远等待。

避免死锁最常见方法：

```text
1. 固定加锁顺序
2. 减少同时持有多个锁
3. 临界区尽量短
4. 必要时使用 trylock
```

最重要：

```text
所有线程统一锁顺序
```

---

# 20. 条件变量

错误的等待方式：

```c
while(!ready)
{
}
```

这会一直占 CPU。

条件变量允许线程：

```text
没数据时睡眠
有数据后被唤醒
```

定义：

```c
pthread_cond_t cond =
    PTHREAD_COND_INITIALIZER;
```

等待：

```c
pthread_cond_wait(&cond, &mutex);
```

唤醒：

```c
pthread_cond_signal(&cond);
```

---

# 21. 条件变量标准模板

消费者：

```c
pthread_mutex_lock(&mutex);

while(!ready)
{
    pthread_cond_wait(
        &cond,
        &mutex
    );
}

处理数据;

pthread_mutex_unlock(&mutex);
```

生产者：

```c
pthread_mutex_lock(&mutex);

修改共享数据;
ready = 1;

pthread_cond_signal(&cond);

pthread_mutex_unlock(&mutex);
```

注意：

```text
通常使用 while，而不是 if
```

因为线程被唤醒后要重新检查条件。

---

# 22. Semaphore 信号量

信号量可以理解为：

```text
带计数功能的同步工具
```

例如资源数量为 2：

```c
sem_init(&sem, 0, 2);
```

等待资源：

```c
sem_wait(&sem);
```

释放资源：

```c
sem_post(&sem);
```

头文件：

```c
#include <semaphore.h>
```

---

# 23. Mutex 与 Semaphore

```text
Mutex
→ 强调互斥
→ 通常一次一个线程
→ 通常谁加锁谁解锁

Semaphore
→ 强调计数/同步
→ 可以允许 N 个线程同时进入
→ 可以用来做线程通知
```

---

# 24. 共享内存

进程默认不能直接共享普通变量。

如果需要高速交换大量数据，可以使用：

```text
Shared Memory
```

结构：

```text
进程 A
    \
     Shared Memory
    /
进程 B
```

常见 API：

```c
shm_open()
ftruncate()
mmap()
munmap()
shm_unlink()
```

流程：

```text
shm_open
↓
ftruncate
↓
mmap
↓
读写内存
↓
munmap
```

共享内存只解决：

```text
能否看到同一份数据
```

它不自动解决：

```text
谁先读
谁先写
同时修改怎么办
```

所以常常还要配合同步机制。

---

# 25. 网络基础：IP 和端口

例如：

```text
127.0.0.1:8888
```

其中：

```text
127.0.0.1 → 本机 IP
8888      → 端口
```

可以理解：

```text
IP   → 找机器
Port → 找机器上的程序
```

---

# 26. TCP 与 UDP

## TCP

特点：

```text
面向连接
可靠
有序
丢失可重传
```

适合：

```text
SSH
HTTP
文件传输
控制命令
```

## UDP

特点：

```text
无连接
开销小
速度快
不保证送达
不保证顺序
```

适合：

```text
实时音视频
广播
部分传感器数据
```

---

# 27. Socket

Socket 是 Linux 的网络编程接口。

创建：

```c
int fd = socket(...);
```

注意：

```text
Socket 最终也是一个 fd
```

所以：

```text
普通文件 → fd
Socket   → fd
Pipe     → fd
```

很多情况下都可以：

```c
read(fd, ...)
write(fd, ...)
close(fd)
```

---

# 28. TCP Server 流程

```text
socket
  ↓
bind
  ↓
listen
  ↓
accept
  ↓
read/write
  ↓
close
```

TCP Client：

```text
socket
  ↓
connect
  ↓
read/write
  ↓
close
```

---

# 29. TCP Server 示例

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

int main()
{
    int server_fd = socket(
        AF_INET,
        SOCK_STREAM,
        0
    );

    if(server_fd < 0)
    {
        perror("socket");
        return 1;
    }

    struct sockaddr_in addr = {0};

    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = htonl(INADDR_ANY);
    addr.sin_port = htons(8888);

    if(bind(
        server_fd,
        (struct sockaddr *)&addr,
        sizeof(addr)
    ) < 0)
    {
        perror("bind");
        return 1;
    }

    if(listen(server_fd, 5) < 0)
    {
        perror("listen");
        return 1;
    }

    printf("listening on 8888...\n");

    int client_fd = accept(
        server_fd,
        NULL,
        NULL
    );

    if(client_fd < 0)
    {
        perror("accept");
        return 1;
    }

    char buf[1024];

    ssize_t n = read(
        client_fd,
        buf,
        sizeof(buf) - 1
    );

    if(n > 0)
    {
        buf[n] = '\0';
        printf("client: %s\n", buf);
    }

    const char *reply = "hello client";

    write(
        client_fd,
        reply,
        strlen(reply)
    );

    close(client_fd);
    close(server_fd);

    return 0;
}
```

---

# 30. TCP Client 示例

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>

int main()
{
    int fd = socket(
        AF_INET,
        SOCK_STREAM,
        0
    );

    struct sockaddr_in server = {0};

    server.sin_family = AF_INET;
    server.sin_port = htons(8888);

    inet_pton(
        AF_INET,
        "127.0.0.1",
        &server.sin_addr
    );

    if(connect(
        fd,
        (struct sockaddr *)&server,
        sizeof(server)
    ) < 0)
    {
        perror("connect");
        return 1;
    }

    const char *msg = "hello server";

    write(fd, msg, strlen(msg));

    char buf[1024];

    ssize_t n = read(
        fd,
        buf,
        sizeof(buf) - 1
    );

    if(n > 0)
    {
        buf[n] = '\0';
        printf("server: %s\n", buf);
    }

    close(fd);

    return 0;
}
```

终端 1：

```bash
gcc server.c -o server
./server
```

终端 2：

```bash
gcc client.c -o client
./client
```

查看端口：

```bash
ss -lntp | grep 8888
```

---

# 31. UDP 流程

UDP Server：

```text
socket
↓
bind
↓
recvfrom
↓
sendto
```

UDP Client：

```text
socket
↓
sendto
↓
recvfrom
```

UDP 通常不需要：

```text
listen
accept
```

---

# 32. 阻塞 IO

例如：

```c
read(fd, buf, 100);
```

如果没有数据：

```text
线程会停在 read()
```

直到数据到来。

这叫：

```text
Blocking IO
```

---

# 33. 为什么需要 IO 多路复用？

如果服务器有：

```text
1000 个客户端
```

一种方法：

```text
每个客户端一个线程
```

可能就要：

```text
1000 个线程
```

这会带来：

```text
内存开销
线程调度
上下文切换
同步问题
```

所以 Linux 提供：

```text
select
poll
epoll
```

目标：

> 一个或少量线程，同时等待很多 fd。

---

# 34. select

核心思想：

```text
fd1
fd2
fd3
fd4
 ↓
select
 ↓
哪些 fd 有数据？
```

常见：

```c
FD_ZERO(&readfds);
FD_SET(fd, &readfds);
FD_ISSET(fd, &readfds);
```

基本模板：

```c
fd_set readfds;

while(1)
{
    FD_ZERO(&readfds);

    FD_SET(fd1, &readfds);
    FD_SET(fd2, &readfds);

    int maxfd = fd1 > fd2 ? fd1 : fd2;

    int ret = select(
        maxfd + 1,
        &readfds,
        NULL,
        NULL,
        NULL
    );

    if(ret < 0)
    {
        perror("select");
        break;
    }

    if(FD_ISSET(fd1, &readfds))
    {
        // fd1 可读
    }

    if(FD_ISSET(fd2, &readfds))
    {
        // fd2 可读
    }
}
```

---

# 35. select 的缺点

先掌握三点：

```text
1. fd 数量通常有限制
2. 每轮都要重新准备 fd 集合
3. 返回后通常要遍历检查多个 fd
```

---

# 36. epoll

Linux 为大量 fd 提供：

```text
epoll
```

三个核心 API：

```c
epoll_create1()
epoll_ctl()
epoll_wait()
```

思路：

```text
第一次：
告诉内核我要关注哪些 fd

以后：
epoll_wait 等待

内核：
直接返回发生事件的 fd
```

---

# 37. epoll 基本模板

```c
#include <sys/epoll.h>

int epfd = epoll_create1(0);

struct epoll_event ev;

ev.events = EPOLLIN;
ev.data.fd = listen_fd;

epoll_ctl(
    epfd,
    EPOLL_CTL_ADD,
    listen_fd,
    &ev
);

struct epoll_event events[64];

while(1)
{
    int n = epoll_wait(
        epfd,
        events,
        64,
        -1
    );

    for(int i = 0; i < n; i++)
    {
        int fd = events[i].data.fd;

        if(fd == listen_fd)
        {
            // accept 新客户端
        }
        else
        {
            // read 客户端数据
        }
    }
}
```

---

# 38. select 与 epoll 区别

| 项目 | select | epoll |
|---|---|---|
| 学习难度 | 较低 | 稍高 |
| 平台 | 多平台 | Linux |
| fd 数量 | 通常有限制 | 更适合大量连接 |
| 每轮重新传 fd 集合 | 是 | 通常不需要 |
| 是否常遍历大量 fd | 是 | 主要处理就绪事件 |
| 高并发 | 一般 | 更适合 |

可以这样理解：

```text
select：
“这 1000 个人里谁举手了？”
然后自己一个个检查。

epoll：
先登记这 1000 个人，
系统直接告诉你：
“3号、27号、811号举手了。”
```

---

# 39. LT 与 ET

epoll 常见：

```text
LT
ET
```

LT：

```text
Level Triggered
```

默认模式。

只要 fd 仍然可读，就会继续提醒。

ET：

```text
Edge Triggered
```

通常需要配合：

```text
非阻塞 IO
```

并一直读取到：

```text
EAGAIN
```

初学阶段：

```text
先掌握 LT
再学 ET
```

---

# 40. 最重要的一张总结图

```text
                 Linux 文件描述符 fd
                         │
        ┌────────────────┼───────────────┐
        │                │               │
     普通文件          Socket          Pipe
        │                │
 open/read/write   socket/read/write
        │                │
        └──────────┬─────┘
                   │
             select / epoll
                   │
              监控多个 fd
```