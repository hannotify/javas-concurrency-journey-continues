<!-- .slide: data-background="img/background/upcoming-station.jpg" data-background-color="black" data-background-opacity="0.5"-->

# Scoped Values  <!-- .element: class="stroke" -->

<https://www.pexels.com/photo/photography-of-railway-894312/> <!-- .element: class="attribution" -->

note: 

**Time Elapsed:** `32:00`.

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
        <tr>
            <td><strong>21</strong></td>
            <td>Preview<br/></td>
            <td><a href="https://openjdk.java.net/jeps/446">JEP 446</a></td>
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

**are elegant way to bind data**

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
> Values that may be safely and efficiently shared to methods without using method parameters.

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

<!-- .slide: data-background="img/background/binary-code.jpg" data-background-color="black" data-background-opacity="0.3" -->

## Demo, Part 3

- Let's migrate `AnnouncementId` from a thread-local to a scoped value

<https://pxhere.com/en/photo/1458897> <!-- .element: class="attribution" -->

note:
*(tag `2-created-sc-bar`)*

Let's see scoped values in action!

- Let's migrate `AnnouncementId` from a thread-local to a scoped value
- We see that scoped values can only be accessed within the defined scope, leading to a limited lifetime ('how long is it accessible?') and access ('by who is it accessible?').
g
*(tag `3-introduced-scoped-value`)*

---

## Rebinding Scoped Values

<pre class="fragment"><code class="java stretch" data-trim data-line-numbers="1-15|4|8-10|9|14|10">
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

<pre class="fragment"><code class="java stretch" data-trim data-line-numbers="1-19|9|11-16|12-13|15">
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

---

<!-- .slide: data-background-color="#222" -->

## Scoped Values Can Be Useful For...  

- hidden method arguments; <!-- .element: class="fragment fade-in-then-semi-out" -->
- re-entrant code; <!-- .element: class="fragment fade-in-then-semi-out" -->
- nested transactions; <!-- .element: class="fragment fade-in-then-semi-out" -->
- graphics contexts. <!-- .element: class="fragment fade-in-then-semi-out" -->

<https://openjdk.org/jeps/446> <!-- .element: class="attribution" -->

note:

**hidden method arguments**

Like we've seen before.

**re-entrant code**

A scoped value can be used to detect or limit recursion.
You can use `ScopedValue.isBound()`  to check if the scoped value has a binding for the current thread.
In such a case the scoped value can model a recursion counter by being repeatedly rebound.

**nested transactions**

Detecting recursion can also be useful in the case of flattened transactions: Any transaction started while a transaction is in progress becomes part of the outermost transaction.

**graphics contexts** 

Another example occurs in graphics, where there is often a drawing context to be shared between parts of the program.
Scoped values, because of their automatic cleanup and re-entrancy, are better suited to this than thread-local variables.

---

<!-- .slide: data-background="img/background/think-while-on-laptop.jpg" data-background-color="black" data-background-opacity="0.5"-->

## When Should You Migrate? <!-- .element: class="stroke" -->

<blockquote class="explanation">
Are scoped values always a better choice than thread-locals, or just in certain situations?
</blockquote>

<https://www.pexels.com/photo/young-woman-using-laptop-at-home-3807747/> <!-- .element: class="attribution" -->

---

### ✅ Migrate when...

* your thread-local only deals with one-way transmission of unchanging data. <!-- .element: class="fragment fade-in-then-semi-out" data-fragment-index="2" -->

<br/>
<br/>
<br/>

### ❌ Don't migrate when... <!-- .element: class="fragment" data-fragment-index="3" -->

<ul>
    <li class="fragment fade-in-then-semi-out" data-fragment-index="4">your thread-local is used in a two-way fashion.</li>
    <small class="fragment fade-in-then-semi-out" data-fragment-index="4">(where a callee deep in the call stack transmits data to a faraway caller via <code>ThreadLocal.set</code>, or in a completely unstructured fashion)</small>
</ul>

---

## To Summarize

<ul>
    <li class="fragment fade-in-then-semi-out"><em>Thread-locals</em> don't limit mutability and might perform badly when many (virtual) threads are involved, because of their unbounded lifetime and memory-intensity when inherited;</li>
    <li class="fragment fade-in-then-semi-out"><em>Scoped values</em> support immutable values, and they come with both a strictly defined lifetime and an efficient way to be inherited (through structured concurrency).</li>
</ul>

notes: 

In summary, thread-locals don't limit mutability and might perform badly when many (virtual) threads are involved, because of their unbounded lifetime and memory-intensity when inherited.
Scoped values support immutable values that come with both a strictly defined lifetime and an efficient way to inherit them (through structured concurrency).

