<!-- .slide: data-background="img/background/destination-arrows.jpg" data-background-color="black" data-background-opacity="0.3"-->

# Have We Come To Journey's End?  <!-- .element: class="stroke" -->

<https://www.pexels.com/photo/sign-arrow-direction-travel-52526/> <!-- .element: class="attribution" -->

note:

**Time Elapsed:** `44:00`.

So is Java's concurrency journey over now?
Have we come to journey's end?
What new features are expected to be developed after these?

---

## Possible Future Features

<ul>
    <li class="fragment fade-in-then-semi-out" data-fragment-index="1">sharing streams of data among threads ('channels');</li>
    <li class="fragment fade-in-then-semi-out" data-fragment-index="2">a new thread cancellation mechanism.
        <br/>
        <small class="fragment fade-in-then-semi-out" data-fragment-index="2">
        (both mentioned in <a href="https://openjdk.org/jeps/480">https://openjdk.org/jeps/480</a>)
        </small>    
    </li>
</ul>

note:

- **sharing streams of data among threads** ('channels')

> It is not a goal to define a means of sharing streams of data among threads (i.e., [channels](https://en.wikipedia.org/wiki/Channel_(programming))). We might propose to do so in the future.

- **a new thread cancellation mechanism**

> It is not a goal to replace the existing thread interruption mechanism with a new thread cancellation mechanism. We might propose to do so in the future.

Both topics were mentioned in JEP 480 ('Structured Concurrency') as possible future additions to the language.

---

## Confirmed Future Features

<ul>
    <li class="fragment fade-in-then-semi-out" data-fragment-index="1">synchronize virtual threads without pinning;</li>
        <small class="fragment fade-in-then-semi-out" data-fragment-index="1">
        (<a href="https://openjdk.org/jeps/491">https://openjdk.org/jeps/491</a>)
        </small>    
    <li class="fragment fade-in-then-semi-out" data-fragment-index="2">improvements to the Structured Concurrency API.</li>
        <small class="fragment fade-in-then-semi-out" data-fragment-index="2">
        (<a href="https://openjdk.org/jeps/8340343">https://openjdk.org/jeps/8340343</a>)
        </small>    
    </li>
</ul>

note:

- **solving thread pinning when virtual threads are executing a synchronized code block**

JEP 491 will allow a virtual thread to unmount when inside a synchronized method or statement, or when blocked on a monitor.
This will be done by allowing virtual threads to acquire, hold and release monitors, independently of their carriers.

- **evolution of the Structured Concurrency API**

The Loom Early-Access Builds has a few improvements that are now summarized in a JEP draft (8340343).
Both features will probably be a part of JDK 24.

---

<!-- .slide: data-auto-animate" -->

### Modeling a Restaurant with Structured Concurrency 

(JDK 21)

<pre data-id="new-sc-api-restaurant"><code class="java stretch" data-trim data-line-numbers="1-18|8,13">
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

<!-- .slide: data-auto-animate" -->

### Modeling a Restaurant with Structured Concurrency 

(JDK 24, probably)

<pre data-id="new-sc-api-restaurant"><code class="java stretch" data-trim data-line-numbers="8,13">
public class StructuredConcurrencyRestaurant implements Restaurant {
    @Override
    public MultiCourseMeal announceMenu() throws ExecutionException, InterruptedException {
        Waiter grover = new Waiter("Grover");
        Waiter zoe = new Waiter("Zoe");
        Waiter rosita = new Waiter("Rosita");

        try (var scope = StructuredTaskScope.open()) {
            var starter = scope.fork(() -> grover.announceCourse(CourseType.STARTER));
            var main = scope.fork(() -> zoe.announceCourse(CourseType.MAIN));
            var dessert = scope.fork(() -> rosita.announceCourse(CourseType.DESSERT));

            scope.join();

            return new MultiCourseMeal(starter.get(), main.get(), dessert.get());
        }
    }
}
</code></pre>

note:
* avoids using constructors like `new ShutdownOnFailure()` or `new ShutdownOnSuccess()`
* static factory method called `open()`
* no need for 'opting in' to throw Expection on failure; `join()` now throws any Exception that might be encountered.
* `open()` without arguments behaves like `ShutdownOnFailure`, but optionally takes a `Joiner` that determines the *policies* and *outcome* of this StructuredTaskScope.

---

<!-- .slide: data-auto-animate" -->

## Modeling a Bar With Structured Concurrency

(JDK 21)

<pre id="new-sc-api-bar"><code class="java stretch" data-trim data-line-numbers="1-14|7,11">
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

## Modeling a Bar With Structured Concurrency

(JDK 24, probably)

<pre id="new-sc-api-bar"><code class="java stretch" data-trim data-line-numbers="7,11">
public class StructuredConcurrencyBar implements Bar {
    @Override
    public DrinkOrder determineDrinkOrder(Guest guest) throws InterruptedException, ExecutionException {
        Waiter zoe = new Waiter("Zoe");
        Waiter elmo = new Waiter("Elmo");

        try (var scope = StructuredTaskScope.open(Joiner.&lt;DrinkOrder&gt;anySuccessfulResultOrThrow())) {
            scope.fork(() -> zoe.getDrinkOrder(guest, BEER, WINE, JUICE));
            scope.fork(() -> elmo.getDrinkOrder(guest, COFFEE, TEA, COCKTAIL, DISTILLED));

            return scope.join();
        }
    }
}
</code></pre>

note:

* here we pass a Joiner to the `open(..)` method to configure how the results should be handled.
* `scope.join()` immediately returns the result (or throws an Expection), a chaining `result()` method call is no longer necessary.

---

## Joiners

A **`Joiner<T,R>`** handles subtask completion and produces the result for the **`join`** method.

<span class="fragment">
<h3>Built-in joiners</h3>
<ul> 
    <li><code>anySuccessfulOrThrow()</code></li>
    <li><code>allSuccessfulOrThrow()</code></li>
    <li><code>awaitAll()</code></li>
    <li><code>awaitAllSuccessfulOrThrow()</code></li>
    <li><code>allUntil(Predicate&lt;Subtask&lt;T&gt;&gt; isDone)</code></li>
    <li class="fragment">...or create your own!</li>
</ul>

<small>(<a href="https://download.java.net/java/early_access/loom/docs/api/java.base/java/util/concurrent/StructuredTaskScope.Joiner.html">https://download.java.net/java/early_access/loom/docs/api/java.base/java/util/concurrent/StructuredTaskScope.Joiner.html</a>)</small>
</span>

