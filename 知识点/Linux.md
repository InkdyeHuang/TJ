<!-- GFM-TOC -->
* [cmake和makefile](#cmake和makefile)
* [cmake的执行过程](#cmake的执行过程)
* [线程池的原理及实现](#线程池的实现)
* [Linux网络编程](#linux网络编程)
    * [io复用](#io复用)
* [常用指令](#常用指令)
<!-- GFM-TOC -->

#### cmake和makefile
Makefile描述了整个工程的编译、连接等规则，Makefile 可以有效的减少编译和连接的程序，只编译和连接那些修改的文件。
Makefile的执行过程如下：
1. 在当前目录下寻找Makefile或makefile。
2. 找到第一个文件中的第一个目标文件，和目标文件依赖的.o文件。
3. 如果.o文件不存在，或是后面.o文件比target文件更新，那么它就会执行后面的语句来生成这个文件。
4.  最后makefile会根据.o文件依赖的.h和.c文件生成.o文件。

Cmke跨平台，可以更加简单的产生makefile，不用自己修改。

### cmake的执行过程
1. 编写CMake的配置文件CMakeLists.txt
2. 执行命令 cmake PATH 或者 ccmake PATH 生成 Makefile, PATH位CMakeLists.txt所在的目录。(ccmake有交互见面，cmake没有)
3. 使用make命令进行编译

### 线程池的原理及实现
多线程技术:解决处理器单元内多线程执行的问题，可以显著减少处理器单元的闲置时间，增加处理器单元的吞吐能力。

假设一个服务器完成一个任务的时间：T1——创建线程时间；T2——线程执行任务时间，T3销毁线程时间。若T1 + T3 远大于T2，则可采用线程池，提高服务器性能。

一个线程池模型一般有三个参与者：
1. 线程池结构：负责管理多个线程并提供任务队列的接口
2. 工作线程：负责处理任务
3. 任务队列：存放处理的任务(链表实现)
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <pthread.h>
#include <assert.h>
/*
*线程池里所有运行和等待的任务都是一个CThread_worker
*由于所有任务都在链表里，所以是一个链表结构
*/
typedef struct worker
{
    /*回调函数，任务运行时会调用此函数，注意也可声明成其它形式*/
    void *(*process) (void *arg);
    void *arg;/*回调函数的参数*/
    struct worker *next;
} CThread_worker;
/*线程池结构*/
typedef struct
{
     pthread_mutex_t queue_lock;
     pthread_cond_t queue_ready;
    /*链表结构，线程池中所有等待任务*/
     CThread_worker *queue_head;
    /*是否销毁线程池*/
    int shutdown;
     pthread_t *threadid;
    /*线程池中允许的活动线程数目*/
    int max_thread_num;
    /*当前等待队列的任务数目*/
    int cur_queue_size;
} CThread_pool;
//线程池的任务链表中加入一个任务，加入后通过调用pthread_cond_signal (&(pool->queue_ready))唤醒一个出于阻塞状态的线程(如果有的话)
int pool_add_worker (void *(*process) (void *arg), void *arg);
void *thread_routine (void *arg);
static CThread_pool *pool = NULL;
void pool_init (int max_thread_num)
{
     pool = (CThread_pool *) malloc (sizeof (CThread_pool));
     pthread_mutex_init (&(pool->queue_lock), NULL);
     pthread_cond_init (&(pool->queue_ready), NULL);
     pool->queue_head = NULL;
     pool->max_thread_num = max_thread_num;
     pool->cur_queue_size = 0;
     pool->shutdown = 0;
     pool->threadid = (pthread_t *) malloc (max_thread_num * sizeof (pthread_t));
    int i = 0;
    for (i = 0; i < max_thread_num; i++)
     {
         pthread_create (&(pool->threadid[i]), NULL, thread_routine,NULL);
     }
}
/*向线程池中加入任务*/
int pool_add_worker (void *(*process) (void *arg), void *arg)
{
    /*构造一个新任务*/
     CThread_worker *newworker =(CThread_worker *) malloc (sizeof (CThread_worker));
     newworker->process = process;
     newworker->arg = arg;
     newworker->next = NULL;/*别忘置空*/
     pthread_mutex_lock (&(pool->queue_lock));
    /*将任务加入到等待队列中*/
     CThread_worker *member = pool->queue_head;
    if (member != NULL)
     {
        while (member->next != NULL)
             member = member->next;
         member->next = newworker;
     }
    else
     {
         pool->queue_head = newworker;
     }
     assert (pool->queue_head != NULL);
     pool->cur_queue_size++;
     pthread_mutex_unlock (&(pool->queue_lock));
    /*好了，等待队列中有任务了，唤醒一个等待线程；
     注意如果所有线程都在忙碌，这句没有任何作用*/
     pthread_cond_signal (&(pool->queue_ready));
    return 0;
}
/*销毁线程池，等待队列中的任务不会再被执行，但是正在运行的线程会一直
把任务运行完后再退出*/
int pool_destroy ()
{
    if (pool->shutdown)
        return -1;/*防止两次调用*/
     pool->shutdown = 1;
    /*唤醒所有等待线程，线程池要销毁了*/
     pthread_cond_broadcast (&(pool->queue_ready));
    /*阻塞等待线程退出，否则就成僵尸了*/
    int i;
    for (i = 0; i < pool->max_thread_num; i++)
         pthread_join (pool->threadid[i], NULL);
     free (pool->threadid);
    /*销毁等待队列*/
     CThread_worker *head = NULL;
    while (pool->queue_head != NULL)
     {
         head = pool->queue_head;
         pool->queue_head = pool->queue_head->next;
         free (head);
     }
    /*条件变量和互斥量也别忘了销毁*/
     pthread_mutex_destroy(&(pool->queue_lock));
     pthread_cond_destroy(&(pool->queue_ready));
    
     free (pool);
    /*销毁后指针置空是个好习惯*/
     pool=NULL;
    return 0;
}
void *thread_routine (void *arg)
{
     printf ("starting thread 0x%x\n", pthread_self ());
     while (1)
     {
         pthread_mutex_lock (&(pool->queue_lock));
        /*如果等待队列为0并且不销毁线程池，则处于阻塞状态; 注意
         pthread_cond_wait是一个原子操作，等待前会解锁，唤醒后会加锁*/
        while (pool->cur_queue_size == 0 && !pool->shutdown)
         {
             printf ("thread 0x%x is waiting\n", pthread_self ());
             pthread_cond_wait (&(pool->queue_ready), &(pool->queue_lock));
         }
        /*线程池要销毁了*/
        if (pool->shutdown)
         {
            /*遇到break,continue,return等跳转语句，千万不要忘记先解锁*/
             pthread_mutex_unlock (&(pool->queue_lock));
             printf ("thread 0x%x will exit\n", pthread_self ());
             pthread_exit (NULL);
         }
         printf ("thread 0x%x is starting to work\n", pthread_self ());
        /*assert是调试的好帮手*/
         assert (pool->cur_queue_size != 0);
         assert (pool->queue_head != NULL);
        
        /*等待队列长度减去1，并取出链表中的头元素*/
         pool->cur_queue_size--;
         CThread_worker *worker = pool->queue_head;
         pool->queue_head = worker->next;
         pthread_mutex_unlock (&(pool->queue_lock));
        /*调用回调函数，执行任务*/
         (*(worker->process)) (worker->arg);
         free (worker);
         worker = NULL;
     }
    /*这一句应该是不可达的*/
     pthread_exit (NULL);
}
//测试

void * myprocess (void *arg)
{
        printf ("threadid is 0x%x, working on task %d\n", pthread_self (),*(int *) arg);
        sleep (1);/*休息一秒，延长任务的执行时间*/
        return NULL;
}
int main (int argc, char **argv)
{
        pool_init (3);/*线程池中最多三个活动线程*/

        /*连续向池中投入10个任务*/
        int *workingnum = (int *) malloc (sizeof (int) * 10);
        int i;
        for (i = 0; i < 10; i++)
        {
        workingnum[i] = i;
        pool_add_worker (myprocess, &workingnum[i]);
        }
        /*等待所有任务完成*/
        sleep (5);
        /*销毁线程池*/
        pool_destroy ();
        free (workingnum);
        return 0;
}
```
## Linux网络编程
### IO复用
IO复用：Linux中的IO模型之一，IO复用就是预先告诉内核需要监视的IO条件，一旦发现进程指定的一个或多个IO条件就绪，就通过进程处理，不会在单个IO上阻塞。

实现IO复用的三个接口函数：select、poll、epoll
1. select函数
```c
#include<sys/select.h>
#include<sys/time.h>
/*nfds：被监听的最大模式符个数，通常为监听的所有描述符最大值加1。
readdfs,writedfs,exceptfds:对应可读可写和异常等事件文件描述符集合，通过者三个参数传入文件描述符，select函数返回后，内核通过修改通知应用系统那些文件描述符就绪。fd_set结构体包含一个整形数组,容纳数量为FD_SETSIZE，所以数量有限制。
timeout：select设置的超时时间;timeval的成员都赋值0，则select将立即返回；如果timeout为NULL，则select将一直阻塞，直到某个文件描述符就绪
*/
int select(int nfs, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout))
```

select缺点：
- 每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大
- 每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大
- select支持的文件描述符数量太小了，默认是1024
2. poll函数：链表实现，文件描述符数量没有限制；与select类似，在一定时间内轮询一定数量的文件描述符，测试是否有就绪者。nfds参数指定被监听事件集合fds的大小，timeout指定poll的超时值，单位为毫秒，当timeout为-1时，poll调用将一直阻塞，直到某个事件发生；当timeout为0时，poll调用马上返回。
```c
#include <poll.h>  
int poll(struct pollfd *fds, nfds_t nfds, int timeout);  
```
3. epoll系列函数
epoll使用一组函数完成，而不是单个函数。如epoll_creat,epoll_ctl,epoll_wait;epoll把用户关心的文件描述符上的事件放在内核上的一个事件表中，从而无须像select和poll那样每次调用都要重复传入文件描述符集合事件表。但epoll需要使用一个额外的文件描述符。
基本原理：
epoll支持水平触发和边缘触发，最大的特点在于边缘触发，它只告诉进程哪些fd刚刚变为就绪态，并且只会通知一次。还有一个特点是，epoll使用“事件”的就绪通知方式，通过epoll_ctl注册fd，一旦该fd就绪，内核就会采用类似callback的回调机制来激活该fd，epoll_wait便可以收到通知。
epoll的优点：
- 没有最大并发连接的限制，能打开的FD的上限远大于1024（1G的内存上能监听约10万个端口）。
- 效率提升，不是轮询的方式，不会随着FD数目的增加效率下降,红黑树实现。
　　只有活跃可用的FD才会调用callback函数；即Epoll最大的优点就在于它只管你“活跃”的连接，而跟连接总数无关，因此在实际的网络环境中，Epoll的效率就会远远高于select和poll。
- 内存拷贝，利用mmap()文件映射内存加速与内核空间的消息传递；即epoll使用mmap减少复制开销。
https://www.cnblogs.com/luoxn28/p/6220372.html
## 常用指令
#### find指令
find pathname -options [-print -exec -ok]
-options: -name 按照文件名查找文件

         -perm 按文件权限查找文件

         -user 按文件属主查找文件

         -group  按照文件所属的组来查找文件。

         -type  查找某一类型的文件
命令参数：
        pathname: find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。
        -print： find命令将匹配的文件输出到标准输出。
        -exec： find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {  } \;，注意{   }和\;之间的空格。
        -ok： 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。
#### rm命令
rm -option filename
-option: -i 删除前逐一询问
         -rf 不用一一确认
         --
##### 删除当前文件夹下满足提交的所有文件
find / -name "test*"|xargs rm -rf
find / -name “test*” -exec rm -rf {} /;
rm -rf $(find / -name “test”)
