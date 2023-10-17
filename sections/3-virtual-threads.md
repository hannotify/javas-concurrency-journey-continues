<!-- .slide: data-background="img/background/current-station.jpg" data-background-color="black" data-background-opacity="0.7"-->

# Virtual Threads <!-- .element: class="stroke" -->

<https://www.pexels.com/photo/photo-of-train-station-1824169/> <!-- .element: class="attribution" -->

note:
**Time Elapsed:** `15:00`.

* The features I covered so far have been a part of Java for a while now.
* They were part of 'our journey so far'.
* I guess it's safe to say our current station is 'virtual threads'.
* Being made final in Java 21.

---

## Feature Status

TODO

---

## Virtual Threads

TODO

---

TODO: photo slide with trains

---

TODO: photo slide with taxis

---

<!-- .slide: data-auto-animate" -->

### Modeling a Restaurant with ExecutorService

<pre data-id="restaurant-virtual-threads"><code class="java stretch" data-trim data-line-numbers>
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

### Modeling a Restaurant with Virtual Threads

<pre data-id="restaurant-virtual-threads"><code class="java stretch" data-trim data-line-numbers="1-16|8">
public class MultiWaiterRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            Future&lt;Course&gt; starter = executor.submit(() -> grover.announceCourse(CourseType.STARTER));
            Future&lt;Course&gt; main = executor.submit(() -> zoe.announceCourse(CourseType.MAIN));
            Future&lt;Course&gt; dessert = executor.submit(() -> rosita.announceCourse(CourseType.DESSERT));

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get());
        }
    }
}
</code></pre>

---

### Pros ✅

* TODO <!-- .element: class="fragment fade-in-then-semi-out" -->

### Cons ❌

* TODO <!-- .element: class="fragment fade-in-then-semi-out" -->

note:

- Pros: millions of threads can run, allows thread-per-request style instead of thread-sharing style, can significantly improve application throughput when the number of concurrent tasks is high (> a few thousand) and the workload is not CPU-bound
- Cons: pinned threads with synchronization blocks, error propagation, communicating cancellation intent, thread-local variables perform badly with many (virtual) threads

---

## What Problems Are Getting Worse?

* TODO <!-- .element: class="fragment fade-in-then-semi-out" -->

note: 

TODO: in the problem-solution style.

* LockSupport, Semaphores enable _parking_ virtual threads.
* TODO: something something, no solution get (tell you a secret: structured concurrency might solve it)
* thread-local variables perform badly with many (virtual) thread, no solution yet (but tell you a secret: scoped values might solve it).

> This is an important part of the talk, because these are the reasons why Java's support for concurrency is being extended.

> Few applications tend to use ThreadLocal variables. In a nutshell, Java ThreadLocal variables are created and stored as variables within the scope of a particular thread alone, and they cannot be accessed by other threads. If your application creates millions of virtual threads, and each virtual thread has its own ThreadLocal variable, then it can quickly consume java heap memory space. Thus, you want to be cautious of the size of data that is stored as ThreadLocal variables.
>
> You might wonder why ThreadLocal variables are not problematic in platform threads. The difference is that in platform threads, we don’t create millions of threads, whereas, in virtual threads, we do. Millions of threads, each with its own copy of ThreadLocal variables, can quickly fill up memory. They say small drops of water make an ocean. It’s very true here.

---

## Is This The Final Station?

TODO