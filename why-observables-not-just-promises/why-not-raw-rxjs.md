# Why not raw RxJS?

RxJS can power any sync or async experience, and has a tremendous installation base, with more monthly installs than React and Angular combined. It is a great tool for building things - such as powering the Angular templating system. But, without a great interface to it—like Angular templates—its API and 'everything is a stream' philosophy is just not easy to learn or remember.

People who work with RxJS routinely surface the same kinds of questions:

* What operator do I use to do X?
* When do I subscribe?
* What do I do with my subscription objects?
* What is the different between mergeMap and exhaustMap? Is flatMap still a thing?
* How do i decompose an app into units that RxJS can work with

> #### 𝗥𝘅𝑓𝑥: _RxJS The Good Parts™️_



Sure you could do the RxJS thing of modeling the entire app as one stream with many operators in its pipe, and call `subscribe()` on a single object... But I think it's cleaner, more decoupled, and more like the Actor model, if listeners can be in separate files, and only indirectly connected via pub-sub messages. Imagine growing an application strictly by adding new files, hardly ever re-opening files, since each new piece of logic can be a new agent processing the event stream. It's really quite an elegant architecture!\
\
In 𝗥𝘅𝑓𝑥, the most commonly used RxJS concepts are available in a simpler API that looks like returning Promises or Observables from Event Listeners. Each listener can be put into a different concurrency mode — via .listenBlocking for example.\
\
This all allows you to bring the power of RxJS into a particular corner of your application, or structure your whole application as services/listeners. Either way, you don't need to model any part of your application as a single stream, saving the refactoring time.

Now, let's see how building apps with 𝗥𝘅𝑓𝑥 compares to raw RxJS with a few examples:
