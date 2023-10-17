# Introduction

## Storytelling: "The Illusion of Multitasking" 

> A workshop that leads to a discussion about focus, focus span, disturbing factors, limit active work.

- [ITP] ("If Time Permits") introduce the results of the three scenarios one-by-one (you have to split up the picture)
- fortunately Java's concurrency features are more sophisticated!

## Sneak Preview

- [ITP] Section postponed for now (couldn't fit it in nicely), but introduce it if time permits and you still see the value in it.

This talk will be about: 

* [ ] trains
* [ ] ordering drinks
* [ ] ordering a meal
* [ ] brown M&M's

# The Journey So Far

> Style this part as station announcement signs

- [ITP] Design a graphic that looks like a train itinerary and that I can use to indicate progress.

### Domain Introduction, Part 1

- The concurrency train is a luxury one - it has a restaurant on board! (on-board bistro image)
- Diagram that quickly explains the restaurant domain

### Threads

* example code: serving meals
* pros: 
* cons: unit-of-work and mechanism to run it are one and the same.

### ExecutorService

* example code: serving meals
* pros: 
* cons: error propagation, communicating cancellation intent

### ThreadLocal

* example code: pass a value that is bound to a thread (waiterId) and that doesn't pollute any method signatures
* pros: 
* cons: unconstrained mutability, unbounded lifetime, expensive inheritance

> Few applications tend to use ThreadLocal variables. In a nutshell, Java ThreadLocal variables are created and stored as variables within the scope of a particular thread alone, and they cannot be accessed by other threads. If your application creates millions of virtual threads, and each virtual thread has its own ThreadLocal variable, then it can quickly consume java heap memory space. Thus, you want to be cautious of the size of data that is stored as ThreadLocal variables.
>
> You might wonder why ThreadLocal variables are not problematic in platform threads. The difference is that in platform threads, we don’t create millions of threads, whereas, in virtual threads, we do. Millions of threads, each with its own copy of ThreadLocal variables, can quickly fill up memory. They say small drops of water make an ocean. It’s very true here.
>
> In general, Java ThreadLocal variables are tricky to manage & maintain. It can also cause nasty production problems. Thus, limiting the scope of use of ThreadLocal variables can benefit your application, especially when using virtual threads.

Bottom line: ThreadLocals are memory-intensive & always mutable

- [ITP]: demo the behavior of the ThreadLocal that holds the `announcementId`.

### Honourable Mentions

* ForkJoinPool
* CompletableFuture
* ReentrantLock
* AtomicReference<T>
* Semaphore
* CountDownLatch

[ITP] Convert the slides in this section to the title-slide-with-background-and-opaque-description style.

## Why Does This Interest Me?

* Introduction of Hanno & Info Support.
* Include the following speaker notes:

  So, my name is Hanno. 
  From the Netherlands, and I work at Info Support as an IT consultant.
  I'm a Java Champion and an Oracle ACE.
  I am @hannotify on Twitter, Mastodon or Bluesky.
  (It'll always be called Twitter to me)
  (About the handle: get notified of everything Hanno does. I thought it was rather clever - my wife disagrees with me though, she thinks I'm an major geek!)
  I post about things I like, as everyone does I suppose.
  Java Development, Version Control, Sustainability and making music.
  If you're into that stuff, by all means give me a follow on your favourite social network!

* Reintroduce is-duke, award icons and social handle there
* Training "Concurrency in Java"

# Current Station: Virtual Threads

- Code: ExecutorService with virtual threads
- 
- Pros: millions of threads can run, allows thread-per-request style instead of thread-sharing style, can significantly improve application throughput when the number of concurrent tasks is high (> a few thousand) and the workload is not CPU-bound
- Cons: pinned threads with synchronization blocks, error propagation, communicating cancellation intent, thread-local variables perform badly with many (virtual) threads

* LockSupport, Semaphores enable _parking_ virtual threads.
* Now that virtual threads are available, what problems are getting worse?
  
> This is an important part of the talk, because these are the reasons why Java's support for concurrency is being extended.

> Few applications tend to use ThreadLocal variables. In a nutshell, Java ThreadLocal variables are created and stored as variables within the scope of a particular thread alone, and they cannot be accessed by other threads. If your application creates millions of virtual threads, and each virtual thread has its own ThreadLocal variable, then it can quickly consume java heap memory space. Thus, you want to be cautious of the size of data that is stored as ThreadLocal variables.
>
> You might wonder why ThreadLocal variables are not problematic in platform threads. The difference is that in platform threads, we don’t create millions of threads, whereas, in virtual threads, we do. Millions of threads, each with its own copy of ThreadLocal variables, can quickly fill up memory. They say small drops of water make an ocean. It’s very true here.

* Is this the final station?

# Next Station: Structured Concurrency

> JEP 437

## Storytelling: "Sitting down for a three-course dinner, but some ingredients are out of stock"

## Definition

> an approach to concurrent programming that preserves the natural relationship between tasks and subtasks, which leads to more readable, maintainable, and reliable concurrent code

The term "structured concurrency" was coined by Martin Sústrik and popularized by Nathaniel J. Smith.

Structured concurrency derives from the simple principle that

> If a task splits into concurrent subtasks then they all return to the same place, namely the task's code block.

> In summary, virtual threads deliver an abundance of threads. Structured concurrency can correctly and robustly coordinate them, and enables observability tools to display threads as they are understood by the developer. Having an API for structured concurrency in the JDK will make it easier to build maintainable, reliable, and observable server applications.

## What Problem Are We Trying To Solve? 

### Sorting M&M's

> Laat vier mensen parallel M&M's sorteren, maar stop er ook 1 Skittle in. Degene die de Skittle heeft, moet zich komen melden met de mededeling dat die subtaak niet voltooid kan worden. De andere drie maken de taak onafhankelijk van elkaar af.

- M & M's (WARNING: ABSOLUTELY NO BROWN ONES)
  - <https://www.insider.com/van-halen-brown-m-ms-contract-2016-9>
  - <https://www.thesmokinggun.com/sites/default/files/imagecache/750x970/documents/1982vanhalen9_0.gif>

## Task-Subtask Relation

TODO: toon een diagram van de ober-taken en laat zien hoe daar een 'task-subtask'-relatie tussen bestaat. 
TODO: toon de single-threaded-versie, omdat de syntactic structure van die code 1-op-1 mapt op dit diagram.

- because task structure should reflect code structure, like single-threaded code

## ShutdownOnFailure

- 'invoke all'
- short-circuiting
- cancellation?
- TODO

### Sorting M&M's, Revisited

> Laat opnieuw vier mensen parallel M&M's sorteren. Degene met de Skittle moet nu ook anderen informeren dat hun taak gestaakt moet worden.
> Grappige twist: iemand zou de Skittle ongemerkt kunnen opeten en alsnog veinzen dat het goed is gegaan. Dat kan ook een leuke live demo zijn.

## Demo, Part 1

- Maak 1 van de restaurant-taken kunstmatig een stuk langer, en laat zien hoe unstructured-versies van de code daar onnodig lang op gaan wachten. Structured-versies zullen de langlopende taak annuleren als een andere faalt.
- Revisit ExecutorService serving meals code
  - it’s missing thread coordination, observability and isolation
- Re-implement with StructuredTaskScope, making the task-subtask relation very clear
- Breid het concept 'kans op falen' uit naar specifiek voor starter, main & dessert. Daarmee kan ik snel demonstreren dat een langlopende main-taak word geannuleerd als 'dessert' faalt.
 
## Storytelling: "Ordering a drink, but no menu available"

## Domain Introduction, Part 2

- The concurrency train is a luxury one - it also has a bar on board! (on-board bistro image)
- Diagram that quickly explains the bar domain

## ShutdownOnSuccess

- 'invoke any'
- again, short-circuiting
- TODO

## Demo, Part 2

TODO

## Custom Shutdown Policies

## Sanity Check: Configuring Threads, With Structured Concurrency

- TODO: Hoe kun je beïnvloeden welke threadconfiguratie wordt gebruikt bij Structured Concurrency?

## Sanity Check: Why Didn't They 'Just' Enhance the ExecutorService Interface?

> StructuredTaskScope enforces structure and order upon concurrent operations. Thus it does not implement the ExecutorService or Executor interfaces since instances of those interfaces are commonly used in a non-structured way (see below). However, it is straightforward to migrate code that uses ExecutorService, but would benefit from structure, to use StructuredTaskScope.

note:

(listed as Alternative in the JEP:)
> Enhance the ExecutorService interface. We prototyped an implementation of this interface that always enforces structure and restricts which threads can submit tasks. However, we found it to be problematic because most uses of ExecutorService (and its parent interface Executor) in the JDK and in the ecosystem are not structured. Reusing the same API for a far more restricted concept is bound to cause confusion. For example, passing a structured ExecutorService instance to existing methods that accept this type would be all but certain to throw exceptions in most situations.

## Sanity Check: Imposing Structure On Concurrent Tasks, With ExecutorService

* Try using `cancel()` in the ExecutorService example, to make it clear how hard it is to get it spot-on.

> Like the JEP states: "We might attempt to do better by explicitly cancelling other subtasks when an error occurs, for example by wrapping tasks with try-finally and calling the cancel(boolean) methods of the futures of the other tasks in the catch block for the failing task. We would also need to use the ExecutorService inside a try-with-resources statement, as shown in the examples in JEP 425, because Future does not offer a way to wait for a task that has been cancelled. But all this can be very tricky to get right, and it often makes the logical intent of the code harder to discern. Keeping track of the inter-task relationships, and manually adding back the required inter-task cancellation edges, is asking a lot of developers."

* Try using `invokeAny()` and `invokeAll()`.
  * TODO: pros?
  * TODO: cons?
  * TODO: Hoe is `ShutdownOnFailure` beter of leesbaarder dan `ExecutorService.invokeAll()`?
    - lees: https://davidvlijmincx.com/posts/loom/invoke-all-with-virtual-threads/ 
    - lees: https://medium.com/@lavneesh.chandna/structured-concurrency-in-java-7a10b36ce0a3
  * TODO: Hoe is `ShutdownOnSuccess` beter of leesbaarder dan `ExecutorService.invokeAny()`?

* ForkJoinPool also imposes structure on concurrent tasks. However, that API is designed for compute-intensive tasks rather than tasks which involve I/O.

# Next Station: Scoped Values

> JEP 446

- Scoped values (JEP 446), TODO intro, live demo
- how do these features compare to their counterparts in Go, C# and Kotlin?
- rebinding scoped values
- inheriting scoped values
- preferable over ThreadLocals

## Useful for:

- hidden method arguments
- re-entrant code
- nested transactions
- graphics contexts

> In general, we advise migration to scoped values when the purpose of a thread-local variable aligns with the goal of a scoped value: one-way transmission of unchanging data.
>
> Thread-local variables retain their values until explicitly removed using the "remove" method or until the thread terminates. Developers often forget to remove them, causing data to persist longer than necessary. This can be a security risk, especially in thread pools where data may unintentionally transfer between tasks. For programs relying on mutable thread-local variables, finding a safe point to call "remove" can be challenging, potentially causing long-term memory leaks. It is preferable to limit data access to a specific execution period to prevent these issues.

## Demo ideas

* Keep a `waiterId` as a Scoped Value

## Sanity Check: How To Use In Conjunction With Structured Concurrency

* TODO (see JEP)

## Sanity Check: TODO

* TODO

# Practical Implications

- How will these features change the day-to-day life of a Java developer?
- TODO

# Have We Come To Journey’s End?

- What new features are expected to be developed after these?
- sharing streams of data among threads

> It is not a goal to define a means of sharing streams of data among threads (i.e., [channels](https://en.wikipedia.org/wiki/Channel_(programming) "‌")). We might propose to do so in the future.

- a new thread cancellation mechanism

> It is not a goal to replace the existing thread interruption mechanism with a new thread cancellation mechanism. We might propose to do so in the future.

- solving thread pinning when virtual threads are executing a synchronized code block.

> Virtual threads not releasing the underlying operating system thread when working on a synchronized method 
> is a limitation in JDK 19. It could be addressed in a future release of Java.


- what alternative features do languages like Go, C# and Kotlin offer, that might inspire Java language designers?
- TODO: even more?

# Wrap-Up And Q&A

TODO

# Meta-inhoud

## Grappen & Grollen

- TODO All aboard!
- TODO Choo choo!
- TODO The hype train
- TODO Grover die de blauwe meneer met snor moet bedienen in Sesamstraat
- TODO Treinachtergrond (of stationsachtergrond) voor elke titelslide.

## Referenties

- [https://inside.java/2020/08/07/loom-performance/](https://inside.java/2020/08/07/loom-performance/ "smartCard-inline")
- [https://dzone.com/articles/pitfalls-to-avoid-when-switching-to-virtual-thread](https://dzone.com/articles/pitfalls-to-avoid-when-switching-to-virtual-thread "smartCard-inline")
- Alle JEPs natuurlijk ook
- <https://github.com/hannotify/structured-concurrency-bar>

# CfP Submission

## Title

Java's Concurrency Journey Continues! Exploring Structured Concurrency and Scoped Values

## Abstract

Java's concurrency journey has been a long and winding one. We departed from the 'classic threads' station and traveled through Runnables, ExecutorServices, CompletableFutures and ForkJoinPools, before finally arriving at 'virtual threads'. But does 'finally' mean that we've arrived at our final destination, or is it a transfer at best?

Now that virtual threads are available, our Java programs will likely use an abundance of threads. This increase in thread count will immediately make thread coordination, observability and isolation more difficult. Two new Java features are currently in development that might make things a bit easier: Structured Concurrency and Scoped Values.

In this talk, we'll introduce and demonstrate these new features, and how they can help address the challenges that have emerged since the introduction of virtual threads. We'll also discuss how the availability of these features will impact your day-to-day programming life and whether Java's concurrency journey is actually over now that these features have become available or if there are still more stops to come.

## Elevator Pitch

The availability of virtual threads poses new challenges to the Java developer: how to ensure thread coordination, observability and isolation now that we're able to use millions of threads? Attend this talk to discover how the new features of Structured Concurrency and Scoped Values can improve the situation!

## Notes for the Program Committee

The talk outline will be roughly as follows:

* Short recap of the current state of concurrency in Java and how virtual threads have changed it recently
* What concurrency features is Java still missing? (will include references to other languages like Go, C# and Kotlin)
* What features are being developed at the moment?
  * Structured concurrency (JEP 437)
  * Scoped values (JEP 446)
* Live demo of these features
* How will these features change the day-to-day life of a Java developer?
* What new features are expected to be developed after these?
* Wrap-up and Q&A
