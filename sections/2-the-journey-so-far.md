<!-- .slide: data-background="img/background/boarding-a-train.jpg" data-background-color="black" data-background-opacity="0.4"-->

# The Journey So Far <!-- .element: class="stroke" -->

<https://www.pexels.com/photo/man-with-yellow-and-black-backpack-standing-near-train-1170181/> <!-- .element: class="attribution" -->

note:

**Time Elapsed:** `5:00`.

* Of course this talk will be about Structured Concurrency and Scoped Values, and we'll get to that.
* But to fully appreciate these additions and why they were introduced in the first place, we're in need of some serious background information.
* So what I want to do is alk you through the history of concurrency in Java and I'll try to make any positive or negative effects of the concurrency constructs that are available very explicit.
* So it'll be a fair comparison between the mainstream stuff and what just has been added to Java.
* Sounds good?

---

## Thread

* since Java 1.0; <!-- .element: class="fragment fade-in-then-semi-out" -->
* models a thread of execution in a program; <!-- .element: class="fragment fade-in-then-semi-out" -->
* allows doing multiple things at the same time. <!-- .element: class="fragment fade-in-then-semi-out" -->

note:

* An easy-to-relate example domain always helps me to understand technical concepts.

---

<!-- .slide: data-background="img/background/summer-dinner.jpg" data-background-color="black" data-background-opacity="0.6"-->

<https://www.pexels.com/photo/chairs-dining-room-food-furniture-460537/> <!-- .element: class="attribution" -->

note: 

**Storytelling**

Let's sit down for a three-course dinner.

---

<!-- .slide: data-background="img/background/grover.jpg" data-background-color="white" data-background-size="77%"-->

<a href="#" class="attribution" style="color: navy !important">(inspired by <em>Sesame Street</em>, hand-drawn by my wife Rianne)</a>

note:

* Our restaurant is not a very good restaurant, though!
* Things go *wrong* here.

Hand-drawn by my wife, btw! She's awesome! (opposites attract, I guess, my drawing has always been terrible, that's why I went into IT)

---

<!-- .slide: data-background="img/background/binary-code.jpg" data-background-color="black" data-background-opacity="0.3" -->

### Demo, Part 1

- Let's create a `SingleWaiterRestaurant`

<https://pxhere.com/en/photo/1458897> <!-- .element: class="attribution" -->

note:
*(tag `0-deep-dive-demo-start`)*

* SHOW: Restaurant interface
* CREATE: Main, SingleWaiterRestaurant
* RUN: Main
* SHOW: Waiter, Chef, Course, Ingredient

---

### A Restaurant With a Single Waiter

<pre><code class="java" data-trim data-line-numbers>
public interface Restaurant {
    MultiCourseMeal announceMenu();
}
</code></pre>

<pre class="fragment"><code class="java" data-trim data-line-numbers>
public class SingleWaiterRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws Exception {
        var elmo = new Waiter("Elmo");

        var starter = elmo.announceCourse(CourseType.STARTER);
        var main = elmo.announceCourse(CourseType.MAIN);
        var dessert = elmo.announceCourse(CourseType.DESSERT);

        return new MultiCourseMeal(starter, main, dessert);
    }
}
</code></pre>

note:
Poor Elmo! Instead of learning to fly (which I know he desperately wants to learn), he's stuck here being a waiter without any help from his coworkers.

---

<!-- .slide: data-background="img/background/binary-code.jpg" data-background-color="black" data-background-opacity="0.3" -->

### Demo, Part 2

- Let's create a `MultiWaiterThreadsRestaurant`

<https://pxhere.com/en/photo/1458897> <!-- .element: class="attribution" -->

note:
*(tag `1-deep-dive-created-single-waiter-restaurant`)*

* CREATE: MultiWaiterThreadsRestaurant, WaiterAnnounceCourseThread
* RUN: Main

---

### Modeling a Restaurant with Threads

<pre><code class="java" data-trim data-line-numbers>
public interface Restaurant {
    MultiCourseMeal announceMenu();
}
</code></pre>

<pre class="fragment"><code class="java" data-trim data-line-numbers>
public class MultiWaiterThreadsRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        var starterThread = new WaiterAnnounceCourseThread(grover, CourseType.STARTER);
        var mainThread = new WaiterAnnounceCourseThread(zoe, CourseType.MAIN);
        var dessertThread = new WaiterAnnounceCourseThread(rosita, CourseType.DESSERT);

        starterThread.start();  
        mainThread.start();
        dessertThread.start();

        starterThread.join();
        mainThread.join();
        dessertThread.join();

        return new MultiCourseMeal(starterThread.getAnnouncedCourse(),
                                   mainThread.getAnnouncedCourse(),
                                   dessertThread.getAnnouncedCourse());
    }
}
</code></pre>

note:

* Our restaurant serves a three-course meal that is announced by three waiters in separate Threads.

---

<pre><code class="java" data-trim data-line-numbers>
class WaiterAnnounceCourseThread extends Thread {
    private final Waiter waiter;
    private final CourseType courseType;
    private Course announcedCourse;

    public WaiterAnnounceCourseThread(Waiter waiter, CourseType courseType) {
        this.waiter = waiter;
        this.courseType = courseType;
    }

    @Override
    public void run() {
        announcedCourse = waiter.announceCourse(courseType);
    }

    public Course getAnnouncedCourse() {
        return announcedCourse;
    }
}
</code></pre>

---

### Pros ✅

* doing multiple things at once. <!-- .element: class="fragment fade-in-then-semi-out" -->
<br/>
<br/>
<br/>

### Cons ❌

* we can't return a value directly; <!-- .element: class="fragment fade-in-then-semi-out" -->
* the workload and mechanism to run it are one and the same. <!-- .element: class="fragment fade-in-then-semi-out" -->

note:

**Doing multiple things at once**

We can announce multiple courses at the same time.

**`Thread` can't return a value directly**

We'd need a `Callable` to be able to do that (from Java 1.5)

**workload and mechanism to run it are one and the same**

Workload (or: the task at hand, the unit-of-work)
One and the same, or to put it differently: they are tightly coupled

---

## ExecutorService

* since Java 1.5; <!-- .element: class="fragment fade-in-then-semi-out" -->
* executes tasks submitted to it; <!-- .element: class="fragment fade-in-then-semi-out" -->
* supports task queuing. <!-- .element: class="fragment fade-in-then-semi-out" -->

---

<!-- .slide: data-background="img/background/binary-code.jpg" data-background-color="black" data-background-opacity="0.3" -->

### Demo, Part 3

- Let's create a `MultiWaiterExecutorServiceRestaurant`

<https://pxhere.com/en/photo/1458897> <!-- .element: class="attribution" -->

note:
*(tag `2-deep-dive-created-multi-waiter-threads-restaurant`)*

* CREATE: MultiWaiterExecutorServiceRestaurant
* RUN: Main

---

<!-- .slide: data-auto-animate" -->

### Modeling a Restaurant with Threads

<pre data-id="restaurant-executorservice"><code class="java" data-trim data-line-numbers>
public class MultiWaiterThreadsRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        var starterThread = new WaiterAnnounceCourseThread(grover, CourseType.STARTER);
        var mainThread = new WaiterAnnounceCourseThread(zoe, CourseType.MAIN);
        var dessertThread = new WaiterAnnounceCourseThread(rosita, CourseType.DESSERT);

        starterThread.start();
        mainThread.start();
        dessertThread.start();

        starterThread.join();
        mainThread.join();
        dessertThread.join();

        return new MultiCourseMeal(starterThread.getAnnouncedCourse(),
                                   mainThread.getAnnouncedCourse(),
                                   dessertThread.getAnnouncedCourse());
    }
}
</code></pre>

---

<!-- .slide: data-auto-animate" -->

### Modeling a Restaurant with ExecutorService

<pre data-id="restaurant-executorservice"><code class="java" data-trim data-line-numbers="1-16|8|9-11|13">
public class MultiWaiterExecutorServiceRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var executor = Executors.newFixedThreadPool(3)) {
            Future&lt;Course&gt; starter = executor.submit(() -> grover.announceCourse(CourseType.STARTER));
            Future&lt;Course&gt; main = executor.submit(() -> zoe.announceCourse(CourseType.MAIN));
            Future&lt;Course&gt; dessert = executor.submit(() -> rosita.announceCourse(CourseType.DESSERT));

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get());
        }
    }
}
</code></pre>

note:

**line 8**

using 3 threads here (but we can easily change it, so we've now separated the unit-of-work and the run mechanism)

**line 9-11**

submitting three 'announceCourse' tasks here, that return Futures.

**line 13**

calling a blocking 'get' on the Futures will yield the 3 results and allow us to construct a `MultiCourseMeal`.

Looks good enough, right?

---

<!-- .slide: data-auto-animate" -->

### Modeling a Restaurant with ExecutorService

<pre data-id="restaurant-executorservice"><code class="java" data-trim data-line-numbers="1-16|10|9|13|10|13|9|1-16|9-11">
public class MultiWaiterRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var executor = Executors.newFixedThreadPool(3)) {
            Future&lt;Course&gt; starter = executor.submit(() -> grover.announceCourse(CourseType.STARTER));
            Future&lt;Course&gt; main = executor.submit(() -> zoe.announceCourse(CourseType.MAIN));
            Future&lt;Course&gt; dessert = executor.submit(() -> rosita.announceCourse(CourseType.DESSERT));

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get());
        }
    }
}
</code></pre>

note:

This means that the `announceCourse` method can throw an `OutOfStockException`!
Do we still think this piece of code is good enough?

**line 10**
* If `zoe.announceCourse(CourseType.MAIN)` takes a long time to execute...

**line 9**
* but `grover.announceCourse(CourseType.STARTER)` fails in the meantime... 
* the entire `announceMenu(..)` method will unnecessarily wait for the main course announcement by...

**line 13**
* blocking on `main.get()`, instead of canceling it (which would be the sensible thing to do).

<hr/>

**line 10**
* If an exception happens in `zoe.announceCourse(CourseType.MAIN)`...

**line 13**
* `main.get()` will throw it, but...

**line 9**
* `grover.announceCourse(CourseType.STARTER)` will continue to run even after `announceMenu()` has failed.

<hr/>

**all lines**
* If the thread executing `announceMenu(...)` is interrupted, the interruption will not propagate to the subtasks: 

**line 9-11**
* all threads that run an `announceCourse(..)` invocation will leak, continuing to run even after `announceMenu()` has failed.


---

<!-- .slide: data-auto-animate" -->

### Modeling a Restaurant with ExecutorService

<pre data-id="restaurant-executorservice"><code class="java" data-trim data-line-numbers>
public class MultiWaiterRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var executor = Executors.newFixedThreadPool(3)) {
            Future&lt;Course&gt; starter = executor.submit(() -> grover.announceCourse(CourseType.STARTER));
            Future&lt;Course&gt; main = executor.submit(() -> zoe.announceCourse(CourseType.MAIN));
            Future&lt;Course&gt; dessert = executor.submit(() -> rosita.announceCourse(CourseType.DESSERT));

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get());
        }
    }
}
</code></pre>

note:

* Ultimately the problem here is that our program is logically structured with task-subtask relationships, 
* (we need three completely assembled courses to return a multi-course meal)
* but these relationships exist only in the mind of the developer. 
* We just have to accept the fact that Java's ExecutorService is based on *unstructured concurrency*.

---

### Pros ✅

* task can directly return a value; <!-- .element: class="fragment fade-in-then-semi-out" -->
* workload and run mechanism are separated. <!-- .element: class="fragment fade-in-then-semi-out" -->
<br/>
<br/>
<br/>

### Cons ❌

<ul>
    <li class="fragment fade-in-then-semi-out">will wait for all tasks to terminate;</li>
    <li class="fragment fade-in-then-semi-out">allows unrestricted patterns of concurrency.</li>
</ul>

note:

**task can directly return a value**

because of the support for `Callable`.

**workload and run mechanism are separated**

making it easy to run the workload on a different thread configuration

**will wait for all tasks to terminate**

Even if one of them would fail or is cancelled!
The reason for this: the `submit()` method returns a `Future`, a Java construct that only supports blocking `get()`s.
Also: when we know for sure the desired result won't be achieved.

**allows unrestricted patterns of concurrency**

* it's very hard with ExecutorService to create relationships among tasks and subtasks
* but that's a valid use case that occurs quite often!
* Unfortunately, ExecutorService doesn't enforce any task structure
* In theory, one thread could create an ExecutorService, a second thread could submit work to it.
* And the threads which actually execute the work would have no relationship to either the first or second thread. 
* They are *one-way jumps*, just like the notorious `goto` statement from the `BASIC` language.
* If threads are spawned in an unstructured way, they are like the concurrent equivalent of `goto`!
* To conclude: **ExecutorService allows unrestricted patterns of concurrency.**

---

## CompletableFuture

<ul>
    <li class="fragment fade-in-then-semi-out">since Java 8; 
    <li class="fragment fade-in-then-semi-out">composes asynchronous operations;
    <li class="fragment fade-in-then-semi-out">handles eventual results in a declarative way;
    <li class="fragment fade-in-then-semi-out">aims to fix the problems that come with the <code>Future</code>.
</ul>

note:

**aims to fix the problems that come with the `Future`**

* it cannot be manually completed;
* you cannot perform further action on a Future’s result without blocking;
* multiple `Future`s cannot be chained together;
* doesn't support exception handling.

---

<!-- .slide: data-auto-animate" -->

### Modeling a Restaurant with ExecutorService

<pre data-id="restaurant-completablefuture"><code class="java" data-trim data-line-numbers>
public class MultiWaiterRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var executor = Executors.newFixedThreadPool(3)) {
            Future&lt;Course&gt; starter = executor.submit(() -> grover.announceCourse(CourseType.STARTER));
            Future&lt;Course&gt; main = executor.submit(() -> zoe.announceCourse(CourseType.MAIN));
            Future&lt;Course&gt; dessert = executor.submit(() -> rosita.announceCourse(CourseType.DESSERT));

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get());
        }
    }
}
</code></pre>

---

<!-- .slide: data-auto-animate" -->

### Modeling a Restaurant with CompletableFuture

<pre data-id="restaurant-completablefuture"><code class="java" data-trim data-line-numbers>
public class CompletableFutureRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws Exception {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        var starter = asFuture(() -> grover.announceCourse(CourseType.STARTER));
        var main = asFuture(() -> zoe.announceCourse(CourseType.MAIN));
        var dessert = asFuture(() -> rosita.announceCourse(CourseType.DESSERT));

        CompletableFuture.allOf(starter, main, dessert).join();

        return new MultiCourseMeal(starter.get(), main.get(), dessert.get());
    }

    public static &lt;T&gt; CompletableFuture&lt;T&gt; asFuture(Callable&lt;? extends T&gt; callable) {
        var future = new CompletableFuture&lt;&gt;();
        future.defaultExecutor().execute(() -> {
            try {
                future.complete(callable.call());
            } catch (Throwable t) {
                future.completeExceptionally(t);
            }
        });
        return future;
    }
}
</code></pre>

note:

CompletableFuture has specifically been designed for the asynchronous programming paradigm, where no blocking operations occur whatsoever. It's a way to circumvent limitations classic threads currently have, such as specifically waiting for an asynchronous operation to complete. Reactive frameworks like Akka or RxJava are based on the same principles.

Both ExecutorService and CompletableFuture offer mechanisms for chaining asynchronous tasks, but they take different approaches. 
  * In ExecutorService, we typically submit tasks for execution and then use the Future objects returned by these tasks to handle dependencies and chain subsequent tasks. However, this involves blocking and waiting for the completion of each task before proceeding to the next, which can lead to inefficiencies in handling asynchronous workflows.
  * On the other hand, CompletableFuture offers a more streamlined and expressive way to chain asynchronous tasks. It simplifies task chaining with built-in methods like `thenApply()`. These methods allow you to define a sequence of asynchronous tasks where the output of one task becomes the input for the next. 
  **An available thread (by default in the ForkJoin.commonPool) is *woken up* when the input is available for the next task.**

---

## ThreadLocal

* since Java 1.2; <!-- .element: class="fragment fade-in-then-semi-out" -->
* a variable that is unique to its thread; <!-- .element: class="fragment fade-in-then-semi-out" -->
* each thread has its own, independently initialized copy of the variable; <!-- .element: class="fragment fade-in-then-semi-out" -->
* can be used as the equivalent of a global variable in the threaded world. <!-- .element: class="fragment fade-in-then-semi-out" -->

note:

**can be used as the equivalent of a global variable in the threaded world**

* imagine you need a certain value in many places that must be unique to the Thread that uses it.
* if you want to avoid cluttering up your method signatures by passing it around, you could use a global variable
* but that would only work in the non-threaded world.
* passing it to all methods where you need them is an option, but would clutter your method signatures.
* so use a ThreadLocal instead! 

---

<!-- .slide: data-background="img/background/binary-code.jpg" data-background-color="black" data-background-opacity="0.3" -->

### Demo, Part 4

- Let's generate a few `announcementId`s with a `ThreadLocal`

<https://pxhere.com/en/photo/1458897> <!-- .element: class="attribution" -->

note:
*(tag `3-deep-dive-created-multi-waiter-executor-service-restaurant`)*

* CREATE: `AnnouncementId`
* USE: `Waiter`, `Chef`
* RUN: `Main`

---

### Generating `announcementId`s with ThreadLocal

<pre><code class="java" data-trim data-line-numbers>
public class AnnouncementId {
    private static final AtomicInteger nextId = new AtomicInteger(1);
    private static final ThreadLocal&lt;Integer&gt; announcementId = 
            ThreadLocal.withInitial(nextId::getAndIncrement);

    public static int get() {
        return announcementId.get();
    }
}
</code></pre>

note:

This class generates a unique `announcementId` local to each thread. 
An announcementId is assigned the first time it invokes AnnouncementId.get() and remains unchanged on subsequent calls by the same thread.

Each thread holds an implicit reference to its copy of a thread-local variable as long as the thread is alive and the ThreadLocal instance is accessible; after a thread goes away, all of its copies of thread-local instances are subject to garbage collection (unless other references to these copies exist).

---

#### Waiter
<pre><code class="java" data-trim data-line-numbers="1-9|5">
public Course announceCourse(CourseType courseType) {
    Course pickedCourse = Chef.pickCourse(name, courseType);

    System.out.format("[%s] Announcement #%d: Today's %s will be '%s'.%n", 
            name, AnnouncementId.get(), courseType.name().toLowerCase(), 
            pickedCourse);

    return pickedCourse;
}
</code></pre>

#### Chef <!-- .element: class="fragment" data-fragment-index="1" -->
<pre class="fragment" data-fragment-index="1"><code class="java" data-trim data-line-numbers="1-11|8">
public static Course pickCourse(String waiterName, CourseType courseType) {
    // MENU is a pre-populated Map&lt;CourseType, List&lt;Course&gt;&gt;.
    var courses = MENU.get(courseType);

    System.out.format("[Chef] %s asked me to pick a %s, so that " + 
            "announcement #%d can take place.%n",
            waiterName, courseType.name().toLowerCase(), 
            AnnouncementId.get());

    return courses.get(new Random().nextInt(courses.size()));
}
</code></pre>

---

### Pros ✅

* avoids cluttering method signatures; <!-- .element: class="fragment fade-in-then-semi-out" -->
* elegant way to bind data that is unique to the current thread. <!-- .element: class="fragment fade-in-then-semi-out" -->
<br/>
<br/>
<br/>

### Cons ❌

* always mutable; <!-- .element: class="fragment fade-in-then-semi-out" -->
* unbounded lifetime; <!-- .element: class="fragment fade-in-then-semi-out" -->
* memory-intensive, especially with many threads, or when inherited. <!-- .element: class="fragment fade-in-then-semi-out" -->

note:

**avoids cluttering method signatures**

Waiter and Chef could both easily read the same value.

**elegant way to bind data that is unique to the current thread**

Because it's bound to the thread by definition; no logic needed to achieve this.

**always mutable**

Every thread-local variable is mutable: any code that can call the `get()` method of a thread-local variable can call the set method of that variable at any time. 

**unbounded lifetime**

* Thread-local variables retain their values until explicitly removed using the "remove" method or until the thread terminates.
* The threads in our code example terminated quickly.
* But if threads live longer, ThreadLocals also live longer in memory.
* Developers might forget to call `remove()`, causing data to persist longer than necessary.
* This can be a security risk, especially in thread pools where data may unintentionally transfer between tasks. 
* For programs relying on mutable thread-local variables, finding a safe point to call "remove" can be challenging. This can cause a long-term memory leak.

**memory-intensive**

* Especially with many threads, or when inherited.
* Thread-local variables can be made inheritable (class `InheritableThreadLocal`).
* So if Thread A starts a thread called Thread B, then Thread B can access Thread A's inheritable thread-locals.
* But in order to make this happen, the child thread (Thread B) has to allocate (redundant) storage for every thread-local variable previously written in the parent thread. 
* This adds significant memory footprint.

---

## Honourable Mentions

<dl class="fragment fade-in-then-semi-out">
    <dt>ReentrantLock</dt>
    <dd>A more flexible and feature-rich alternative to the traditional <code>synchronized</code> keyword.</dd>
</dl>

<dl class="fragment fade-in-then-semi-out">
    <dt>ForkJoinPool</dt>
    <dd>A specialized implementation of <code>Executor</code>, designed for divide-and-conquer-style parallelism for compute-intensive workloads. Used by parallel streams.</dd>
</dl>

---

## Honourable Mentions

<dl class="fragment fade-in-then-semi-out">
    <dt>AtomicReference</dt>
    <dd>Manages an object reference by ensuring atomic, thread-safe operations without requiring explicit synchronization through locks.</dd>
</dl>

<dl class="fragment fade-in-then-semi-out">
    <dt>Semaphore</dt>
    <dd>Restricts thread access to a shared resource by handing out a limited number of permits.</dd>
</dl>

<dl class="fragment fade-in-then-semi-out">
    <dt>CountdownLatch</dt>
    <dd>Enables one or more threads to wait for a set of operations to complete before proceeding.</dd>
</dl>

notes:

**CountDownLatch**

An elegant way to wait until all threads have completed their work, without the need to call `Thread.join()` and having a reference to all threads that were created..

They can be excellent for integration tests that use multithreading, as a way to coordinate when it is time to run your assertions.

---

### Why Does This Stuff Interest Me?

<ul>
    <li class="fragment">Course: "Concurrency in Java"<br/>
        <small><a href="https://training.infosupport.com/en/trainingen/CONCURJAVA/concurrency-in-java/">training.infosupport.com/en/trainingen/CONCURJAVA/concurrency-in-java/</a></small>
    </li>
    <li class="fragment">Articles on <a href="https://foojay.io">foojay.io</a>:<br/>
        <small><a href="https://foojay.io/today/its-java-20-release-day-heres-whats-new/">foojay.io/today/its-java-20-release-day-heres-whats-new/</a></small><br/>
        <small><a href="https://foojay.io/today/java-21-is-available-today-and-its-quite-the-update/">foojay.io/today/java-21-is-available-today-and-its-quite-the-update/</a></small><br/>
        <small><a href="https://foojay.io/today/java-22-is-here-and-its-ready-to-rock/">foojay.io/today/java-22-is-here-and-its-ready-to-rock/</a></small><br/>
        <small><a href="https://foojay.io/today/java-23-has-arrived-and-it-brings-a-truckload-of-changes/">foojay.io/today/java-23-has-arrived-and-it-brings-a-truckload-of-changes/</a></small><br/>        
    </li>
</ul>

<img data-src="img/logos/java-community-logo.png" width="9%" class="no-background" style="margin-right: 2em">

<table class="fragment">
    <tr>
        <td style="text-align: right; vertical-align: middle;" width="45.3%">Hanno Embregts</td>
        <td style="text-align: left; padding: 0 0 0 0; vertical-align: middle;"><img width="16%" data-src="img/logos/ace-pro-spade.png" class="no-background" style="margin-top: 30px; vertical-align: middle;"/><img width="22%" data-src="img/logos/java-champion.png" class="no-background" style="margin-top: 30px; vertical-align: middle;"/></td>
        <td style="text-align: right;"><img width="45%" data-src="img/icons/twitter-white.png" class="no-background" style="margin-top: 35px"/></td>
        <td style="vertical-align: middle; padding: 0 0 0 0"><a href="https://www.twitter.com/hannotify">@hannotify</a></td>
    </tr>
</table>
<br/>

note:
But why does this stuff interest me?

* Apart from the fact that it feels nice to be the concurrency expert somewhere...
* My employer Info Support gives me the chance to combine Java consultancy with teaching courses.
* Course: "Concurrency in Java"
* New concurrency features in recent Java versions, on which I wrote a few articles.

While we're on the subject, let's finish introducing myself...

* I'm a Java Champion and an Oracle ACE.
* I am @hannotify on Twitter, Mastodon or Bluesky.

(Let's keep calling it Twitter, just to annoy Elon)
(About the handle: get notified of everything Hanno does. I thought it was rather clever - my wife disagrees with me though, she thinks I'm an major geek!)

* I post about things I like, as everyone does I suppose.
* Java Development, Concurrency, Version Control, Sustainability and making music.
* If you're into that stuff, by all means give me a follow wherever you like to keep track of your network!
