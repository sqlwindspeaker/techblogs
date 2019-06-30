# Runnable, Callable 和 Future

Callable 其实是 Runnable的一个扩展

最开始的线程处理的都是一个没有返回值的逻辑，因此在最初的时候只有Runnable接口，它规定了一个 *run* 方法，来实现具体的业务逻辑，它**没有返回值**，并且**不抛出异常**，通过在Thread中装载后执行。

后来考虑到有返回值的逻辑，尤其是类似ForkJoin框架的任务，引入了Callable接口，它规定了 *call* 方法，具**有一个返回值**，并且**允许抛出异常**。

因为任务是异步执行的，因此需要有一种机制能够获取任务的运行结果。一个最直接的思路就是提供一个该任务的Handle，因此，J.U.C提供了Future类，Future类的最醒目的目标是提供了get方法，当任务被异步的执行后，调用者可以通过该任务对应的Future对象的get方法，获取该任务的结果，并自己处理该任务执行过程中的异常，从而实现了比Runnable更灵活的任务处理。

前面我们说Future类其实是任务的一个Handle，而不是一个单纯的返回结果的包装。Future还解决了之前Runnable时代的另外一个缺陷。在Runnable时代，对一个任务的控制，是需要显式的通过线程来完成的。简单来说，一个任务被提交到线程中执行之后，提交者没有较好的办法去对这个任务**直接**进行管理操作，比如cancel，而必须要通过获得该任务所在的线程才可以。这样一来，对于在线程池中执行的任务，由于无法得到该任务对应的线程对象，因此无法对任务进行有效的控制。现在有了Future作为一个任务的Handle，已经提交的任务可以通过Future进行管理操作，因此Future类还提供了*cancel*, *isCancelled*, *isDone* 三个方法来检查线程的状态。

我们知道，要启动一个新线程执行任务，对于Runnable任务来说，可以创建一个Thread来执行该任务。而对于Callable任务，Thread类没有提供处理Callable的构造器，因此，为了能够直接在线程中启动一个Callable，我们需要对Callable进行一层包装，让他成为一个Runnable，再通过Thread类进行异步执行。J.U.C中实现该任务的组件叫做FutureTask，这个类被设计为首先是一个Runnable，因此它可以被Thread执行，其次它还是一个Future，也就是说它自己就是自己的Handle（这个设计主要是因为历史上Thread类没有提供获取Future的方法）。FutureTask中的 *run* 方法会被路由到该Callable的 *call* 方法。

刚才我们说，Future还有一个好处是可以检查任务执行的情况以及处理异常。因此如果对于原始的Runnable任务也想使用类似的服务的话，同样可以通过FutureTask对一个Runnable任务进行包装来实现。对于Runnable任务，FutureTask首先在内部将其封装为一个RunnableAdapter对象，该对象是一个Callable的实现，进而将其作为一个普通的Callable任务进行处理。

了解了这些组件之后，对于线程池来处理Callable任务就很简单了。线程池对Callable的处理和Thread类一样，通过FutureTask将一个Callable任务包装成一个Runnable任务，通过线程池的 *execute* 方法提交运行，同时将该FutureTask对象本身作为该Callable的Future对象返回给任务的提交者。

总结一下，Callable任务通过提供Future对象作为任务的Handle，扩展了原先Runnable任务的不足，主要包括：

1. **获取任务的返回值**

2. **更灵活的处理任务异常**

3. **更和线程机制解耦的任务状态管理**

这三个功能







