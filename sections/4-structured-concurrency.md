<!-- .slide: data-background="img/background/upcoming-station.jpg" data-background-color="black" data-background-opacity="0.5"-->

# Structured Concurrency  <!-- .element: class="stroke" -->

<https://www.pexels.com/photo/photography-of-railway-894312/> <!-- .element: class="attribution" -->

note:

**Time Elapsed:** `47:00`.

As we're leaving our current station 'Virtual Threads', let's see what the rest of the journey has in store for us.
It looks like 'Structured Concurrency' is one of the upcoming stations.
So all aboard! And let's check out what Structured Concurrency is all about.

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
        <tr class="greyed-out">
            <td><strong>21</strong></td>
            <td>Preview<br/></td>
            <td><a href="https://openjdk.java.net/jeps/453">JEP 453</a></td>
        </tr>
        <tr class="greyed-out">
            <td><strong>22</strong></td>
            <td>Second Preview<br/></td>
            <td><a href="https://openjdk.java.net/jeps/462">JEP 462</a></td>
        </tr>
        <tr>
            <td><strong>23</strong></td>
            <td>Third Preview<br/></td>
            <td><a href="https://openjdk.java.net/jeps/480">JEP 480</a></td>
        </tr>
    </tbody>
</table>

note:

It's an *upcoming station* indeed, because it's in 'preview' in Java 21.
Note that you need to pass the compiler option `--enable-preview` to be able to work with it.

---

<!-- .slide: data-auto-animate" -->

## What Problem Are We Trying To Solve?

<pre class="fragment" data-id="restaurant-single-threaded"><code class="java" data-trim data-line-numbers>
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

<pre data-id="restaurant-single-threaded"><code class="java" data-trim data-line-numbers>
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

It becomes clear from these examples that concurrent programming would be a lot easier (and more intuitive) if the task structure used would reflect code structure, like with single-threaded code.
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

<!-- .slide: data-background="img/background/unstructured-vs-structured.png" data-background-size="contain" data-background-color="black" -->

<https://auroratide.com/assets/posts/understanding-kotlin-coroutines/unstructured-vs-structured.png> <!-- .element: class="attribution" -->

note:

* I mentioned 'one-way jump' earlier.
* Spawning a thread is a 'one-way jump'.
* Structured Concurrency is a way to turn this into a 'two-way jump'.
* Threads in a Structured Concurrency context run in their own *scope* (the grey area to the right).
* They have clear *entry and exit points* (and only one of each).

---

<!-- .slide: data-background="img/background/concurrency-hierarchy.png" data-background-size="contain" data-background-color="black" -->

<https://auroratide.com/assets/posts/understanding-kotlin-coroutines/concurrency-hierarchy.png> <!-- .element: class="attribution" -->

note:

* They can also be organized in a clear *hierarchy*.
* Just like with function calls, a tree of threads can be created with parent-child relationships. 

---

## Scopes And Their Benefits

A scope continues until all child threads have completed,

resulting in: <!-- .element: class="fragment" -->

* a strict nesting of the lifetimes of operations; <!-- .element: class="fragment fade-in-then-semi-out" -->
* a streamlined error and cancellation propagation; <!-- .element: class="fragment fade-in-then-semi-out" -->
* improved reliability and observability in concurrent code. <!-- .element: class="fragment fade-in-then-semi-out" -->

note:

A scope continues until all child threads have completed (slide)
resulting in (slide)
* a strict nesting of the lifetimes of operations in a way that mirrors their syntactic nesting in the code. (slide)
* This streamlined error and cancellation propagation ultimately leads to (slide) improved reliability and observability in concurrent code.

To summarize: 
> Structured concurrency derives from the simple principle that: if a task splits into concurrent subtasks then they all return to the same place, namely the task's code block.

---

<!-- .slide: data-background="img/background/binary-code.jpg" data-background-color="black" data-background-opacity="0.3" -->

## Demo, Part 1

- Let's create a `StructuredConcurrencyRestaurant`

<https://pxhere.com/en/photo/1458897> <!-- .element: class="attribution" -->

note:
*(tag `0-demo-start`)*

Let's see structured concurrency in action!

- Let's create a `StructuredConcurrencyRestaurant`
- explain that `join()` blocks, `throwIfFailed` optionally throws, `get()` always returns a valid result
- explain the introduction of `Subtask` (meant for calling 'get()'s after a result is already known, unlike (Completable)Future)

*(tag `1-created-sc-restaurant`)*

---

### Modeling a Restaurant with Structured Concurrency

<pre><code class="java" data-trim data-line-numbers="1-19">
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
Which would very elegantly solve the following scenario:

---

<!-- .slide: data-background="img/background/bar-with-drinks.jpg" data-background-color="black" data-background-opacity="1.0"-->

<https://www.pexels.com/photo/stylish-interior-of-bar-in-restaurant-5490965/> <!-- .element: class="attribution" -->

note: 

(I actually got the idea for this talk on a Friday night, while I was out for drinks with a few friends.)

**Storytelling**

Ordering a drink with my Java Community coworkers, but no menu available.

...tell the story...

---

<!-- .slide: data-background="img/background/binary-code.jpg" data-background-color="black" data-background-opacity="0.3" -->

## Demo, Part 2

- Let's create a `StructuredConcurrencyBar`

<https://pxhere.com/en/photo/1458897> <!-- .element: class="attribution" -->

note:
*(tag `1-created-sc-restaurant`)*

- Let's create a `StructuredConcurrencyBar`
- Again: `join()` blocks
- `result()` returns the result of the first subtask that completed
- Run it and explain the behavior.

*(tag `2-created-sc-bar`)*

---

## Modeling a Bar With Structured Concurrency

<pre><code class="java" data-trim data-line-numbers>
public class StructuredConcurrencyBar implements Bar {
    @Override
    public DrinkOrder determineDrinkOrder(Guest guest) throws InterruptedException, ExecutionException {
        Waiter zoe = new Waiter("Zoe");
        Waiter elmo = new Waiter("Elmo");

        try (var scope = new StructuredTaskScope.ShutdownOnSuccess&lt;DrinkOrder&gt;()) {
            scope.fork(() -> zoe.getDrinkOrder(guest, BEER, WINE, JUICE));
            scope.fork(() -> elmo.getDrinkOrder(guest, COFFEE, TEA, COCKTAIL, DISTILLED));

            return scope.join().result();
        }
    }
}
</code></pre>

---

## ShutdownOnSuccess

- Shuts down the scope and returns the result when the first subtask succeeds;
- Also known as *invokeAny*.

note:

It effectively cancels any remaining tasks in the scope.

---

## Custom Shutdown Policies

<ul>
    <li class="fragment fade-in-then-semi-out">create your own shutdown policies by extending the class <code>StructuredTaskScope</code>;</li>
    <li class="fragment fade-in-then-semi-out">then implement the <code>handleComplete(..)</code> method;</li>
    <li class="fragment fade-in-then-semi-out">this allows you to have full control over when the scope shuts down and what results will be collected.</li>
</ul>

note:

Use case for a custom shutdown policy:

* collect results that succeed, ignore results that fail;
* more use cases are listed in the JEP.

---

<!-- .slide: data-background="https://media3.giphy.com/media/35DmSDIGveFaNoBffM/giphy.gif?cid=ecf05e47rnpqbg3pe1bk2pidk7twfqqawsexv72qsvcp8g50&ep=v1_gifs_related&rid=giphy.gif&ct=g" data-background-color="black" data-background-opacity="0.6"-->

## ...wait a minute! <!-- .element: class="stroke" -->

<blockquote class="explanation fragment">
At least <code>ExecutorService</code> allowed me to pass a thread configuration. Does Structured Concurrency support that at all?
</blockquote>

<https://media.giphy.com/media/35DmSDIGveFaNoBffM/giphy.gif> <!-- .element: class="attribution" -->

note:

Ah, we've come to the section I like to call "Sanity Checks".
Containing a few questions that I can imagine you have after hearing about all this.

(slide)

In a way, yes.

* StructuredTaskScope uses Virtual Threads by default, but you can pass a Platform Thread Factory to an overloaded StructuredTaskScope constructor.
* But it is a deliberate decision to not allow thread pools with certain sizes, because Structured Concurrency performs best with a `thread-per-request` approach

---

<!-- .slide: data-background="https://media3.giphy.com/media/Wgb2FpSXxhXLVYNnUr/giphy.gif?cid=ecf05e47cvfnmtykpjag4gqjo08clspaduvrztyb3huaanrm&ep=v1_gifs_search&rid=giphy.gif&ct=g" data-background-color="black" data-background-opacity="0.6"-->

## ...but wait a minute! <!-- .element: class="stroke" -->

<blockquote class="explanation fragment">
Why do we need all these new 'scope' classes? Why didn't they just enhance the <code>ExecutorService</code> to support Structured Concurrency?
</blockquote>

<https://media.giphy.com/media/Wgb2FpSXxhXLVYNnUr/giphy.gif> <!-- .element: class="attribution" -->

note:

(slide)
The new class `StructuredTaskScope` enforces structure and order upon concurrent operations. 
Thus it does not implement the ExecutorService or Executor interfaces since instances of those interfaces are commonly used in a non-structured way. 
So they opted not to change the ExecutorService interface because of **backwards compatibility**.
However, I expect it to be straightforward to migrate code that uses ExecutorService, but would benefit from structure, to use StructuredTaskScope.

---

<!-- .slide: data-background="https://media.giphy.com/media/l0Iy6nCgWE0b5Jh96/giphy.gif" data-background-color="black" data-background-opacity="0.6"-->

## ...now wait a minute! <!-- .element: class="stroke" -->

<blockquote class="explanation fragment">I know for a fact that <code>ExecutorService</code> already supports <code>invokeAll(..)</code> and <code>invokeAny(..)</code>. Why would I need Structured Concurrency? </blockquote>

<https://media.giphy.com/media/l0Iy6nCgWE0b5Jh96/giphy.gif> <!-- .element: class="attribution" -->

note:
(slide)
This is true, ExecutorService does support these operations:

**`List<Future<T>> ExecutorService.invokeAll(Collection<Callable>)`**
Executes the given tasks, returning a list of Futures when all complete (or failed).

**`T ExecutorService.invokeAny(Collection<Callable>)`**
Executes the given tasks, returning the result of one that has completed successfully (i.e., without throwing an exception), if any do. Upon normal or exceptional return, tasks that have not completed are cancelled.

---

### Well, there are differences.

<ul>
    <li class="fragment fade-in-then-semi-out">cancellation;</li>
    <li class="fragment fade-in-then-semi-out">creating a task hierarchy and limiting the scope of tasks;</li>
    <li class="fragment fade-in-then-semi-out">making the created tasks return to the same place and limiting resource leaks in the process;</li>
    <li class="fragment fade-in-then-semi-out">custom shutdown policies;</li>
    <li class="fragment fade-in-then-semi-out">virtual threads by default.</li>
</ul>

<br/>
<br/>

<small class="fragment">A more detailed comparison of ExecutorService and Structured Concurrency: 
<br/>
<a href="https://medium.com/@lavneesh.chandna/structured-concurrency-in-java-7a10b36ce0a3">https://medium.com/@lavneesh.chandna/structured-concurrency-in-java-7a10b36ce0a3</a>
</small>

note:
Well, there are differences.

**cancellation**
`ExecutorService.invokeAll(...)` doesn't support cancellation.
It can only wait until all tasks have finished (exceptionally).

**Time Elapsed:** `1:15:00`. If so, then it's time for a break!

If Time Permits: demonstrate this in the example domain by changing the following line in `StructuredConcurrencyRestaurant` and `MultiWaiterInvokeAllRestaurant`:

```java
var starter = scope.fork(() -> { throw new RuntimeException("sorry, no starters today"); });
```

**creating a task hierarchy and limiting the scope of tasks** 
parent tasks waiting for child tasks to complete

**making the created tasks return to the same place**
ES is still 'unstructured'

**custom shutdown policies**

**virtual threads by default**
although you could configure ES to use them

---

<!-- .slide: data-background="https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExZ3d0NHdsb2p4d3d3YjEzenJuaG1qbjE0NHhseHRueDMzNGEyenk1MSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9dg/A19tglYyfc7JPQYwNi/giphy.gif" data-background-color="black" data-background-opacity="0.6" data-background-size="contain"-->

## ...no really, wait a minute! <!-- .element: class="stroke" -->

<blockquote class="explanation fragment">
I can create multiple <code>CompletableFuture</code>s and wrap them in a <code>CompletableFuture.allOf()</code>, right? Why would I need Structured Concurrency?
</blockquote>

note: 

Well, here I can simply list the same reasons I did for ExecutorService.invokeAll():

- doesn't support cancellation
- can't create a task hierarchy / limit the scope of tasks
- can't make the created tasks return to the same place
- no custom shutdown policies
- no virtual threads by default (CF uses ForkJoinPool)

On top of that, CompletableFuture is designed for the asynchronous programming paradigm, whereas StructuredTaskScope encourages the blocking paradigm. To summarize, it was designed to offer degrees of freedom that are counterproductive in structured concurrency.

---

<!-- .slide: data-background="https://media.giphy.com/media/7Rlt5qEC1BlSXbSpae/giphy.gif" data-background-color="black" data-background-opacity="0.6" data-background-size="contain"-->

## ...seriously man, wait a minute! <!-- .element: class="stroke" -->

<blockquote class="explanation fragment">
Doesn't <code>ForkJoinPool</code> also impose structure on concurrent tasks? Why would I need Structured Concurrency?
</blockquote>

<https://media.giphy.com/media/7Rlt5qEC1BlSXbSpae/giphy.gif> <!-- .element: class="attribution" -->

note:
(slide)
Indeed, `ForkJoinPool` also imposes structure on concurrent tasks. 
However, that API is designed for compute-intensive tasks, whereas Structured Concurrency is specifically targeting towards tasks that involve I/O.

---

## To Summarize

<ul>
    <li class="fragment fade-in-then-semi-out"><em>Virtual threads</em> deliver an abundance of threads;</li>
    <li class="fragment fade-in-then-semi-out"><em>Structured concurrency</em> can correctly and robustly coordinate them.</li>
</ul>

notes: 

In summary, virtual threads deliver an abundance of threads. 
Structured concurrency can correctly and robustly coordinate them.
Having an API for structured concurrency in the JDK will make it easier to build maintainable & reliable server applications.
