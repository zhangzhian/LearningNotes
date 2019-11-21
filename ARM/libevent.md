# libevent文档

[官方网站](https://libevent.org/)

[官方文档](https://libevent.org/doc/)

[官方GitHub](https://github.com/libevent/libevent)

[libevent-2.1.11-stable.tar.gz](https://github.com/libevent/libevent/releases/download/release-2.1.11-stable/libevent-2.1.11-stable.tar.gz)

## 前言

Libevent是用于开发可伸缩网络服务器的事件通知库。Libevent API提供了一种机制，在文件描述符上发生特定事件或达到超时后执行回调函数。此外，Libevent还支持由于信号或定期超时而进行的回调。make

Libevent用于取代在事件驱动的网络服务器中的事件的循环。应用程序只需调用[event_base_dispatch（）](https://libevent.org/doc/event_8h.html#a19d60cb72a1af398247f40e92cf07056)，然后动态添加或删除事件，而无需更改事件循环。

目前，Libevent支持/dev/poll、kqueue（2）、select（2）、poll（2）、epoll（4）和evports。内部事件机制完全独立于公开的事件API，简单的Libevent更新可以提供新的功能，而无需重新设计应用程序。因此，Libevent允许进行可移植的应用程序开发，并提供操作系统上可用的最可伸缩的事件通知机制。Libevent也可以用于多线程程序。Libevent应该在Linux、*BSD、Mac OS X、Solaris和Windows上编译。

其**设计目标**是： 

- 可移植性：使用 libevent 编写的程序应该可以在 libevent 支持的所有平台上工作。即使 没有好的方式进行非阻塞 IO，libevent 也应该支持一般的方式，让程序可以在受限的环境中运行。 

- 速度：libevent 尝试使用每个平台上最高速的非阻塞 IO 实现，并且不引入太多的额外开销。 

- 可扩展性：libevent 被设计为程序即使需要上万个活动套接字的时候也可以良好工作。 

- 方便：无论何时，最自然的使用 libevent 编写程序的方式应该是稳定的、可移植的。

libevent 由**下列组件构成**：

-  evutil：用于抽象不同平台网络实现差异的通用功能。 

- event 和 event_base：libevent 的核心，为各种平台特定的、基于事件的非阻塞 IO 后 端提供抽象 API，让程序可以知道套接字何时已经准备好，可以读或者写，并且处理基 本的超时功能，检测 OS 信号。 

- bufferevent：为 libevent 基于事件的核心提供使用更方便的封装。除了通知程序套接字 已经准备好读写之外，还让程序可以请求缓冲的读写操作，可以知道何时 IO 已经真正 发生。 

- evbuffer：在 bufferevent 层之下实现了缓冲功能，并且提供了方便有效的访问函数。 

- evhttp：一个简单的 HTTP 客户端/服务器实现。 

- evdns：一个简单的 DNS 客户端/服务器实现。 

- evrpc：一个简单的 RPC 实现。

## 编译

### linux

```shell
$ ./configure
$ make
$ make verify   # (optional)
$ sudo make install
```

### ARM

```shell
$ ./configure --host=arm-linux --prefix=/usr/local/libevent_arm CC=arm-none-linux-gnueabi-gcc CXX=arm-none-linux-gnueabi-g++
$ make
$ make verify   # (optional)
$ sudo make install
```

[Building and installing Libevent](https://github.com/libevent/libevent/blob/master/Documentation/Building.md#autoconf)

### 生成库

创建 libevent 时，默认安装下列库： 

- ibevent_core：所有核心的事件和缓冲功能，包含了所有的 event_base、evbuffer、 bufferevent 和工具函数。 

- ibevent_extra：定义了程序可能需要，也可能不需要的协议特定功能，包括 HTTP、 DNS 和 RPC。 

- libevent：这个库因为历史原因而存在，它包含 libevent_core 和 libevent_extra 的内容。 不应该使用这个库，未来版本的 libevent 可能去掉这个库。 

某些平台上可能安装下列库： 

- libevent_pthreads：添加基于 pthread 可移植线程库的线程和锁定实现。它独立于 libevent_core，这样程序使用 libevent 时就不需要链接到 pthread，除非实际上是以多线程的方式使用libevent。 

- libevent_openssl：这个库为使用 bufferevent 和 OpenSSL 进行加密的通信提供支持。 它独立于 libevent_core，这样程序使用 libevent 时就不需要链接到 OpenSSL，除非实际使用的是加密连接。 

## 概述

### 标准用法

使用Libevent的每个程序都必须包含[<event2/event.h>](https://libevent.org/doc/event_8h.html)头文件，并将-levent传递给链接器。（如果只需要主事件和缓冲的基于IO的代码，而不想链接任何协议代码，则可以改为链接-levent_core。）



### 库设置

在调用任何其他Libevent函数之前，需要设置库。如果要在多线程应用程序的多个线程中使用Libevent，则需要初始化线程支持：通常使用[evthread_use_pthreads（）](https://libevent.org/doc/thread_8h.html#acc0cc708c566c14f4659331ec12f8a5b)或[evthread_use_windows_threads（）](https://libevent.org/doc/thread_8h.html#a1b0fe36dcb033da2c679d39ce8a190e2)。有关更多信息，请参见[<event2/thread.h>](https://libevent.org/doc/thread_8h.html)。

这也是可以用event_set_mem_functions替换Libevent的内存管理函数，并用[event_enable_debug_mode（）](https://libevent.org/doc/event_8h.html#a37441a3defac55b5d2513521964b2af5)启用调试模式的地方。

### 创建event base

接下来，需要使用[event_base_new（）](https://libevent.org/doc/event_8h.html#af34c025430d445427a2a5661082405c3)或[event_base_new_with_config（）](https://libevent.org/doc/event_8h.html#a864798e5b3c6aa8d2f9e9c5035a3e6da)创建一个[event_base](https://libevent.org/doc/structevent__base.html)结构体。event_base负责跟踪哪些事件是“待处理的”（也就是说，被监视以查看它们是否变为活动的）以及哪些事件是“活动的”。每个事件都与单个[event_base](https://libevent.org/doc/structevent__base.html)相关联。

### 事件通知

对于要监视的每个文件描述符，必须使用[event_new（）](https://libevent.org/doc/event_8h.html#a6cd300e5cd15601c5e4570b221b20397)创建事件结构。（您也可以声明一个事件结构并调用[event_assign（）](https://libevent.org/doc/event_8h.html#a3e49a8172e00ae82959dfe64684eda11)来初始化该结构的成员。）要启用通知，您可以通过调用[event_add（）](https://libevent.org/doc/event_8h.html#ab0c85ebe9cf057be1aa17724c701b0c8)将该结构添加到监视的事件列表中。只要事件结构处于活动状态，它就必须保持分配状态，因此通常应该在堆上分配它。

### 调度事件。

最后，调用[event_base_dispatch（）](https://libevent.org/doc/event_8h.html#a19d60cb72a1af398247f40e92cf07056)循环和分派事件。您还可以使用[event_base_loop（）](https://libevent.org/doc/event_8h.html#acd7da32086d1e37008e7c76c9ea54fc4)进行更细粒度的控制。

目前，一次只能有一个线程调度给定的[event_base](https://libevent.org/doc/structevent__base.html) 。如果希望一次在多个线程中运行事件，可以有一个将工作添加到工作队列中的[event_base](https://libevent.org/doc/structevent__base.html) ，也可以创建多个[event_base](https://libevent.org/doc/structevent__base.html)对象。

### I/O缓冲区

Libevent在常规事件回调的基础上提供了缓冲I/O抽象。这个抽象称为bufferevent。bufferevent提供自动填充和移除的输入和输出缓冲区。缓冲事件的用户不再直接处理I/O，而是读取输入和写入输出缓冲区。

一旦通过[bufferevent_socket_new（）](https://libevent.org/doc/bufferevent_8h.html#a71181be5ab504e26f866dd3d91494854)初始化，bufferevent结构就可以与[bufferevent_enable（）](https://libevent.org/doc/bufferevent_8h.html#aa8a5dd2436494afd374213b99102265b)和[bufferevent_disable（）](https://libevent.org/doc/bufferevent_8h.html#a4f3974def824e73a6861d94cff71e7c6)一起重复使用。您将调用[bufferevent_read（）](https://libevent.org/doc/bufferevent_8h.html#a9e36c54f6b0ea02183998d5a604a00ef)和[bufferevent_write（）](https://libevent.org/doc/bufferevent_8h.html#a7873bee379202ca1913ea365b92d2ed1)，而不是直接读取和写入套接字。

启用读取后，bufferevent将尝试从文件描述符读取并调用读取回调。每当输出缓冲区被排放到写低水位线（默认为0）以下时，就会执行写回调。

有关更多信息，请参阅<event2/bufferevent*.h>。

### 计时器

Libevent还可以用来创建计时器，在一定时间过期后调用回调。evtimer_new（）宏返回要用作计时器的事件结构。要激活计时器，请调用evtimer_add（）。可以通过调用evtimer_del（）来停用计时器。（这些宏是围绕[event_new（）](https://libevent.org/doc/event_8h.html#a6cd300e5cd15601c5e4570b221b20397)、[event_add（）](https://libevent.org/doc/event_8h.html#ab0c85ebe9cf057be1aa17724c701b0c8)和[event_del（）](https://libevent.org/doc/event_8h.html#a2ffc10a652c67bea16b0a71f7d5a44dc)的精简封装；您也可以使用它们。）

### 异步DNS解析

Libevent提供了一个异步DNS解析器，应该使用它来代替标准DNS解析器函数。有关更多详细信息，请参阅[<event2/dns.h>](https://libevent.org/doc/dns_8h.html)函数。

### 事件驱动的HTTP服务器

Libevent提供了一个非常简单的事件驱动HTTP服务器，可以嵌入到程序中并用于服务HTTP请求。

要使用此功能，您需要在程序中包含[<event2/http.h>](https://libevent.org/doc/http_8h.html)头。有关详细信息，请参阅该标题。

### RPC服务器和客户机的框架

Libevent提供了一个创建RPC服务器和客户端的框架。它负责对所有数据结构进行编组和解组。

### API参考

要浏览libevent API的完整文档，请单击以下任何链接。

[event2/event.h](https://libevent.org/doc/event_8h.html) ：主libevent头

[event2/thread.h](https://libevent.org/doc/thread_8h.html) ：多线程程序

[event2/buffer.h](https://libevent.org/doc/buffer_8h.html) 和[event2/bufferevent.h](https://libevent.org/doc/bufferevent_8h.html) ：网络读写的缓冲区管理

[event2/util.h](https://libevent.org/doc/util_8h.html) ： 可移植非阻塞网络代码的实用函数

[event2/dns.h](https://libevent.org/doc/dns_8h.html)  ：异步DNS解析

[event2/http.h](https://libevent.org/doc/http_8h.html)  ：基于libevent的嵌入式HTTP服务器

[event2/rpc.h](https://libevent.org/doc/rpc_8h.html)  ：创建RPC服务器和客户端的框架

[event2/watch.h](https://libevent.org/doc/watch_8h.html)  ：“准备”和“检查”观察者



## 详细说明

基于Libevent 2.0+，在C语言中编写快速可移植的异步网络IO程序。

设计目标是：

### 一、设置libevent库

Libevent具有一些在整个进制中共享的会影响整个库的全局设置。

在调用Libevent库的任何其他部分之前，必须对这些设置进行任何更改。 如果您不这样做，Libevent可能会处于不一致状态。

#### 1. Libevent中的日志消息

Libevent可以记录内部错误和警告。 如果它是在日志支持下编译的，它还会记录调试消息。 默认情况下，这些消息将写入stderr。 可以通过提供自己的日志记录功能来覆盖此行为。

**接口**

```c
#define EVENT_LOG_DEBUG 0
#define EVENT_LOG_MSG   1
#define EVENT_LOG_WARN  2
#define EVENT_LOG_ERR   3

/* Deprecated; see note at the end of this section */
#define _EVENT_LOG_DEBUG EVENT_LOG_DEBUG
#define _EVENT_LOG_MSG   EVENT_LOG_MSG
#define _EVENT_LOG_WARN  EVENT_LOG_WARN
#define _EVENT_LOG_ERR   EVENT_LOG_ERR

typedef void (*event_log_cb)(int severity, const char *msg);

void event_set_log_callback(event_log_cb cb);
```

要覆盖Libevent的日志记录行为，请编写与event_log_cb参数匹配的自己的函数，并将其作为参数传递给event_set_log_callback（）。 只要Libevent想要记录一条消息，它将把它传递给该函数。 可以通过以NULL作为参数再次调用event_set_log_callback（）来使Libevent恢复其默认行为。

**示例**

```c
#include <event2/event.h>
#include <stdio.h>

static void discard_cb(int severity, const char *msg)
{
    /* This callback does nothing. */
}

static FILE *logfile = NULL;
static void write_to_file_cb(int severity, const char *msg)
{
    const char *s;
    if (!logfile)
        return;
    switch (severity) {
        case _EVENT_LOG_DEBUG: s = "debug"; break;
        case _EVENT_LOG_MSG:   s = "msg";   break;
        case _EVENT_LOG_WARN:  s = "warn";  break;
        case _EVENT_LOG_ERR:   s = "error"; break;
        default:               s = "?";     break; /* never reached */
    }
    fprintf(logfile, "[%s] %s\n", s, msg);
}

/* Turn off all logging from Libevent. */
void suppress_logging(void)
{
    event_set_log_callback(discard_cb);
}

/* Redirect all Libevent log messages to the C stdio file 'f'. */
void set_logfile(FILE *f)
{
    logfile = f;
    event_set_log_callback(write_to_file_cb);
}
```

**注意**

在用户提供的event_log_cb回调中调用Libevent函数是不安全的！ 例如，如果您尝试编写一个使用bufferevents向网络套接字发送警告消息的日志回调，则很可能会遇到奇怪且难以诊断的错误。 在将来的Libevent版本中，某些功能可能会取消此限制。

通常，调试日志不会启用，并且不会发送到日志回调。 如果Libevent要支持它们，则可以手动将其打开。

**接口**

```c
#define EVENT_DBG_NONE 0
#define EVENT_DBG_ALL 0xffffffffu

void event_enable_debug_logging(ev_uint32_t which);
```

调试日志很冗长，在大多数情况下不一定有用。 使用EVENT_DBG_NONE调用event_enable_debug_logging（）将获得默认行为。 使用EVENT_DBG_ALL调用它会打开所有受支持的调试日志。 将来的版本中可能会支持更多细致的选项。

这些函数在<event2 / event.h>中声明。 它们首先出现在Libevent 1.0c中，除event_enable_debug_logging（）首次出现在Libevent 2.1.1-alpha中。

**兼容性说明**

在Libevent 2.0.19-稳定版本之前，EVENT_LOG_ *宏的名称以下划线开头：_EVENT_LOG_DEBUG，_EVENT_LOG_MSG，_EVENT_LOG_WARN和_EVENT_LOG_ERR。 这些较早的名称已被弃用，仅应用于与Libevent 2.0.18-stable和更早版本的向后兼容。 在将来的Libevent版本中可能会删除它们。

#### 2. 处理致命错误

当Libevent检测到不可恢复的内部错误（例如，损坏的数据结构）时，其默认行为是调用exit（）或abort（）退出当前运行的进程。 这些错误几乎总是意味着某个地方存在bug：在你的代码中，或者在Libevent本身中。

如果您希望应用程序更优雅地处理致命错误，则可以通过提供Libevent代替退出而调用的函数来覆盖Libevent的行为。

**接口**

```c
typedef void (*event_fatal_cb)(int err);
void event_set_fatal_callback(event_fatal_cb cb);
```

要使用这些函数，首先定义一个新函数，遇到致命错误时Libevent应该调用该函数，然后将其传递给event_set_fatal_callback（）。以后，如果Libevent遇到致命错误，它将调用您提供的函数。

该函数不应将控制权返回给Libevent。这样做可能会导致不确定的行为，并且Libevent可能仍会退出以避免崩溃。一旦调用了函数，就不应调用任何其他Libevent函数。

这些函数在<event2 / event.h>中声明。首先出现在Libevent 2.0.3-alpha中。

#### 3. 内存管理
默认情况下，Libevent使用C库的内存管理功能从堆中分配内存。您可以通过提供自己的malloc，realloc和free替换项来使Libevent使用另一个内存管理器。如果有想要使用Libevent的更高效的分配器，或者如果想要使用Libevent的检测化分配器来查找内存泄漏，则可能需要这样做。

**接口**

```c
void event_set_mem_functions(void *(*malloc_fn)(size_t sz),
                             void *(*realloc_fn)(void *ptr, size_t sz),
                             void (*free_fn)(void *ptr));
```

这是一个简单的示例，该示例将Libevent的分配函数替换为对已分配的字节总数进行计数的变体。 实际上，您可能希望在此处添加锁，以防止Libevent在多个线程中运行时出错。

**示例**

```c
#include <event2/event.h>
#include <sys/types.h>
#include <stdlib.h>

/* This union's purpose is to be as big as the largest of all the
 * types it contains. */
union alignment {
    size_t sz;
    void *ptr;
    double dbl;
};
/* We need to make sure that everything we return is on the right
   alignment to hold anything, including a double. */
#define ALIGNMENT sizeof(union alignment)

/* We need to do this cast-to-char* trick on our pointers to adjust
   them; doing arithmetic on a void* is not standard. */
#define OUTPTR(ptr) (((char*)ptr)+ALIGNMENT)
#define INPTR(ptr) (((char*)ptr)-ALIGNMENT)

static size_t total_allocated = 0;
static void *replacement_malloc(size_t sz)
{
    void *chunk = malloc(sz + ALIGNMENT);
    if (!chunk) return chunk;
    total_allocated += sz;
    *(size_t*)chunk = sz;
    return OUTPTR(chunk);
}
static void *replacement_realloc(void *ptr, size_t sz)
{
    size_t old_size = 0;
    if (ptr) {
        ptr = INPTR(ptr);
        old_size = *(size_t*)ptr;
    }
    ptr = realloc(ptr, sz + ALIGNMENT);
    if (!ptr)
        return NULL;
    *(size_t*)ptr = sz;
    total_allocated = total_allocated - old_size + sz;
    return OUTPTR(ptr);
}
static void replacement_free(void *ptr)
{
    ptr = INPTR(ptr);
    total_allocated -= *(size_t*)ptr;
    free(ptr);
}
void start_counting_bytes(void)
{
    event_set_mem_functions(replacement_malloc,
                            replacement_realloc,
                            replacement_free);
}
```

**注意**

- 替换内存管理功能会影响以后所有从Libevent分配，调整大小或释放内存的调用。因此，在调用任何其他Libevent函数之前，需要确保已替换函数。否则，Libevent将使用你的free版本释放从C库的malloc版本分配的内存。

- 你的malloc和realloc函数需要返回与C库相同的内存块。

- 你的realloc函数需要正确处理realloc（NULL，sz）（即，将其视为malloc（sz））。

- 你的realloc函数需要正确处理realloc（ptr，0）（也就是说，将其视为free（ptr））。

- 你的free函数不需要处理free（NULL）。

- 你的malloc函数不需要处理malloc（0）。

- 如果从多个线程中使用Libevent，则替换后的内存管理功能必须是线程安全的。

- Libevent将使用这些函数来分配返回给您的内存。因此，如果要释放由Libevent函数分配和返回的内存，并且已经替换了malloc和realloc函数，则可能必须使用替换free函数来释放它。

在<event2 / event.h>中声明了event_set_mem_functions（）函数。它首先出现在Libevent 2.0.1-alpha中。

可以在禁用event_set_mem_functions（）的情况下构建Libevent。如果是这样，则使用event_set_mem_functions的程序将不会编译或链接。在Libevent 2.0.2-alpha和更高版本中，您可以通过检查是否已定义EVENT_SET_MEM_FUNCTIONS_IMPLEMENTED宏来检测是否存在event_set_mem_functions（）。

#### 4. 锁和线程
编写多线程程序，在多个线程同时访问相同的数据并不总是安全的。

Libevent结构通常可以在多线程中以三种方式工作。

- 有些结构本质上是单线程的：从多个线程同时使用它们永远是不安全的。

- 某些结构是有可选的锁：可以告诉每个对象Libevent是否需要在多线程使用每个对象。

- 某些结构始终是锁定的：如果Libevent在锁定支持下运行，则始终可以安全地多个线程使用它们。

为获取锁，在调用分配需要在多个线程间共享的结构体的 libevent 函数之前，必须告知 libevent 使用哪个锁函数。

如果使用 pthreads 库，或者使用 Windows 本地线程代码，那么已经有设置 libevent 使用正确的 pthreads 或者 Windows 函数的预定义函数。 

**接口**

```c
#ifdef WIN32
int evthread_use_windows_threads(void);
#define EVTHREAD_USE_WINDOWS_THREADS_IMPLEMENTED
#endif
#ifdef _EVENT_HAVE_PTHREADS
int evthread_use_pthreads(void);
#define EVTHREAD_USE_PTHREADS_IMPLEMENTED
#endif
```

这些函数在成功时都返回0，失败时返回-1。 

如果使用不同的线程库，则需要一些额外的工作，必须使用你的线程库来定义函数去实现： 

- 锁 

- 锁定 

- 解锁 

- 分配锁 

- 析构锁 

- 条件变量 

- 创建条件变量 

- 析构条件变量 

- 等待条件变量 

- 触发/广播某条件变量 

- 线程 

- 线程 ID 检测 

使用 evthread_set_lock_callbacks 和 evthread_set_id_callback 接口告知 libevent 这些函 

数。

**接口**

```c
#define EVTHREAD_WRITE  0x04
#define EVTHREAD_READ   0x08
#define EVTHREAD_TRY    0x10

#define EVTHREAD_LOCKTYPE_RECURSIVE 1
#define EVTHREAD_LOCKTYPE_READWRITE 2

#define EVTHREAD_LOCK_API_VERSION 1

struct evthread_lock_callbacks {
       int lock_api_version;
       unsigned supported_locktypes;
       void *(*alloc)(unsigned locktype);
       void (*free)(void *lock, unsigned locktype);
       int (*lock)(unsigned mode, void *lock);
       int (*unlock)(unsigned mode, void *lock);
};

int evthread_set_lock_callbacks(const struct evthread_lock_callbacks *);

void evthread_set_id_callback(unsigned long (*id_fn)(void));

struct evthread_condition_callbacks {
        int condition_api_version;
        void *(*alloc_condition)(unsigned condtype);
        void (*free_condition)(void *cond);
        int (*signal_condition)(void *cond, int broadcast);
        int (*wait_condition)(void *cond, void *lock,
            const struct timeval *timeout);
};

int evthread_set_condition_callbacks(
        const struct evthread_condition_callbacks *);
```

evthread_lock_callbacks 结 构 体 描 述 的 锁 回 调 函 数 及 其 能 力 。 对 于 上 述 版 本 ， 

lock_api_version 字 段 必 须 设 置 为 EVTHREAD_LOCK_API_VERSION 。 必 须 设 置 

supported_locktypes 字段为 EVTHREAD_LOCKTYPE_*常量的组合以描述支持的锁类型 

（ 在 2.0.4-alpha 版 本 中 ， EVTHREAD_LOCK_RECURSIVE 是 必 须 的 ， 

EVTHREAD_LOCK_READWRITE 则没有使用）。alloc 函数必须返回指定类型的新锁；free 

函数必须释放指定类型锁持有的所有资源；lock 函数必须试图以指定模式请求锁定，如果成 

功则返回0，失败则返回非零；unlock 函数必须试图解锁，成功则返回0，否则返回非零。 

**可识别的锁类型有：** 

- 0：通常的，不必递归的锁。 

- EVTHREAD_LOCKTYPE_RECURSIVE：不会阻塞已经持有它的线程的锁。一旦持有它的线程进行原来锁定次数的解锁，其他线程立刻就可以请求它了。 

- EVTHREAD_LOCKTYPE_READWRITE：可以让多个线程同时因为读而持有它，但是任何时刻只有一个线程因为写而持有它。写操作排斥所有读操作。 

**可识别的锁模式有：** 

- EVTHREAD_READ：仅用于读写锁：为读操作请求或者释放锁 

- EVTHREAD_WRITE：仅用于读写锁：为写操作请求或者释放锁 

- EVTHREAD_TRY：仅用于锁定：仅在可以立刻锁定的时候才请求锁定 

id_fn 参数必须是一个函数，它返回一个无符号长整数，标识调用此函数的线程。对于相同线程，这个函数应该总是返回同样的值；而对于同时调用该函数的不同线程，必须返回不同的值。evthread_condition_callbacks 结构体描述了与条件变量相关的回调函数。对于上述版本，condition_api_version 字 段 必 须 设 置 为EVTHREAD_CONDITION_API_VERSION 。alloc_condition 函数必须返回到新条件变量的指针。它接受0作为其参数。free_condition 函数必须释放条件变量持有的存储器和资源。wait_condition 函数要求三个参数：一个由alloc_condition 分配的条件变量，一个由你提供的 evthread_lock_callbacks.alloc 函数分配的锁，以及一个可选的超时值。调用本函数时，必须已经持有参数指定的锁；本函数应该释放指定的锁，等待条件变量成为授信状态，或者直到指定的超时时间已经流逝（可选 ）。wait_condition 应该在错误时返回-1，条件变量授信时返回0，超时时返回1。返回之前，函数应该确定其再次持有锁。最后，signal_condition 函数应该唤醒等待该条件变量的某个线程（broadcast 参数为 false 时），或者唤醒等待条件变量的所有线程（broadcast 参数为 true 时）。只有在持有与条件变量相关的锁的时候，才能够进行这些操作。 

关于条件变量的更多信息，请查看 pthreads 的 pthread_cond_*函数文档，或者 Windows的 CONDITION_VARIABLE（Windows Vista 新引入的）函数文档。

**示例**

关于使用这些函数的示例，请查看 Libevent 源代码发布版本中的 evthread_pthread.c 和 evthread_win32.c 文件。 

这些函数在<event2/thread.h>中声明，其中大多数在 2.0.4-alpha 版本中首次出现。2.0.1-alpha 到2.0.3-alpha 使用较老版本的锁函数。event_use_pthreads 函数要求程序链接event_pthreads 库。 

条件变量函数是2.0.7-rc 版本新引入的，用于解决某些棘手的死锁问题。 

可以创建禁止锁支持的 libevent。这时候已创建的使用上述线程相关函数的程序将不能运行。

**5.** **调试锁的使用** 

为帮助调试锁的使用，libevent 有一个可选的“锁调试”特征。这个特征包装了锁调用，以便捕获典型的锁错误，包括： 

- 解锁并没有持有的锁 

- 重新锁定一个非递归锁 

如果发生这些错误中的某一个，libevent 将给出断言失败并且退出。 

**接口**

```c
void evthread_enable_lock_debugging(void);
#define evthread_enable_lock_debuging() evthread_enable_lock_debugging()
```

**注意** 

必须在创建或者使用任何锁之前调用这个函数。为安全起见，请在设置完线程函数后立即调用这个函数。 

此功能是Libevent 2.0.4-alpha中的新增功能，拼写错误的名称为“ evthread_enable_lock_debuging（）”。 拼写固定为2.1.2-alpha中的evthread_enable_lock_debugging（）； 目前都支持这两个名称。

#### 6. 调试事件的使用

libevent 可以检测使用事件时的一些常见错误并且进行报告。这些错误包括： 

- 将未初始化的 event 结构体当作已经初始化的 

- 试图重新初始化未决的 event 结构体 

跟踪哪些事件已经初始化需要使用额外的内存和处理器时间，所以只应该在真正调试程序的时候才启用调试模式。

**接口**

```c
void event_enable_debug_mode(void);
```

必须在创建任何 event_base 之前调用这个函数。 

如果在调试模式下使用大量由 event_assign（而不是 event_new）创建的事件，程序可能会耗尽内存，这是因为没有方式可以告知 libevent 由 event_assign 创建的事件不会再被使用了（可以调用 event_free 告知由 event_new 创建的事件已经无效了）。如果想在调试时避免耗尽内存，可以显式告知 libevent 这些事件不再被当作已分配的了：

**接口**

```c
void event_debug_unassign(struct event *ev);
```

没有启用调试的时候调用 event_debug_unassign 没有效果。

**示例**

```c
#include <event2/event.h>
#include <event2/event_struct.h>

#include <stdlib.h>

void cb(evutil_socket_t fd, short what, void *ptr)
{
    /* We pass 'NULL' as the callback pointer for the heap allocated
     * event, and we pass the event itself as the callback pointer
     * for the stack-allocated event. */
    struct event *ev = ptr;

    if (ev)
        event_debug_unassign(ev);
}

/* Here's a simple mainloop that waits until fd1 and fd2 are both
 * ready to read. */
void mainloop(evutil_socket_t fd1, evutil_socket_t fd2, int debug_mode)
{
    struct event_base *base;
    struct event event_on_stack, *event_on_heap;

    if (debug_mode)
       event_enable_debug_mode();

    base = event_base_new();

    event_on_heap = event_new(base, fd1, EV_READ, cb, NULL);
    event_assign(&event_on_stack, base, fd2, EV_READ, cb, &event_on_stack);

    event_add(event_on_heap, NULL);
    event_add(&event_on_stack, NULL);

    event_base_dispatch(base);

    event_free(event_on_heap);
    event_base_free(base);
}
```

详细的事件调试是一项只能在编译时使用CFLAGS环境变量“ -DUSE_DEBUG”启用的功能。 启用此标志后，针对Libevent编译的任何程序都将输出非常详细的日志，详细说明后端的低级活动。 这些日志包括但不限于以下内容：

- 活动添加

- 事件删除

- 平台特定的事件通知信息

无法通过API调用启用或禁用此功能，因此只能在开发人员内部使用。

这些调试功能已添加到Libevent 2.0.4-alpha中。

#### 7.检测libevent的版本

新版本的 libevent 会添加特征，移除 bug。有时候需要检测 libevent 的版本，以便： 

- 检测已安装的 libevent 版本是否可用于创建你的程序 

- 为调试显示 libevent 的版本 

- 检测 libevent 的版本，以便向用户警告 bug，或者提示要做的工作

**接口**

```c
#define LIBEVENT_VERSION_NUMBER 0x02000300
#define LIBEVENT_VERSION "2.0.3-alpha"
const char *event_get_version(void);
ev_uint32_t event_get_version_number(void);
```

宏返回编译时的 libevent 版本；函数返回运行时的 libevent 版本。注意：如果动态链接到libevent，这两个版本可能不同。 

可以获取两种格式的 libevent 版本：用于显示给用户的字符串版本，或者用于数值比较的4字节整数版本。整数格式使用高字节表示主版本，低字节表示副版本，第三字节表示修正版本，最低字节表示发布状态：0表示发布，非零表示某特定发布版本的后续开发序列。 

所以，libevent 2.0.1-alpha 发布版本的版本号是[02 00 01 00]，或者说0x02000100。2.0.1-alpha 和2.0.2-alpha 之间的开发版本可能是[02 00 01 08]，或者说0x02000108。

**示例：编译时检测**

```c
#include <event2/event.h>

#if !defined(LIBEVENT_VERSION_NUMBER) || LIBEVENT_VERSION_NUMBER < 0x02000100
#error "This version of Libevent is not supported; Get 2.0.1-alpha or later."
#endif

int
make_sandwich(void)
{
        /* Let's suppose that Libevent 6.0.5 introduces a make-me-a
           sandwich function. */
#if LIBEVENT_VERSION_NUMBER >= 0x06000500
        evutil_make_me_a_sandwich();
        return 0;
#else
        return -1;
#endif
}
```

**示例：运行时检测**

```c
#include <event2/event.h>
#include <string.h>

int
check_for_old_version(void)
{
    const char *v = event_get_version();
    /* This is a dumb way to do it, but it is the only thing that works
       before Libevent 2.0. */
    if (!strncmp(v, "0.", 2) ||
        !strncmp(v, "1.1", 3) ||
        !strncmp(v, "1.2", 3) ||
        !strncmp(v, "1.3", 3)) {

        printf("Your version of Libevent is very old.  If you run into bugs,"
               " consider upgrading.\n");
        return -1;
    } else {
        printf("Running with Libevent version %s\n", v);
        return 0;
    }
}

int
check_version_match(void)
{
    ev_uint32_t v_compile, v_run;
    v_compile = LIBEVENT_VERSION_NUMBER;
    v_run = event_get_version_number();
    if ((v_compile & 0xffff0000) != (v_run & 0xffff0000)) {
        printf("Running with a Libevent version (%s) very different from the "
               "one we were built with (%s).\n", event_get_version(),
               LIBEVENT_VERSION);
        return -1;
    }
    return 0;
}
```

本节描述的宏和函数定义在<event2/event.h>中。event_get_version 函数首次出现在1.0c版本；其他的首次出现在2.0.1-alpha 版本。

#### 8. 释放全局的Libevent结构
即使释放了使用Libevent分配的所有对象，也将剩下一些全局分配的结构。通常这不是问题：退出流程后，无论如何都将对其进行清理。但是拥有这些结构可能会使某些调试工具误以为Libevent正在泄漏资源。如果需要确保Libevent已发布所有内部库全局数据结构，则可以调用：

**接口**

```c
void libevent_global_shutdown（void）;
```

此函数不会释放Libevent函数返回给您的任何结构。如果要在退出之前释放所有内容，则需要自己释放所有事件，event_bases，bufferevents等。

调用libevent_global_shutdown（）将使其他Libevent函数的行为无法预期。除了作为您的程序调用的最后一个Libevent函数外，不要调用它。一个例外是libevent_global_shutdown（）是幂等(idempotent)的：可以调用它，即使它已经被调用也可以。

此函数在<event2 / event.h>中声明。它是在Libevent 2.1.1-alpha中引入的。

### 二、创建event_base

使用 libevent 函数之前需要分配一个或者多个 event_base 结构体。每个 event_base 结构体持有一个事件集合，可以检测以确定哪个事件是激活的。 

如果设置 event_base 使用锁，则可以安全地在多个线程中访问它。然而，其事件循环只能运行在一个线程中。如果需要用多个线程检测 IO，则需要为每个线程使用一个 event_base。 

每个 event_base 都有一种用于检测哪种事件已经就绪的“方法”，或者说后端。可以识别的方法有： 

- select 

- poll 

- epoll 

- kqueue 

- devpoll 

- evport 

- win32 

用户可以用环境变量禁止某些特定的后端。比如说，要禁止 kqueue 后端，可以设置 EVENT_NOKQUEUE 环境变量 。 如果要用编程的方法禁止后端 ， 请看下面关于event_config_avoid_method（）的说明。

#### 1. 建立默认的 event_base
event_base_new（）函数分配并且返回一个新的具有默认设置的 event_base。函数会检测环境变量，返回一个到 event_base 的指针。如果发生错误，则返回 NULL。选择各种方法时，函数会选择 OS 支持的最快方法。

**接口**

```c
struct event_base *event_base_new(void);
```

大多数程序使用这个函数就够了。 

event_base_new（）函数声明在<event2/event.h>中，首次出现在 libevent 1.4.3版。

#### 2. 创建复杂的 event_base
要对取得什么类型的 event_base 有更多的控制，就需要使用 event_config。event_config是一个容纳event_base 配置信息的不透明结构体。需要 event_base 时，将 event_config传递给 event_base_new_with_config（）。

**接口**

```c
struct event_config *event_config_new(void);
struct event_base *event_base_new_with_config(const struct event_config *cfg);
void event_config_free(struct event_config *cfg);
```

要使用这些函数分配 event_base，先调用 event_config_new（）分配一个 event_config。 然后，对event_config 调用其它函数，设置所需要的 event_base 特征。最后，调用 event_base_new_with_config（）获取新的 event_base。完成工作后，使用 event_config_free（）释放 event_config。

**接口**

```c
int event_config_avoid_method(struct event_config *cfg, const char *method);

enum event_method_feature {
    EV_FEATURE_ET = 0x01,
    EV_FEATURE_O1 = 0x02,
    EV_FEATURE_FDS = 0x04,
};
int event_config_require_features(struct event_config *cfg,
                                  enum event_method_feature feature);

enum event_base_config_flag {
    EVENT_BASE_FLAG_NOLOCK = 0x01,
    EVENT_BASE_FLAG_IGNORE_ENV = 0x02,
    EVENT_BASE_FLAG_STARTUP_IOCP = 0x04,
    EVENT_BASE_FLAG_NO_CACHE_TIME = 0x08,
    EVENT_BASE_FLAG_EPOLL_USE_CHANGELIST = 0x10,
    EVENT_BASE_FLAG_PRECISE_TIMER = 0x20
};
int event_config_set_flag(struct event_config *cfg,
```

调用 event_config_avoid_method（）可以通过名字让 libevent 避免使用特定的可用后端。 

调用 event_config_require_feature（）让 libevent 不要使用不能提供所有功能的后端。 

调用 event_config_set_flag（）让 libevent 在创建 event_base 时设置一个或者多个将在下面介绍的运行时标志。

**event_config_require_features（）可识别的特征值有：**

- EV_FEATURE_ET：要求支持边沿触发的后端 

- EV_FEATURE_O1：要求添加、删除单个事件，或者确定哪个事件激活的操作是 O（1）复杂度的后端 
-  EV_FEATURE_FDS：要求支持任意文件描述符，而不仅仅是套接字的后端

**event_config_set_flag（）可识别的选项值有：**

- **EVENT_BASE_FLAG_NOLOCK**：不要为 event_base 分配锁。设置这个选项可以为event_base 节省一点用于锁定和解锁的时间，但是让在多个线程中访问 event_base成为不安全的。 

- **EVENT_BASE_FLAG_IGNORE_ENV**：选择使用的后端时，不要检测 EVENT_*环境变量。使用这个标志需要三思：这会让用户更难调试你的程序与 libevent 的交互。 

- **EVENT_BASE_FLAG_STARTUP_IOCP**：仅用于 Windows，让 libevent 在启动时就启用任何必需的 IOCP 分发逻辑，而不是按需启用。 

- **EVENT_BASE_FLAG_NO_CACHE_TIME**：不是在事件循环每次准备执行超时回调时检测当前时间，而是在每次超时回调后进行检测。注意：这会消耗更多的 CPU 时间。 

- **EVENT_BASE_FLAG_EPOLL_USE_CHANGELIST**：告诉 libevent，如果决定使用epoll 后端，可以安全地使用更快的基于 changelist 的后端。epoll-changelist 后端可以在后端的分发函数调用之间，同样的 fd 多次修改其状态的情况下，避免不必要的系统调用。但是如果传递任何使用 dup（）或者其变体克隆的 fd 给 libevent，epoll-changelist后端会触发一个内核 bug，导致不正确的结果。在不使用 epoll 后端的情况下，这个标志是没有效果的。也可以通过设置 EVENT_EPOLL_USE_CHANGELIST 环境变量来打开 epoll-changelist 选项。 
- **EVENT_BASE_FLAG_PRECISE_TIMER**: 默认情况下，Libevent尝试使用操作系统提供的最快的可用计时机制。 如果存在较慢的计时机制，可以提供更精细的计时精度，则此标志告诉Libevent改用该计时机制。 如果操作系统不提供这种慢速但精确的机制，则此标志无效。

上述操作 event_config 的函数都在成功时返回0，失败时返回-1。 

>  **注意**  设置 event_config，请求 OS 不能提供的后端是很容易的。比如说，对于 libevent 2.0.1-alpha，在 Windows 中是没有 O（1）后端的；在 Linux 中也没有同时提供 EV_FEATURE_FDS 和EV_FEATURE_O1 特 征 的 后 端 。 如 果 创 建 了 libevent 不 能 满 足 的 配 置 ，event_base_new_with_config（）会返回 NULL

**接口**

```c
int event_config_set_num_cpus_hint(struct event_config *cfg, int cpus)
```

这个函数当前仅在 Windows 上使用 IOCP 时有用，虽然将来可能在其他平台上有用。这个函数告诉 event_config 在生成多线程 event_base 的时候，应该试图使用给定数目的 CPU。注意这仅仅是一个提示：event_base 使用的CPU 可能比你选择的要少。  

**接口**

```c
int event_config_set_max_dispatch_interval(struct event_config *cfg,
    const struct timeval *max_interval, int max_callbacks,
    int min_priority);
```

此功能通过限制在检查更多高优先级事件之前可以调用多少个低优先级事件回调来防止优先级倒置。 如果max_interval为非null，则事件循环将检查每个回调之后的时间，如果max_interval已通过，则重新扫描高优先级事件。 如果max_callbacks为非负数，则在调用max_callbacks回调后，事件循环还会检查更多事件。 这些规则适用于min_priority或更高级别的任何事件。

**示例：优先使用边缘触发的后端**

```c
struct event_config *cfg;
struct event_base *base;
int i;

/* My program wants to use edge-triggered events if at all possible.  So
   I'll try to get a base twice: Once insisting on edge-triggered IO, and
   once not. */
for (i=0; i<2; ++i) {
    cfg = event_config_new();

    /* I don't like select. */
    event_config_avoid_method(cfg, "select");

    if (i == 0)
        event_config_require_features(cfg, EV_FEATURE_ET);

    base = event_base_new_with_config(cfg);
    event_config_free(cfg);
    if (base)
        break;

    /* If we get here, event_base_new_with_config() returned NULL.  If
       this is the first time around the loop, we'll try again without
       setting EV_FEATURE_ET.  If this is the second time around the
       loop, we'll give up. */
}
```

**示例：避免优先级倒置**

```c
struct event_config *cfg;
struct event_base *base;

cfg = event_config_new();
if (!cfg)
   /* Handle error */;

/* I'm going to have events running at two priorities.  I expect that
   some of my priority-1 events are going to have pretty slow callbacks,
   so I don't want more than 100 msec to elapse (or 5 callbacks) before
   checking for priority-0 events. */
struct timeval msec_100 = { 0, 100*1000 };
event_config_set_max_dispatch_interval(cfg, &msec_100, 5, 1);

base = event_base_new_with_config(cfg);
if (!base)
   /* Handle error */;

event_base_priority_init(base, 2);
```

这些函数和类型在<event2 / event.h>中声明。

EVENT_BASE_FLAG_IGNORE_ENV标志首先出现在Libevent 2.0.2-alpha中。 EVENT_BASE_FLAG_PRECISE_TIMER标志首先出现在Libevent 2.1.2-alpha中。 event_config_set_num_cpus_hint（）函数是Libevent 2.0.7-rc中的新增功能，而event_config_set_max_dispatch_interval（）是2.1.1-alpha中的新增功能。 本节中的所有其他内容首先出现在Libevent 2.0.1-alpha中。

#### 3. 检查 event_base 的后端方法

有时候需要检查 event_base 支持哪些特征，或者当前使用哪种方法。

**接口**

```c
const char **event_get_supported_methods(void);
```

event_get_supported_methods（）函数返回一个指针，指向 libevent 支持的方法名字数组。这个数组的最后一个元素是 NULL。

**示例**

```c
int i;
const char **methods = event_get_supported_methods();
printf("Starting Libevent %s.  Available methods are:\n",
    event_get_version());
for (i=0; methods[i] != NULL; ++i) {
    printf("    %s\n", methods[i]);
}
```

> **注意**这个函数返回 libevent 被编译以支持的方法列表。然而 libevent 运行的时候，操作系统可能 不能支持所有方法。比如说，可能 OS X 版本中的 kqueue 的 bug 太多，无法使用。

**接口**

```c
const char *event_base_get_method(const struct event_base *base);
enum event_method_feature event_base_get_features(const struct event_base *base);
```

event_base_get_method（）返回 event_base 正在使用的方法。

event_base_get_features（）返回 event_base 支持的特征的比特掩码。

**示例**

```c
struct event_base *base;
enum event_method_feature f;

base = event_base_new();
if (!base) {
    puts("Couldn't get an event_base!");
} else {
    printf("Using Libevent with backend method %s.",
        event_base_get_method(base));
    f = event_base_get_features(base);
    if ((f & EV_FEATURE_ET))
        printf("  Edge-triggered events are supported.");
    if ((f & EV_FEATURE_O1))
        printf("  O(1) event notification is supported.");
    if ((f & EV_FEATURE_FDS))
        printf("  All FD types are supported.");
    puts("");
}
```

这个函数定义在<event2/event.h>中。event_base_get_method（）首次出现在1.4.3版本中，其他函数首次出现在2.0.1-alpha 版本中。

#### 4. 释放 event_base
使用完 event_base 之后，使用 event_base_free（）进行释放。
**接口**

```c
void event_base_free(struct event_base *base);
```

> 注意：这个函数不会释放当前与 event_base 关联的任何事件，或者关闭他们的套接字，或者释放任何指针。

event_base_free（）定义在<event2/event.h>中，首次由 libevent 1.2实现。

#### 5. 设置 event_base 的优先级
libevent 支持为事件设置多个优先级。然而，event_base 默认只支持单个优先级。可以调用event_base_priority_init（）设置 event_base 的优先级数目。

**接口**

```c
int event_base_priority_init(struct event_base *base, int n_priorities);
```

成功时这个函数返回0，失败时返回-1。base 是要修改的 event_base，n_priorities 是要支持的优先级数目，这个数目至少是1。每个新的事件可用的优先级将从0（最高）到n_priorities-1（最低）。
常量 EVENT_MAX_PRIORITIES 表示 n_priorities 的上限。调用这个函数时为 n_priorities给出更大的值是错误的。

> **注意**
> 必须在任何事件激活之前调用这个函数，最好在创建 event_base 后立刻调用。

要查找某个数据库当前支持的优先级数量，可以调用event_base_getnpriorities（）。

**接口**

```c
int event_base_get_npriorities(struct event_base *base);
```

返回值等于基础中配置的优先级数。 因此，如果event_base_get_npriorities（）返回3，则允许的优先级值为0、1和2。
  

**示例**
关于示例，请看 event_priority_set 的文档。
默认情况下，与 event_base 相关联的事件将被初始化为具有优先级 n_priorities / 2。event_base_priority_init（）函数定义在<event2/event.h>中，从 libevent 1.0版就可用了。

#### 6. 在 fork（）之后重新初始化 event_base
不是所有事件后端都在调用 fork（）之后可以正确工作。所以，如果在使用 fork（）或者其
他相关系统调用启动新进程之后，希望在新进程中继续使用 event_base，就需要进行重新
初始化。
**接口**

```c
int event_reinit(struct event_base *base);
```

成功时这个函数返回0，失败时返回-1。 
 **示例**

```c
struct event_base *base = event_base_new();

/* ... add some events to the event_base ... */

if (fork()) {
    /* In parent */
    continue_running_parent(base); /*...*/
} else {
    /* In child */
    event_reinit(base);
    continue_running_child(base); /*...*/
}
```

event_reinit（）定义在<event2/event.h>中，在 libevent 1.4.3-alpha 版中首次可用。













































































## 示例

### Timer

### TCP

### HTTP

### HTTPS



