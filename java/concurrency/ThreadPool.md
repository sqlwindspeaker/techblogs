# ThreadPoolExecutor 深入理解

## 线程池的几大要素

这几大基本要素都在构造函数中体现出来了

#### corePoolSize 和 maximumPoolSize

这两个参数决定了线程池里可能的工作线程个数

#### workQueue

用于缓存任务的队列(回忆一下BlockingQueue和Queue的区别)。主要分为有界队列（基于ArrayList），无界队列（基于linkedList）和同步队列三类

#### keepAliveTime 和 TimeUnit

这两个参数来决定线程空闲多久可以被销毁

#### threadFactory

需要创建线程时用到的线程池，可以自己控制线程创建出来的名字类型，或者是否为守护线程，线程优先级等等

#### rejectHandler

当线程池无法接受提交的任务时的处理，默认是抛出一个异常，也可以选择直接丢弃当前任务，或者丢弃队列中最旧的任务，或者转由提交任务的线程自己执行该任务

## 线程池的设计思路

首先说，虽然线程池的基本思想是提前准备好一批线程闲呆着，等用的时候可以拿来直接用，避免了频繁创建线程的系统开销，但实际操作起来并不是这么简单。

假设你用前面的思路想去实现一个线程池，你稍加尝试就会发现Thread类的任务是通过构造函数传递的，并没有一个setTask(Runnable)这样的方法。因此，你并没有办法能够为一个已经Thread替换其中的Task。

既然这个需求如此之合理，为什么J.U.C的设计者没有实现这样一个方法呢？总不会是设计者的失误吧，毕竟如果这是个问题的话，这么多年来，早就应该像后来增加Callable接口那样为Thread类增加一个setTask方法了。

其实这个问题只要我们稍加尝试去按照这个思路去实现一下就会发现问题所在。我们不失关键的只考虑线程已经启动之后的情况，setTask执行的时候应该会要求线程当前是空闲的。那么怎么算空闲呢，如果把Thread中的线程任务执行完毕叫做空闲，那么线程的结束退出又应该是什么条件呢？所以我们可以看出，其实并没有一个很好的方法可以进行setTask操作，因此这条路是显然走不通的。

我们可以看出，问题的症结其实在于线程的退出和空闲状态的在Thread类上的矛盾。造成这样的问题的核心在于Thread类是一个通用的线程，它需要面向通用的情况就是，独立的一个线程执行完给定的任务即退出线程。而对于我们线程池的需求来说，我们需要的是一个永不退出的线程，因此如果我们提供一个永不退出的线程实现，就可以把当前用户的任务执行完毕当作线程的空闲状态来看待。实际的实现中ThreadPoolService也是这样做的，它的每一个工作线程会执行一个叫Worker的任务（Runnable对象），这个任务在线程池没有回收该线程时是永远不会退出的。这个Worker的核心工作就是接收用户实际提交的任务（Runnable对象），并同步调用该任务的run方法来执行用户任务。

接下来的问题是线程在空闲的时候要做什么？最简单的方案是让他死循环空转，这肯定不是一个好方案，因为会极大的浪费CPU资源。稍微好一点的方案是让这些空闲线程们定时检查一下是否有任务添加进去，如果有就执行，没有继续睡过去。这个方案虽然避免了空转，但是由于时间片的存在，导致吞吐量很难上去，并且任务执行的延迟比较大。最合适的方案其实是让这些线程都等待在一个Condition上，当一个任务提交时，激活其中一个线程，执行该任务。当任务执行完毕后，继续等待在这个Condition上。我们可以看出这是一个经典的生产者消费者问题，因此我们可以通过一个阻塞队列来实现这个模型。

到现在为止，我们的方案和实际的ThreadPoolExecutor的实现已经很相似了，让我看看实际的线程池的实现方案是怎样的。

## 线程池的工作流程

J.U.C中的ThreadPoolExecutor的基本思路就是在提交任务时，将任务提交到任务队列中（workQueue），然后所有的工作线程作为消费者来消费这个workQueue，从而执行任务。但是考虑到具体的情况，ThreadPoolExecutor还有一些其他的设计优化点。

### 核心线程池大小和最大线程池大小

首先，ThreadPoolExecutor提出了一个核心线程池大小（corePoolSize），这个参数可以是0，但是如果它大于0的话，则线程池中的工作线程数在不超过这个size的时候，**默认情况下**是不会被销毁的（可以通过方法设置）。这个设计的目的是为了保证线程池中永远有一定数量的工作线程，从而提高任务的执行响应速度。

除了核心线程池大小之外，还有一个最大线程池大小（maximumPoolSize）参数。当它大于0的时候，线程池中的实际工作线程数是永远不会超过这数字的。这个参数不能小于核心线程池的大小，而这两个参数的差，则是默认情况下有多少个线程在空闲一段时间后（keepAliveTime），可以被线程池销毁从而节约系统资源。

这里需要注意的是，这两个参数的命名可能会有一些误导。实际上ThreadPoolExecutor中并没有一个所谓“核心线程池”的东西，对于线程池中创建的任何一个线程，它们都是平等的。线程池只是保证工作线程数满足前面的约束条件，而不会关心保留哪些线程而销毁哪些线程。本文后续为了描述简便，将不再特意强调这一点概念。

### 任务的提交过程

#### 线程池的预热

把核心线程池填满的过程叫做线程池的预热，这是为了保证后续任务提交的时候的高效执行。在默认情况下，预热的过程是通过前N个任务的提交来实现的（假设corePoolSize＝N）。换句话说，如果N>0，最开始提交的N个任务，会直接创建一个新的线程，并直接执行该任务。

如果需要的话，也可以通过*prestartAllCoreThreads*方法手动进行线程池的预热，而不是等待任务提交。

#### 任务提交到队列

线程池预热完毕后，提交到线程池的任务就会被直接添加到任务队列中，由后端的工作线程进行消费。在这个阶段中，如果所有的工作线程都在忙，则新加入的线程会一直等待在工作队列中，而不会新建工作线程。

#### 队列已满的情况，尝试增加非核心线程

如果使用的队列是有界队列，则当任务的提交速度比消费速度高时，则会面临队列被填满的情况。这种情况下，线程池会尝试在工作线程没有超过最大线程数的情况下，直接新建一个线程来直接执行该任务（再次强调，实际上只是增加了一个线程，并不意味着一定是这个线程后续被销毁）。该线程执行任务完毕后，也同时成为任务队列的消费者继续消费队列中的积压任务。从另外一个角度说，尽管我们使用了任务队列，但是并不能保证提交的任务是按照提交顺序被执行的。

#### 增加非核心线程失败

如果线程数达到最大限度，增加非核心线程失败的话，线程池则会拒绝该任务的提交。拒绝的具体方式取决于创建线程池时提供的RejectPolocy
































