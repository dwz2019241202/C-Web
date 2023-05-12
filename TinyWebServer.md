# TinyWebServer



## lock

### 信号量sem

```
class sem
{
public:
    sem()
    {
        if (sem_init(&m_sem, 0, 0) != 0)
        {
            throw std::exception();
        }
    }
    sem(int num)
    {
        if (sem_init(&m_sem, 0, num) != 0)
        {
            throw std::exception();
        }
    }
    ~sem()
    {
        sem_destroy(&m_sem);
    }
    bool wait()
    {
        return sem_wait(&m_sem) == 0;
    }
    bool post()
    {
        return sem_post(&m_sem) == 0;
    }

private:
    sem_t m_sem;
};
```

#### 信号量类型 sem_t

#### 信号量初始化 

**sem_init(sem_t* sem, int pshared, unsigned int value)**

- 用于对信号量sem进行初始化
- pshared参数控制着信号量的类型：
  	如果 pshared的值是0，就表示它是当前进程的局部信号量
  	如果pshared的值不为0，其它进程就能够共享这个信号量
  	Linux线程一般不支持进程间共享信号量，pshared传递一个非零将会使函数返回ENOSYS错误
- value参数代表了sem的值
- 成功时返回0，否则返回-1

#### 信号量P()阻塞操作 

**sem_wait(sem_t* sem)**

- 用来等待信号量的值大于0（value > 0），等待时该线程为阻塞状态
- 解除阻塞后sem值会减去1
- 成功时返回0，否则返回-1

#### 信号量P()非阻塞操作 

**sem_trywait(sem_t* sem)**

- sem_wait()的非阻塞版本，直接将sem的值减去1
- 成功时返回0，否则返回-1

#### 信号量V()操作 

**sem_post(sem_t* sem)**

- 用来增加信号量的值（value + 1）
- 成功时返回0，否则返回-1

#### 释放信号量 

**sem_destroy(sem_t* sem)**

- 释放信号量sem



### 条件变量cond

#### 条件变量初始化

**int pthread_cond_init(pthread_cond_t *m_cond, const pthread_condattr_t *cattr);**

- 当参数cattr为[空指针](https://so.csdn.net/so/search?q=空指针&spm=1001.2101.3001.7020)时，函数创建的是一个缺省的条件变量
- 否则条件变量的属性将由cattr中的属性值来决定
- 这个函数返回时，条件变量被存放在参数m_cond指向的内存中。
- m_cond可以理解为一个队列，内部存储等待被唤醒的线程

#### 等待条件

**条件等待：int pthread_cond_wait(pthread_cond_t *m_cond, pthread_mutex_t * m_mutex)**

**计时等待：int pthread_cond_timedwait(pthread_cond_t *m_cond, pthread_mutex_t *m_mutex, const struct timespec *abstime)**

pthread_cond_wait函数用于等待目标条件变量.该函数调用时需要传入 **mutex参数(加锁的互斥锁)** ,函数执行时,先把调用线程放入条件变量的请求队列,然后将互斥锁mutex解锁,当函数成功返回为0时,互斥锁会再次被锁上. **也就是说函数内部会有一次解锁和加锁操作**.

无论哪种等待方式，都必须和一个[互斥锁](https://baike.baidu.com/item/互斥锁?fromModule=lemma_inlink)配合，以防止多个线程同时请求pthread_cond_wait()（或pthread_cond_timedwait()，下同）的[竞争条件](https://baike.baidu.com/item/竞争条件/10354815?fromModule=lemma_inlink)（Race Condition）。mutex在调用pthread_cond_wait()前必须由本线程[加锁](https://baike.baidu.com/item/加锁/53340052?fromModule=lemma_inlink)（[pthread_mutex_lock](https://baike.baidu.com/item/pthread_mutex_lock/8058751?fromModule=lemma_inlink)()），而在更新条件[等待队列](https://baike.baidu.com/item/等待队列/9223804?fromModule=lemma_inlink)以前，mutex保持锁定状态，并在线程挂起进入等待前解锁。在条件满足从而离开pthread_cond_wait()之前，mutex将被重新加锁，以与进入pthread_cond_wait()前的加锁动作对应。阻塞时处于解锁状态。

#### 激发条件

**激发一个线程：pthread_cond_signal(pthread_cond_t*  m_cond);**

**激发所有线程：pthread_cond_broadcast(pthread_cond_t*  m_cond);**

```
#include<pthread.h>
#include<unistd.h>
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
 
static pthread_mutex_t mtx=PTHREAD_MUTEX_INITIALIZER;
static pthread_cond_t cond=PTHREAD_COND_INITIALIZER;
 
struct node {
    int n_number;
    struct node *n_next;
} *head=NULL; /*[thread_func]*/
 
/*释放节点内存*/
static void cleanup_handler(void*arg) {
    printf("Clean up handler of second thread.\n");
    free(arg);
    (void)pthread_mutex_unlock(&mtx);
}
 
static void *thread_func(void *arg) {
    struct node*p=NULL;
    pthread_cleanup_push(cleanup_handler,p);
 
    pthread_mutex_lock(&mtx);
    //这个mutex_lock主要是用来保护wait等待临界时期的情况，
    //当在wait为放入队列时，这时，已经存在Head条件等待激活
    //的条件，此时可能会漏掉这种处理
    //这个while要特别说明一下，单个pthread_cond_wait功能很完善，
    //为何这里要有一个while(head==NULL)呢？因为pthread_cond_wait
    //里的线程可能会被意外唤醒，如果这个时候head==NULL，
    //则不是我们想要的情况。这个时候，
    //应该让线程继续进入pthread_cond_wait
    while(1) {
        while(head==NULL) {
            pthread_cond_wait(&cond,&mtx);
        }
        //pthread_cond_wait会先解除之前的pthread_mutex_lock锁定的mtx，
        //然后阻塞在等待队列里休眠，直到再次被唤醒
        //（大多数情况下是等待的条件成立而被唤醒，唤醒后，
        //该进程会先锁定先pthread_mutex_lock(&mtx);，
        //再读取资源用这个流程是比较清楚的
        /*block-->unlock-->wait()return-->lock*/
        p=head;
        head=head->n_next;
        printf("Got%dfromfrontofqueue\n",p->n_number);
        free(p);
    }
    pthread_mutex_unlock(&mtx);//临界区数据操作完毕，释放互斥锁
 
    pthread_cleanup_pop(0);
    return 0;
}
 
int main(void) {
    pthread_t tid;
    int i;
    struct node *p;
    pthread_create(&tid,NULL,thread_func,NULL);
    //子线程会一直等待资源，类似生产者和消费者，
    //但是这里的消费者可以是多个消费者，
    //而不仅仅支持普通的单个消费者，这个模型虽然简单，
    //但是很强大
    for(i=0;i<10;i++) {
        p=(struct node*)malloc(sizeof(struct node));
        p->n_number=i;
        pthread_mutex_lock(&mtx);//需要操作head这个临界资源，先加锁，
        p->n_next=head;
        head=p;
        pthread_cond_signal(&cond);
        pthread_mutex_unlock(&mtx);//解锁
        sleep(1);
    }
    printf("thread1wannaendthecancelthread2.\n");
    pthread_cancel(tid);
    //关于pthread_cancel，有一点额外的说明，它是从外部终止子线程，
    //子线程会在最近的取消点，退出线程，而在我们的代码里，最近的
    //取消点肯定就是pthread_cond_wait()了。
    pthread_join(tid,NULL);
    printf("Alldone--exiting\n");
    return 0;
}
```



## log

### 阻塞队列block_queue

#### 终止进程exit(int statu)

C 库函数 **void exit(int status)** 立即终止调用进程。任何属于该进程的打开的文件描述符都会被关闭，该进程的子进程由进程 1 继承，初始化，且会向父进程发送一个 SIGCHLD 信号。**status**是返回给父进程的状态值。

#### 获取系统时间

**int gettimeofday(struct  timeval*tv,struct  timezone *tz )**

这个函数会把时间包装为一个结构体返回。包括秒，微妙，时区等信息，成功时返回0。



### 日志类定义

#### 创建线程

```
int pthread_create(pthread_t *thread,
                   const pthread_attr_t *attr,
                   void *(*start_routine) (void *),
                   void *arg);
```

各个参数的含义是：

1) pthread_t *thread：传递一个 pthread_t 类型的指针变量，也可以直接传递某个 pthread_t 类型变量的地址。pthread_t 是一种用于表示线程的数据类型，每一个 pthread_t 类型的变量都可以表示一个线程。

2) const pthread_attr_t *attr：用于手动设置新建线程的属性，例如线程的调用策略、线程所能使用的栈内存的大小等。大部分场景中，我们都不需要手动修改线程的属性，将 attr 参数赋值为 NULL，pthread_create() 函数会采用系统默认的属性值创建线程。

pthread_attr_t 类型以结构体的形式定义在`<pthread.h>`头文件中，此类型的变量专门表示线程的属性。

3) void *(*start_routine) (void *)：以函数指针的方式指明新建线程需要执行的函数，该函数的参数最多有 1 个（可以省略不写），形参和返回值的类型都必须为 void* 类型。void* 类型又称空指针类型，表明指针所指数据的类型是未知的。使用此类型指针时，我们通常需要先对其进行强制类型转换，然后才能正常访问指针指向的数据。

4) void *arg：指定传递给 start_routine 函数的实参，当不需要传递任何数据时，将 arg 赋值为 NULL 即可。





## 字符串操作

[<string.h>链接](https://www.runoob.com/cprogramming/c-standard-library-string-h.html)

```
void *memset(void *str, int c, size_t n);
填充c字符到str的前n个位置；常用于
str -- 指向要填充的内存块。
c -- 要被设置的值。该值以 int 形式传递，但是函数在填充内存块时是使用该值的char形式。
n -- 要被设置为该值的字符数。
```

```
char *strrchr(const char *str, int c)
从后往前寻找第一次出现的字符c
str -- C 字符串。
c -- 要搜索的字符。以 int 形式传递，但是最终会转换回 char 形式。
```

```
int snprintf(char *str, size_t size, const char *format, ...)
将可变参数 ...” 按照format的格式格式化为字符串，然后再将其拷贝至str中
如果格式化后的字符串长度 < size，则将此字符串全部复制到str中，并给其后添加一个字符串结束符(’\0’)；
如果格式化后的字符串长度 >= size，则只将其中的(size-1)个字符复制到str中，并给其后添加一个字符串结束符(’\0’)，返回值为欲写入的字符串长度。
注意：无论格式化后的字符串长度与size的大小是如何，都会在后面添加一个字符串结束符(’\0’)

示例：
snprintf(log_full_name, 255, "%d_%02d_%02d_%s", my_tm.tm_year + 1900, my_tm.tm_mon + 1, my_tm.tm_mday, file_name);
```



## 时间操作

```
//时间戳（时间结构体）
time_t: 存储从1970年到现在经过了多少秒。格式为long int。UTC时间。

struct timeval {
               time_t      tv_sec;            /* seconds */
               suseconds_t tv_usec;    /* microseconds 微秒*/
};

struct timespec {
    			time_t tv_sec;		/* Seconds.  */
    			syscall_slong_t tv_nsec;	/* Nanoseconds 纳秒 */
};

struct tm
{
  int tm_sec;			/* Seconds.	[0-60] (1 leap second) */
  int tm_min;			/* Minutes.	[0-59] */
  int tm_hour;			/* Hours.	[0-23] */
  int tm_mday;			/* Day.		[1-31] */
  int tm_mon;			/* Month.	[0-11] */
  int tm_year;			/* Year	- 1900.  */
  int tm_wday;			/* Day of week.	[0-6] */
  int tm_yday;			/* Days in year.[0-365]	*/
  int tm_isdst;			/* DST.		[-1/0/1]*/
 
# ifdef	__USE_MISC
  long int tm_gmtoff;		/* Seconds east of UTC.  */
  const char *tm_zone;		/* Timezone abbreviation.  */
# else
  long int __tm_gmtoff;		/* Seconds east of UTC.  */
  const char *__tm_zone;	/* Timezone abbreviation.  */
# endif
};

```

```
//时间函数
time_t time(time_t * timer)
timer=NULL时得到机器日历时间；
timer=时间数值时，用于设置日历时间；

struct tm *localtime(const time_t *timer)
根据time_t结构时间生成tm结构时间


```





## 文件操作

```
FILE *fopen(char *filename, *type);
打开文件，返回文件描述符
第二个形式参数表示打开文件的类型
 		   "r"           打开文字文件只读
           "w"           创建文字文件只写
           "a"           增补, 如果文件不存在则创建一个
           "r+"          打开一个文字文件读/写
           "w+"          创建一个文字文件读/写
           "a+"          打开或创建一个文件增补
           "b"           二进制文件(可以和上面每一项合用)
           "t"           文这文件(默认项)
```

