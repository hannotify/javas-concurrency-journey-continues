<!-- .slide: data-background="img/background/upcoming-station.jpg" data-background-color="black" data-background-opacity="0.5"-->

# Scoped Values  <!-- .element: class="stroke" -->

<https://www.pexels.com/photo/photography-of-railway-894312/> <!-- .element: class="attribution" -->

note: 

**Time Elapsed:** `1:55:00`.

Another station that we'll visit today on our continued journey is 'Scoped Values'.

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
            <td><strong>20</strong></td>
            <td>Incubator <br/></td>
            <td><a href="https://openjdk.java.net/jeps/429">JEP 429</a></td>
        </tr>
        <tr class="greyed-out">
            <td><strong>21</strong></td>
            <td>Preview<br/></td>
            <td><a href="https://openjdk.java.net/jeps/446">JEP 446</a></td>
        </tr>
        <tr class="greyed-out">
            <td><strong>22</strong></td>
            <td>Second Preview<br/></td>
            <td><a href="https://openjdk.java.net/jeps/464">JEP 464</a></td>
        </tr>
        <tr>
            <td><strong>23</strong></td>
            <td>Third Preview<br/></td>
            <td><a href="https://openjdk.java.net/jeps/481">JEP 481</a></td>
        </tr>
    </tbody>
</table>

note:

Also an *upcoming station*, because it's in 'preview' in Java 21.
Again: you need to pass the compiler option `--enable-preview` to be able to work with it.

---

## What Problem Are We Trying To Solve?

note:

* But let's take a step back first..
* Let's remind ourselves of the problem that we're trying to solve here.
* Earlier in this talk, we added unique `AnnouncementIds` to our restaurant code with a ThreadLocal.
* Remember?
* What did we think about them?

---

### Thread-locals...

* ✅ avoid cluttering method signatures; <!-- .element: class="fragment fade-in-then-semi-out" -->
* ✅ are an elegant way to bind data that is unique to the current thread; <!-- .element: class="fragment fade-in-then-semi-out" -->
* ❌ are always mutable; <!-- .element: class="fragment fade-in-then-semi-out" -->
* ❌ have an unbounded lifetime; <!-- .element: class="fragment fade-in-then-semi-out" -->
* ❌ are memory-intensive, especially with many threads, or when inherited. <!-- .element: class="fragment fade-in-then-semi-out" -->

note:

**avoid cluttering method signatures**

Waiter and Chef could both easily read the same value.

**are elegant way to bind data that is unique to the current thread**

Because it's bound to the thread by definition; no logic needed to achieve this.

**are always mutable**

Every thread-local variable is mutable: any code that can call the `get()` method of a thread-local variable can call the set method of that variable at any time. 

**have an unbounded lifetime**

* Thread-local variables retain their values until explicitly removed using the "remove" method or until the thread terminates.
* The threads in our code example terminated quickly.
* But if threads live longer, ThreadLocals also live longer in memory.
* Developers might forget to call `remove()`, causing data to persist longer than necessary.
* This can be a security risk, especially in thread pools where data may unintentionally transfer between tasks. 
* For programs relying on mutable thread-local variables, finding a safe point to call "remove" can be challenging. This can cause a long-term memory leak.

**are memory-intensive**

* Thread-local variables can be made inheritable (class `InheritableThreadLocal`).
* So if Thread A starts a thread called Thread B, then Thread B can access Thread A's inheritable thread-locals.
* But in order to make this happen, the child thread (Thread B) has to allocate (redundant) storage for every thread-local variable previously written in the parent thread. 
* This adds significant memory footprint.
* This effect is amplified now that virtual threads are available.

So let's see if these newly added 'Scoped Values' can help us out here.

---

<!-- .slide: data-background="img/background/spider-web.jpg" data-background-color="black" data-background-opacity="0.3"-->

## Scoped Values <!-- .element: class="stroke" -->

<blockquote class="explanation">
Values unique to a thread that may be safely and efficiently shared to methods without using method parameters.
</blockquote>

<https://www.pexels.com/photo/spider-web-in-closeup-photo-1038278/> <!-- .element: class="attribution" -->

note:
(first read the definition out loud)

* They are effectively an implicit method parameter.
* It is "as if" every method in a sequence of calls has an additional, invisible, parameter. 
* This reminds us of thread-locals, their purpose seemed similar.
* But scoped values additionally offer some improvements over thread-locals.

---

## Scoped Values Are...

<ul>
    <li class="fragment fade-in-then-semi-out"><em>immutable</em>; they are written once and can be read multiple times;</li>
    <li class="fragment fade-in-then-semi-out"><em>comprehensible</em>; they have a limited lifetime, which is made visible from the syntactic structure of code;</li>
    <li class="fragment fade-in-then-semi-out"><em>robust</em>; data shared by a caller can be retrieved only by legitimate callees;</li>
</ul>

---

## And This Is How They Look!

<pre class="fragment"><code class="java" data-trim>
private static final ScopedValue&lt;String&gt; X = ScopedValue.newInstance();

void foo() {
    ScopedValue.where(X, "hello").run(() -> bar());
}

void bar() {
    System.out.println(X.get()); // prints hello
}
</code></pre>

---

<!-- .slide: data-background="img/background/binary-code.jpg" data-background-color="black" data-background-opacity="0.3" -->

### Demo, Part 8

- Let's migrate `AnnouncementId` from a thread-local to a scoped value

<https://pxhere.com/en/photo/1458897> <!-- .element: class="attribution" -->

note:
*(tag `7-deep-dive-created-multi-waiter-invoke-all-restaurant-demonstrated-cancellation`)*

* CHANGE: `AnnouncementId` to have a ScopedValue instead of a ThreadLocal
* CALL: `ScopedValue.where(..)` in `Waiter.announceCourse(courseType)`.
* CALL: `ScopedValue.get()` outside of the scope to show what happens.
* EXPLAIN: We see that scoped values can only be accessed within the defined scope, leading to a limited lifetime ('how long is it accessible?') and access ('by who is it accessible?').

---

### AnnouncementId
<pre><code class="java" data-trim data-line-numbers="1-4|2,4">
    private static final AtomicInteger nextId = new AtomicInteger(1);
    private static final ScopedValue&lt;Integer&gt; scopedValue = ScopedValue.newInstance();
    public static int nextId() { return nextId.getAndIncrement(); }
    public static ScopedValue&lt;Integer&gt; scopedValue() { return scopedValue; }
    public static int get() { return scopedValue.get(); }
</code></pre>

### Waiter <!-- .element: class="fragment" data-fragment-index="1" -->
<pre class="fragment" data-fragment-index="1"><code class="java" data-trim data-line-numbers="1-16|4-6,13|10">
public Course announceCourse(CourseType courseType) throws Exception {
    if (!introduced) introduce();

    return ScopedValue
            .where(AnnouncementId.scopedValue(), AnnouncementId.nextId())
            .call(() -> announce(courseType));
}

private Course announce(CourseType courseType) throws OutOfStockException {
    Course pickedCourse = Chef.pickCourse(name, courseType);

    System.out.format("[%s] Announcement #%d: Today's %s will be '%s'.%n", name, 
            AnnouncementId.get(), courseType.name().toLowerCase(), pickedCourse);

    return pickedCourse;
}
</code></pre>

---

### Chef
<pre><code class="java" data-trim data-line-numbers="1-11|8">
public static Course pickCourse(String waiterName, CourseType courseType) throws OutOfStockException {
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

## Rebinding Scoped Values

<pre class="fragment"><code class="java" data-trim data-line-numbers="1-15|4|8-10|9|14|10">
private static final ScopedValue&lt;String&gt; X = ScopedValue.newInstance();

void foo() {
    ScopedValue.where(X, "hello").run(() -> bar());
}

void bar() {
    System.out.println(X.get()); // prints hello
    ScopedValue.where(X, "goodbye").run(() -> baz());
    System.out.println(X.get()); // prints hello
}

void baz() {
    System.out.println(X.get()); // prints goodbye
}
</code></pre>

<https://openjdk.org/jeps/446#Rebinding-scoped-values> <!-- .element: class="attribution" -->

note:

* Scoped values are immutable (they have no `set()` method), but it is possible to bind a new value to an existing scoped value.

(slide)

* Code example from JEP 446.
* Explain the highlighted lines.

---

## Inheriting Scoped Values

<pre class="fragment"><code class="java" data-trim data-line-numbers="1-19|9|11-16|12-13">
public class StructuredConcurrencyBar implements Bar {
    private static final ScopedValue&lt;Integer&gt; drinkOrderId = ScopedValue.newInstance();

    @Override
    public DrinkOrder determineDrinkOrder(Guest guest) throws Exception {
        Waiter zoe = new Waiter("Zoe");
        Waiter elmo = new Waiter("Elmo");

        return ScopedValue.where(drinkOrderId, 1)
            .call(() -> {
                try (var scope = new StructuredTaskScope.ShutdownOnSuccess&lt;DrinkOrder&gt;()) {
                    scope.fork(() -> zoe.getDrinkOrder(guest, BEER, WINE, JUICE));
                    scope.fork(() -> elmo.getDrinkOrder(guest, COFFEE, TEA, COCKTAIL, DISTILLED));

                    return scope.join().result();
                }
            });
    }
}
</code></pre>

note:

* Scoped values can be inherited by child threads.
* Unlike with thread-local variables, there is no copying of a parent thread's scoped value bindings to the child thread.

(slide)

* Scoped values in the parent thread are automatically inherited by child threads created with StructuredTaskScope.
* Legacy thread management classes such as ForkJoinPool do not support inheritance of ScopedValues because they cannot guarantee that a child thread forked from some parent thread scope will exit before the parent leaves that scope.
* With structured concurrency, this CAN be guaranteed, which is the reason scoped value inheriting is possible with this mechanism.
* It's very nice to see these two new features work so well together!

---

<!-- .slide: data-background="img/background/think-while-on-laptop.jpg" data-background-color="black" data-background-opacity="0.5"-->

## When Should You Migrate? <!-- .element: class="stroke" -->

<blockquote class="explanation">
Are scoped values always a better choice than thread-locals, or just in certain situations?
</blockquote>

<https://www.pexels.com/photo/young-woman-using-laptop-at-home-3807747/> <!-- .element: class="attribution" -->

---

### ✅ Migrate when...

* your thread-locals only deal with one-way transmission of unchanging data; <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="1" -->
* you expect to use many virtual threads. <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="2" -->

<br/>
<br/>
<br/>

### ❌ Don't migrate when... <!-- .element: class="fragment" data-fragment-index="3" -->

<ul>
    <li class="fragment fade-in-then-semi-out" data-fragment-index="4">your thread-local is used in a two-way fashion;</li>
    <li class="fragment fade-in-then-semi-out" data-fragment-index="5">you don't expect to use virtual threads.</li>
</ul>

---

## To Summarize

<h3>Thread-locals</h3>
<ul>
    <li>don't limit mutability;</li>
    <li>perform badly with many (virtual) threads.</li>
</ul>
<br/>
<br/>
<div class="fragment">
    <h3>Scoped values</h3>
    <ul>
        <li>support immutable values;</li>
        <li>come with a strictly defined lifetime;</li>
        <li>can be inherited efficiently through <em>structured concurrency</em>.</li>
    </ul>
</div>

notes: 

In summary, thread-locals don't limit mutability and might perform badly when many (virtual) threads are involved, because of their unbounded lifetime and memory-intensity when inherited. 
*(slide)*
Scoped values support immutable values that come with both a strictly defined lifetime and an efficient way to inherit them (through structured concurrency).

