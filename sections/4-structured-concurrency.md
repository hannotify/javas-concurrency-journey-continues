<!-- .slide: data-background="img/background/upcoming-station.jpg" data-background-color="black" data-background-opacity="0.5"-->

# Structured Concurrency  <!-- .element: class="stroke" -->

<https://www.pexels.com/photo/photography-of-railway-894312/> <!-- .element: class="attribution" -->

note:

**Time Elapsed:** `20:00`.

Upcoming Station: Structured Concurrency

---

## Feature Status

<table style="font-size: 100%">
    <thead>
        <tr>
            <th>Java version</th>
            <th>Feature status</th>
            <th>JEP</th>
        </tr>
    </thead>
    <tbody>
        <tr class="greyed-out">
            <td><strong>19</strong></td>
            <td>Incubator <br/></td>
            <td><a href="https://openjdk.java.net/jeps/428">JEP 428</a></td>
        </tr>        
        <tr class="greyed-out">
            <td><strong>20</strong></td>
            <td>Second Incubator <br/></td>
            <td><a href="https://openjdk.java.net/jeps/437">JEP 437</a></td>
        </tr>
        <tr>
            <td><strong>21</strong></td>
            <td>Preview<br/></td>
            <td><a href="https://openjdk.java.net/jeps/453">JEP 453</a></td>
        </tr>
    </tbody>
</table>

note:

It's an *upcoming station* indeed, because it's in 'preview' in Java 21.
Note that you need to pass the compiler option `--enable-preview` to be able to work with it.

---

<!-- .slide: data-auto-animate" -->

## What Problem Are We Trying To Solve?

<pre class="fragment" data-id="restaurant-single-threaded"><code class="java stretch" data-trim data-line-numbers>
public class MultiWaiterRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException {
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

note:

* But let's take a step back first..
* Let's remind ourselves of the problem that we're trying to solve here.

(slide)

* Do you remember this code example?
* We liked a few things here, but we also hated a few.
  * Like the unstructured concurrency.
* Ultimately the problem was that our program is logically structured with task-subtask relationships, 
  * (we need three completely assembled courses to return a multi-course meal)
  * but these relationships exist only in the mind of the developer. 
  * We just had to accept the fact that Java's `ExecutorService` is based on *unstructured concurrency*.
* In contrast, single-threaded code *always* enforces a hierarchy of tasks and subtasks.
* Let's consider a single-threaded version of our restaurant example.

---

<!-- .slide: data-auto-animate" -->

## Modeling a Restaurant With a Single Thread

<pre data-id="restaurant-single-threaded"><code class="java stretch" data-trim data-line-numbers>
public class SingleWaiterRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws OutOfStockException {
        Waiter elmo = new Waiter("Elmo");

        Course starter = elmo.announceCourse(CourseType.STARTER);
        Course main = elmo.announceCourse(CourseType.MAIN);
        Course dessert = elmo.announceCourse(CourseType.DESSERT);

        return new MultiCourseMeal(starter, main, dessert);
    }
}
</code></pre>

note:

* Looks like the waiter on call today is Elmo.
* And here we don’t have *any* of the problems we had before. 
* Elmo will announce the courses in exactly the right order, and if one subtask fails the remaining one(s) won’t even be started. 
* And because all work runs in the same thread, there is no risk of thread leakage.

It becomes clean from these examples that concurrenc programming would be a lot easier (and more intuitive) if the task structure used would reflect code structure, like with single-threaded code.
And *this* is where structured concurrency comes in!

---

<!-- .slide: data-background="img/background/spider-web.jpg" data-background-color="black" data-background-opacity="0.3"-->

## Structured Concurrency <!-- .element: class="stroke" -->

<blockquote class="explanation">
An approach to concurrent programming that preserves the natural relationship between tasks and subtasks, which leads to more readable, maintainable, and reliable concurrent code.
</blockquote>

<https://www.pexels.com/photo/spider-web-in-closeup-photo-1038278/> <!-- .element: class="attribution" -->

note:
(first read the definition out loud)
> An approach to concurrent programming that preserves the natural relationship between tasks and subtasks, which leads to more readable, maintainable, and reliable concurrent code.

The term *structured concurrency* was first coined by Martin Sústrik in 2016, when he worked on *goroutines* in the Go language. 
Independently, Roman Elizarov came up with the same concept, and his work ended up in the Kotlin language as *coroutines*. 
So the feature is already prominent in both Go and Kotlin, and is now also available in Java!

---

## Structured Concurrency

With structured concurrency, threads have:

* a hierarchy; <!-- .element: class="fragment fade-in-then-semi-out" -->
* their own scope; <!-- .element: class="fragment fade-in-then-semi-out" -->
* clear entry and exit points. <!-- .element: class="fragment fade-in-then-semi-out" -->

note:

In a structured concurrency approach, threads have a clear *hierarchy*, their own *scope*, and clear *entry and exit points*. 
Just like with function calls, a tree of threads is created with parent-child relationships. 

---

## Scopes And Their Benefits

A scope continues until all child threads have completed,

resulting in: <!-- .element: class="fragment" -->

* a strict nesting of the lifetimes of operations; <!-- .element: class="fragment fade-in-then-semi-out" -->
* a streamlined error and cancellation propagation; <!-- .element: class="fragment fade-in-then-semi-out" -->
* improved reliability and observability in concurrent code. <!-- .element: class="fragment fade-in-then-semi-out" -->

note:

A scope continues until all child threads have completed. 
Structured concurrency yields a strict nesting of the lifetimes of operations in a way that mirrors their syntactic nesting in the code. 
This streamlined error and cancellation propagation ultimately leads to improved reliability and observability in concurrent code.

To summarize: 
> Structured concurrency derives from the simple principle that: if a task splits into concurrent subtasks then they all return to the same place, namely the task's code block.

---

<!-- .slide: data-background="img/background/binary-code.jpg" data-background-color="black" data-background-opacity="0.3" -->

## Demo, Part 1

- Let's create a `StructuredConcurrencyRestaurant`
- Compare the behavior of `MultiWaiterRestaurant` with this new one

<https://pxhere.com/en/photo/1458897> <!-- .element: class="attribution" -->

note:
[TODO] Rehearse demo and sets appropriate Git tags.

Let's see structured concurrency in action!

- Let's create a `StructuredConcurrencyRestaurant`
- explain that `join()` blocks, `throwIfFailed` optionally throws, `get()` always returns a valid result
- explain the introduction of `Subtask` (meant for calling 'get()'s after a result is already known, unlike (Completable)Future)
- Compare the behavior of `MultiWaiterRestaurant` with this new one

---

### Modeling a Restaurant with Structured Concurrency

<pre><code class="java stretch" data-trim data-line-numbers="1-19">
public class StructuredConcurrencyRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            var starter = scope.fork(() -> grover.announceCourse(CourseType.STARTER));
            var main = scope.fork(() -> zoe.announceCourse(CourseType.MAIN));
            var dessert = scope.fork(() -> rosita.announceCourse(CourseType.DESSERT));

            scope.join().throwIfFailed();

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get());
        }
    }
}
</code></pre>

---

## ShutdownOnFailure

- Shuts down the scope when the first subtask fails;
- Also known as *invokeAll*.

note:

Structured concurrency uses short-circuiting patterns to avoid doing unnecessary work.
These patterns are supported by shutdown policies, implemented by subclasses of `StructuredTaskScope`.
We've used the `ShutdownOnFailure` policy in the demo.
A second subclass exists: `ShutdownOnSuccess`.
Let me introduce a use case for that one first.

---

<!-- .slide: data-background="img/background/bar-with-drinks.jpg" data-background-color="black" data-background-opacity="1.0"-->

<https://www.pexels.com/photo/stylish-interior-of-bar-in-restaurant-5490965/> <!-- .element: class="attribution" -->

note: 

**Storytelling**

Ordering a drink with my Java Community coworkers, but no menu available.

---

<!-- .slide: data-background="img/background/binary-code.jpg" data-background-color="black" data-background-opacity="0.3" -->

## Demo, Part 2

- Let's create a `StructuredConcurrencyBar`

<https://pxhere.com/en/photo/1458897> <!-- .element: class="attribution" -->

note:
[TODO] Rehearse demo and sets appropriate Git tags.

- Let's create a `StructuredConcurrencyBar`
- Again: `join()` blocks
- `result()` returns the result of the first subtask that completed
- Run it and explain the behavior.

---

## ShutdownOnSuccess

- Shuts down the scope and returns the result when the first subtask succeeds;
- Also known as *invokeAny*.

note:

It effectively cancels any remaining tasks in the scope.

---

## Custom Shutdown Policies

* create your own shutdown policies by extending the class `StructuredTaskScope`; <!-- .element: class="fragment fade-in-then-semi-out" -->
* implement the `handleComplete(..)` method; <!-- .element: class="fragment fade-in-then-semi-out" -->
* this allows you to have full control over when the scope shuts down and what results will be collected. <!-- .element: class="fragment fade-in-then-semi-out" -->

note:

Use cases for a custom shutdown policy:

* collect results that succeed, ignore results that fail;
* .. TODO (from JEP)
* ..

---

<!-- .slide: data-background="https://media3.giphy.com/media/35DmSDIGveFaNoBffM/giphy.gif?cid=ecf05e47rnpqbg3pe1bk2pidk7twfqqawsexv72qsvcp8g50&ep=v1_gifs_related&rid=giphy.gif&ct=g" data-background-color="black" data-background-opacity="0.6"-->

## ...Wait A Minute! <!-- .element: class="stroke" -->

<blockquote class="explanation fragment">
At least <code>ExecutorService</code> allowed me to pass a thread configuration. Does Structured Concurrency support that at all?
</blockquote>

<https://media.giphy.com/media/35DmSDIGveFaNoBffM/giphy.gif> <!-- .element: class="attribution" -->

note:

In a way, yes.

* StructuredTaskScope uses Virtual Threads by default, but you can pass a Platform Thread Factory to an overloaded StructuredTaskScope constructor.
* But it is a deliberate decision to not allow thread pools with certain sizes, because Structured Concurrency performs best with a `thread-per-request` approach

---

<!-- .slide: data-background="https://media3.giphy.com/media/Wgb2FpSXxhXLVYNnUr/giphy.gif?cid=ecf05e47cvfnmtykpjag4gqjo08clspaduvrztyb3huaanrm&ep=v1_gifs_search&rid=giphy.gif&ct=g" data-background-color="black" data-background-opacity="0.6"-->

## ...But Wait A Minute! <!-- .element: class="stroke" -->

<blockquote class="explanation fragment">
Why do we need all these new 'scope' classes? Why didn't they just enhance the <code>ExecutorService</code> to support Structured Concurrency?
</blockquote>

<https://media.giphy.com/media/Wgb2FpSXxhXLVYNnUr/giphy.gif> <!-- .element: class="attribution" -->

note:

The new class `StructuredTaskScope` enforces structure and order upon concurrent operations. 
Thus it does not implement the ExecutorService or Executor interfaces since instances of those interfaces are commonly used in a non-structured way. 
So they opted not to change the ExecutorService interface because of backwards compatibility.
However, I expect it to be straightforward to migrate code that uses ExecutorService, but would benefit from structure, to use StructuredTaskScope.

---

<!-- .slide: data-background="https://media.giphy.com/media/l0Iy6nCgWE0b5Jh96/giphy.gif?cid=ecf05e47sk6l7mtj7g0a6ezkktriqotxwju48ep1ydb2vkdw&ep=v1_gifs_related&rid=giphy.gif&ct=g" data-background-color="black" data-background-opacity="0.6"-->

## ...Now Wait A Minute! <!-- .element: class="stroke" -->

<blockquote class="explanation fragment">
<code>ExecutorService</code> already supports <code>invokeAll(..)</code> and <code>invokeAny(..)</code>. Why would I still need Structured Concurrency then? 
</blockquote>

<https://media.giphy.com/media/l0Iy6nCgWE0b5Jh96/giphy.gif> <!-- .element: class="attribution" -->

note:
TODO

* Try using `invokeAny()` and `invokeAll()`.
  * TODO: pros?
  * TODO: cons?
  * TODO: Hoe is `ShutdownOnFailure` beter of leesbaarder dan `ExecutorService.invokeAll()`?
    - lees: https://davidvlijmincx.com/posts/loom/invoke-all-with-virtual-threads/ 
    - lees: https://medium.com/@lavneesh.chandna/structured-concurrency-in-java-7a10b36ce0a3
  * TODO: Hoe is `ShutdownOnSuccess` beter of leesbaarder dan `ExecutorService.invokeAny()`?

---

<!-- .slide: data-background="https://media.giphy.com/media/7Rlt5qEC1BlSXbSpae/giphy.gif?cid=ecf05e47nzety4zj5qzhoe66og8xnltlk13aymuubq6jyhj6&ep=v1_gifs_related&rid=giphy.gif&ct=g" data-background-color="black" data-background-opacity="0.6"-->

## ...No Really, Wait A Minute! <!-- .element: class="stroke" -->

<blockquote class="explanation fragment">
Doesn't <code>ForkJoinPool</code> also impose structure on concurrent tasks? Why would I still need Structured Concurrency then?
</blockquote>

<https://media.giphy.com/media/7Rlt5qEC1BlSXbSpae/giphy.gif> <!-- .element: class="attribution" -->

note:

Indeed, `ForkJoinPool` also imposes structure on concurrent tasks. 
However, that API is designed for compute-intensive tasks, whereas Structured Concurrency is meant for tasks which involve I/O.

---

## To Summarize

<ul>
    <li class="fragment fade-in-then-semi-out"><em>Virtual threads</em> deliver an abundance of threads;</li>
    <li class="fragment fade-in-then-semi-out"><em>Structured concurrency</em> can correctly and robustly coordinate them.</li>
</ul>

* *Virtual threads* deliver an abundance of threads; <!-- .element: class="fragment fade-in-then-semi-out" -->
* *Structured concurrency* can correctly and robustly coordinate them. <!-- .element: class="fragment fade-in-then-semi-out" -->

notes: 

In summary, virtual threads deliver an abundance of threads. 
Structured concurrency can correctly and robustly coordinate them.
Having an API for structured concurrency in the JDK will make it easier to build maintainable & reliable server applications.
