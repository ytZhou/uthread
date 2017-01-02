uthread
=======

一个简单的C++用户级线程（协程）库


* 一个调度器可以拥有多个协程
* 通过`uthread_create`创建一个协程
* 通过`uthread_resume`运行或者恢复运行一个协程
* 通过`uthread_yield`挂起一个协程，并切换到主进程中
* 通过`schedule_finished` 判断调度器中的协程是否全部运行完毕
* 每个协程最多拥有128Kb的栈，增大栈空间需要修改源码的宏`DEFAULT_STACK_SIZE `，并重新编译


更详细的介绍，请查看我的中文博客 [人既无名的专栏](http://blog.csdn.net/qq910894904/article/details/41911175). 

===============================================================
笔记：
create的时候，如果有空闲的协程，则复用，没有则创建新的，同时保存fun/arg指针参数，设置状态为runenable。
resume的时候，判断状态，如果是runenable则获取上下文放到协程的ctx当中，设置栈和栈大小，设置ulink，修改状态为running，通过makecontext设置ctx指向，并设置参数。注意runenable状态后无break，直接调用swapcontext切换到ctx上下文当中，同时保存当前上下文到mainctx当中。
切换到ctx上下文的时候，会调用create创建时指定的函数。在指定的函数，需要自己主动挂起，调用yield函数。
yield函数，主动挂起。设置状态为suspend，同时保存当前上下文到协程的ctx当中，切换到mainctx上下文当中（其实就是切到了上一次调用resume的地方，具体是resume函数当中swapcontext函数后面，后面什么也没做，因此直接返回，接着运行下面的代码）。

当再次调用resume的时候，判断协程状态为suspend，那么直接调用swapcontext，切换到上次调用yield的地方，同时保存当前上下文到mainctx当中，这样就恢复到上次执行的地方。

通过对代码的走读，其实就是上下文的保存，首次resume的时候，需要保存当前上下文main，同时切换到新的上下文中去执行指定的函数。当挂起的时候，恢复到之前的保存的上下main，同时保存当前上下文到ctx。
非首次resume，那么直接切换到ctx当中去，保存当前上下文到main当中即可。
