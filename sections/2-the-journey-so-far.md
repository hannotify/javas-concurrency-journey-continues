<!-- .slide: data-background="img/background/boarding-a-train.jpg" data-background-color="black" data-background-opacity="0.4"-->

# The Journey So Far <!-- .element: class="stroke" -->

<https://www.pexels.com/photo/man-with-yellow-and-black-backpack-standing-near-train-1170181/> <!-- .element: class="attribution" -->

note:
**Time Elapsed:** `2:00`.

* Let's take a quick look at what concurrency features Java currently offers.
* Now, Java has been on a journey towards better concurrency features since the very beginning of the language.
* Taking that analogy a bit further, let's see where this journey began and through what stations we have traveled so far!
* Now, an easy-to-relate example domain always helps me to understand technical concepts.
* So let's use the bar story I just told, and extend it with a restaurant.
* We might become a bit hungry after we've had a few drinks, right?

---

<!-- .slide: data-background="img/background/grover.jpg" data-background-color="white" data-background-size="77%"-->

<a href="#" class="attribution" style="color: navy !important">(inspired by <em>Sesame Street</em>, hand-drawn by my wife Rianne)</a>

note:

* But our restaurant is not a very good one. 
* Things go wrong here.

Hand-drawn by my wife, btw! She's awesome! (opposites attract, I guess, my drawing has always been terrible, that's why I went into IT)

---

### Modeling a Restaurant with Threads

<pre><code class="java stretch" data-trim data-line-numbers>
public interface Restaurant {
    MultiCourseMeal announceMenu();
}
</code></pre>

<pre class="fragment"><code class="java stretch" data-trim data-line-numbers>
public class ThreadsMultiWaiterRestaurant implements Restaurant {
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
* The Thread is the basic building block of concurrency features in Java.

---

<pre><code class="java stretch" data-trim data-line-numbers>
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

* doing multiple things at once.
<br/>
<br/>
<br/>

### Cons ❌

* we can't return a value directly;
* the workload and mechanism to run it are one and the same.

note:

**Doing multiple things at once**

We can announce multiple courses at the same time.

**`Thread` can't return a value directly**

We'd need a `Callable` to be able to do that (from Java 1.5)

**workload and mechanism to run it are one and the same**

Workload (or: the task at hand, the unit-of-work)
One and the same, or to put it differently: they are tightly coupled

---

<!-- .slide: data-auto-animate" -->

### Modeling a Restaurant with Threads

<pre data-id="restaurant-executorservice"><code class="java stretch" data-trim data-line-numbers>
public class ThreadsMultiWaiterRestaurant implements Restaurant {
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
So let's introduce something more sophisticated: the ExcecutorService, introduced in Java 1.5.

---

<!-- .slide: data-auto-animate" -->

### Modeling a Restaurant with ExecutorService

<pre data-id="restaurant-executorservice"><code class="java stretch" data-trim data-line-numbers="1-16|8|9-11|13">
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

note:

**line 8**

using 3 threads here (but we can easily change it, so we've now separated the unit-of-work and the run mechanism)

**line 9-11**

submitting three 'announceCourse' tasks here, that return Futures.

**line 13**

calling a blocking 'get' on the Futures will yield the 3 results and allow us to construct a `MultiCourseMeal`.

Looks good enough, right?

---

<!-- .slide: data-background="img/background/summer-dinner.jpg" data-background-color="black" data-background-opacity="0.6"-->

<https://www.pexels.com/photo/chairs-dining-room-food-furniture-460537/> <!-- .element: class="attribution" -->

note: 

**Storytelling**

What if some ingredients are out of stock?

---

<!-- .slide: data-auto-animate" -->

### Modeling a Restaurant with ExecutorService

<pre data-id="restaurant-executorservice"><code class="java stretch" data-trim data-line-numbers="1-16|10|9|13|10|13|9|1-16|9-11">
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

<pre data-id="restaurant-executorservice"><code class="java stretch" data-trim data-line-numbers>
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

* task can directly return a value;
* workload and run mechanism are separated.
<br/>
<br/>
<br/>

### Cons ❌

* will wait for all tasks to terminate;
* allows unrestricted patterns of concurrency.

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

* ExecutorService doesn't enforce any task structure
* In theory, one thread could create an ExecutorService, a second thread could submit work to it.
* And the threads which actually execute the work would have no relationship to either the first or second thread. 
* Who remembers the classic programming language `BASIC`? And what's the most famous statement in that language?
* `goto`. It's a *one-way jump*.
* Spawning threads from an ExecutorService is like doing one-way jumps.
* They are like the concurrent equivalent of the `goto` statement.
* This is what I mean when I say: **ExecutorService allows unrestricted patterns of concurrency.**

---

## ThreadLocal

* since Java 1.2; 
* a variable that is unique to its thread;
* each thread has its own, independently initialized copy of the variable.

note:
Many frameworks use ThreadLocals to identify a user's request using a requestId, for example.

---

### Generating `announcementId`s with ThreadLocal

<pre><code class="java stretch" data-trim data-line-numbers>
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
<pre><code class="java stretch" data-trim data-line-numbers="1-9|5">
public Course announceCourse(CourseType courseType) {
    Course pickedCourse = Chef.pickCourse(name, courseType);

    System.out.format("[%s] Announcement #%d: Today's %s will be '%s'.%n", 
            name, AnnouncementId.get(), courseType.name().toLowerCase(), 
            pickedCourse);

    return pickedCourse;
}
</code></pre>

#### Chef <!-- .element: class="fragment" data-fragment-index="1" -->
<pre class="fragment" data-fragment-index="1"><code class="java stretch" data-trim data-line-numbers="1-11|8">
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

* avoids cluttering method signatures;
* elegant way to bind data that is unique to the current thread.
<br/>
<br/>
<br/>

### Cons ❌

* always mutable;
* unbounded lifetime;
* memory-intensive, especially with many threads, or when inherited.

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

<dl class="fragment fade-in-then-semi-out">
    <dt>CompletableFuture</dt>
    <dd>Simplifies asynchronous programming by defining a <em>chain</em> of operations. Each subsequent operation starts running after the first one has completed.</dl>

notes:

on **CompletableFuture**:

CompletableFuture has specifically been designed for the asynchronous programming paradigm, where no blocking operations occur whatsoever. It's a way to circumvent limitations classic threads currently have, such as specifically waiting for an asynchronous operation to complete. Reactive frameworks like Akka or RxJava are based on the same principles.

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

* My employer Info Support gives me the chance to combine Java development with teaching courses.
* (visit our booth to learn more)
* Course: "Concurrency in Java"
* New concurrency features in recent Java versions, on which I wrote a few articles.
* I'm a Java Champion and an Oracle ACE.
* I am @hannotify on Twitter, Mastodon or Bluesky.

(Let's keep calling it Twitter, just to annoy Elon)

* I post about things I like, as everyone does I suppose.
* Java Development, Concurrency, Version Control, Sustainability and making music.
* If you're into that stuff, give me a follow!
