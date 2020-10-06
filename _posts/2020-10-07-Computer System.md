---
layout: post
title: Computer System
tags: Computer
---
## 系统级I/O
### Unix I/O
- 打开文件：内核返回非负整数，描述符
- 进程开始时，三个文件打开：标准输入（描述符为0）、标准输出（1）、标准错误（2）
- 每个文件，内核保存一个文件位置，字节偏移量k
- 读文件k增，当k>=file.length时，EOF；写文件k增
- 关闭文件，释放内存

### 打开关闭文件
1. 调用`int open(char *filename, int flags, mode_t mode)`，返回进程中未打开的最小描述符，出错返回`-1`
- flags：一个或多位掩码的或。只读、只写、读写 | 不存在则创建、存在则新建、结尾处添加
- mode：访问权限，调用umask函数，文件权限为`mode&~umask`（取反）
```
#define DEF_MODE S_IRUSR|S_IWUSR|S_IWGRP|S_IWOTH
#define DEF_UMASK S_IWGRP|S_IWOTH
umask(DEF_UMASK) // S_IRUSR|S_IWUSR当前用户的读写权限
```
2. 调用`int close(int fd)`

### 读写文件
1. 调用`ssize_t read(int fd, void *buf, size_t n)`，成功返回字节数，EOF返回0，出错返回`-1`。复制最多n字节到内存位置buf
2. 调用`ssize_t write(int fd, const *buf, size_t n)`，成功返回字节数，出错返回`-1`

	>size_t: unsigned long
	>ssize_t: long，有符号，由于存在出错，导致最大值小了一半

### RIO读写文件
Rubust健壮的读写
1. 无缓冲输入输出，内存和文件中传输数据，适用于网络读写二进制
```
ssize_t rio_readn(int fd, void *usrbuf, size_t n)
ssize_t rio_writen(int fd, void *usrbuf, size_t n)
```
2. 带缓冲输入输出，文件内容缓存在应用级缓冲区内，线程安全
```
void rio_readinitb(rio_t *rp, int fd)//将描述符fd与地址为rp处的类型为rio_t的读缓存区联系
ssize_t rio_readlineb(rio_t *rp, void *usrbuf, size_t maxlen)
ssize_t rio_readnb(rio_t *rp, void *usrbuf, size_t n) //包含文本行和二进制
```

### 共享文件
同一个文件可以对应多个文件表表项，但只对应一个v-node表表项。多个描述符通过不同的文件表项引用同一个文件
- 描述符表：进程独立，每个表项指向文件表中一个表项
- 文件表：进程共享，每个表项包含字节偏移量、引用计数、指向v-node表中表项的指针；引用计数为0时，内核删除此表项
- v-node表：进程共享，每个表项包含stat结构中大部分信息；同样会被删除

### 重定向
调用`dup2(int oldfd, int newfd)`，dup2(4,1)会将1指向4所指向的文件表表项，1原指向的文件表表项引用计数减1

### 标准I/O
C语言将一个打开的文件模型化为一个流，类型为FILE的流时对文件描述符和流缓冲区的抽象

### 流的选择
- 尽可能使用标准I/O
- 不要使用scanf或rio_readlineb读取二进制
- 网络嵌套字使用RIO
	> 限制一：跟在输出函数之后的输入函数。如果中间没有插入对fflush，fseek，fsetpos，rewind的调用，一个输入函数不能跟在一个输出函数之后。清空缓存区或重置文件位置
	> 限制二：跟在输入函数之后的输出函数，如果中间没有插入对fseek，fsetpos或者rewind的调用，一个输出函数不能跟在一个输入函数之后，除非该输入函数遇到了一个EOF

## 网络编程
### 套接字
- socket：客户端和服务端创建套接字描述符
- connect：客户端调用来建立连接，套接字描述符clientfd可以读写
- bind：服务器将地址与套接字描述符sockfd联系
- listen：服务器将主动套接字sockfd转换为监听套接字listenfd
- accept：服务端连接请求到达的监听描述符，返回已连接描述符connfd

### 辅助函数
- getaddrinfo：将主机名、主机地址、服务名、端口号字符串转换为套接字地址结构
- getnameinfo：与getaddrinfo相反
- `open_clientfd(*hostname, *port)`客户端调用
- `open_listenfd(*port)`服务端调用

### 迭代服务器，一次一个
客户端
```
int clientfd;
char *host, *port, buf[MAXLINE];
rio_t rio;

clientfd = open_clientfd(host, port);
rio_readinitb(&rio, clientfd);

while(fget(buf, MAXLINE, stdin)!=NULL) {
    rio_writen(clientfd, buf, strlen(buf));
    rio_readlineb(&rio, buf, MAXLINE);
    fput(buf, stout);
}
close(clientfd);
exit(0);
```
服务端
```
int listenfd, connfd;
socklen_t clientlen;
struct sockadd_storage clientaddr;
char client_hostname[MAXLINE], client_port[MAXLINE], buf[MAXLINE];
size_t n;
rio_t rio;

listenfd = open_listenfd(port);
while(1) {
    clientlen = sizeof(struct sockadd_storage);
    connfd = accept(listenfd, (SA *)&clientaddr, &clientlen);
    getnameinfo((SA *)&clientaddr, clientlen, client_hostname, MAXLINE, client_port, MAXLINE, 0);
    printf("Connect to (%s, %s)", client_hostname, client_port);
    
    rio_readinitb(&rio, connfd);
    while((n=rio_readlineb(&rio, buf, MAXLINE))!=0) {
        printf("Server received %d bytes", (int) n);
        rio_writen(connfd, buf, n);
    }
    close(connfd);
}
exit(0);
```

### CGI通用网关接口
- 客户端将参数传给服务器：GET，POST等
- 服务器将参数传给子进程：接收请求后，CGI调用fork创建一个子进程，子进程设置CGI环境变量（参数来源），调用execve在子进程上下文中执行CGI程序或CGI脚本（Perl脚本编写），例如计算两数之和的CGI程序
- 子进程输出：子进程加载CGI程序前，CGI使用dup2将标准输出重定向到和客户端相连的已连接描述符
```
char arg1[MAXLINE], arg2[MAXLINE], content[MAXLINE];
if((buf=getenv("QUERY_STRING"))!=null){
    p = strchr(buf, '&'); //返回第一次出现字符的指针
    *p = '\0'; //将&赋值为\0
    strcpy(arg1,buf);strcpy(arg2,p+1);
    n1=atoi(arg1);n2=atoi(arg2);
}
sprintf(content,"answer=%d",n1+n2);
printf("Connection: close\r\n");
printf("Content-length: %d\r\n", (int)strlen(content));
printf("Content-type: text/html\r\n\r\n")
printf("%s",content);
fflush(stdout);
exit(0)
```
## 并发编程
### 基于进程
- 包含一个SIGCHLD（一个进程终止或者停止时，将SIGCHLD信号发送给其父进程，父进程捕获）处理程序，回收僵死子进程的资源
```
void sigchld_handler(int sig) {
    while (waitpid(-1, 0, WNOHANG) > 0);
    return;
}
signal(SIGCHLD, sigchld_handler);
```

- 父子进程关闭各自的connfd
- 父子进程的connfd都关闭了，客户端连接才会终止
- 优劣：共享文件表，不共享用户地址空间。不会覆盖另一个进程的虚拟内存，但进程共享状态信息困难，需要显式的IPC（进程间通信）机制；进程控制和IPC的开销高会比较慢

### 基于I/O多路复用
使用select函数，内核挂起进程，在一个或多个I/O事件发生后，才将控制返回给应用

- `FD_ZERO(fd_set *fdset)`创建一个空的读集合，fd_set为描述符集合
- `FD_SET(int fd, fd_set *fdset)`添加fd到fd_set
- `int select(int n, fd_set *fdset, NULL, NULL, NULL)`监听
- `FD_ISSET(int fd, fd_set *fdset)`判断是否触发
```
while(1){
    ready_set=read_set;
    select(listenfd+1, &ready_set, NULL, NULL, NULL);
    if(FD_ISSET(listened, &ready_set)) {
        clientlen=sizeof(struct sockaddr_storage);
        connfd=accept(listenfd, (SA*)&clientaddr, &clientlen);
        echo(connfd);
        close(connfd);
	}
}

```

- 优劣：更多的程序控制行为，单一进程上下文中共享数据，不需要切换上下文高效；编码复杂，不能充分利用多核

### 基于线程
线程独立的线程上下文（寄存器），包括唯一整数线程ID、栈、栈指针、程序计数器、同于目的寄存器、条件码。线程共享进程虚拟空间
#### 线程基础
- 创建Posix线程
```
void *thread(void *vargp) {
    return NULL;
}
pthread tid;//存放对等线程ID
pthread_create(&tid, NULL, thread, NULL);
pthread_job(tid, NULL);

int pthread_create(pthread_t *tid, pthread_attr_t *attr, func *f, void *arg)//线程ID,线程默认属性,线程例程,输入变量
```

- 获取自己的线程ID`pthread_t pthread_self(void)`
- 终止线程
    > return，隐式终止
    > `void pthread_exit(void *thread_return)`，显式终止。主线程调用，会等待其他所有对等线程终止，再终止主线程和进程，返回值为thread_return，`pthread_exit(NULL)`
    > Linux exit终止进程
    > `void pthread_cancel(pthread_t tid)`
- 回收终止线程资源，`int pthread_join(pthread_t tid, void **thread_return)`，阻塞直到`tid`终止，线程例程返回的通用`(void*)`指针赋值为`thread_return`，然后回收资源
- 线程创建默认是可结合的，可以被其他线程收回和杀死；对立的为分离的`int pthread_detach(pthread_t tid)`
- 初始化多个线程共享的全局变量，代码只会执行一次
```
pthread_once_t once = PTHREAD_ONCE_INIT;
void once_run(void) {
    cout << "once" << endl;
}
void *child1(void *arg){
    pthread_once(&once, once_run);//两个线程里只会执行一次
}
void *child2(void *arg){
    pthread_once(&once, once_run);//两个线程里只会执行一次
}
```

- 共享变量
    > 全局变量：函数之外。虚拟内存读写区域只包含一个实例，任何线程可以引用
    > 本地自动变量：函数内部没有static属性的变量。在线程栈中
    > 本地静态变量：函数内部并有static属性的变量。虚拟内存读写区域只包含一个实例

- 信号量同步线程：共享变量，单纯的cnt++汇编对应三步：加载到线程中的寄存器，增加，更新共享变量。多线程时，会有同步错误
    > P操作：`s=s-1`；s大于等于0，继续执行；否则挂起线程入队
    > V操作：`s=s+1`；s大于0，继续执行；否则释放队列中第一个，多个不可预测
    > 保护共享变量的信号量为二元信号量，值总是0或1。互斥为目的的二元信号量称为互斥锁，P为加锁，V为解锁
    > 调度共享资源，生产者消费者（三个信号量：buf互斥锁、可用位置初始化为n、已用位置初始化为0）、读者写者（读者优先，第一个读者加写锁，最后一个读者解写锁，读者不断道道导致写者饥饿；写者优先）
```
// 全局变量
sem_t mutex;
// main
sem_init(&mutex, 0, 1);//mutex=1
// thread
p(&mutex);
cnt++;
v(&mutex);
```

#### 并发服务器
```
while(1){
    clientlen=sizeof(struct sockaddr_storage);
    connfd = malloc(sizeof(int));//避免竞争，动态分配内存块
    *connfd = accept(listenfd,(SA*)&clientaddr, &clientlen); //竞争
    pthread_create(&tid, NULL, thread, connfd);
}

void *thread(void *vargp) {
    int connfd = *((int *)vargp);// 竞争
    pthread_detach(pthread_self());
    free(vargp);//释放内存
    echo(connfd);
    close(connfd);
    return NULL;
}
```

#### 线程不安全
- 不保护共享变量的函数，例如多个线程对全局变量cnt++，PV
- 保持跨越多个调用状态的函数，例如当前调用结果依赖于前次调用的结果，结果保存为全局变量，调用者参数传参
- 返回指向静态变量的指针的函数，例如gethostname计算结果保存在static变量返回一个指向这个变量的指针，加锁复制将返回结果PV加锁至私有的内存位置
- 调用线程不安全函数的函数

#### 可重入性
被多个线程调用时，不会使用共享数据，线程安全函数的真子集

#### 死锁
互斥操作全序，每个线程一种顺序获得互斥锁，并相反顺序释放，则无死锁
