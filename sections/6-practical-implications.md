<!-- .slide: data-background="img/background/upcoming-station.jpg" data-background-color="black" data-background-opacity="0.5"-->

# Practical Implications  <!-- .element: class="stroke" -->

<https://www.pexels.com/photo/photography-of-railway-894312/> <!-- .element: class="attribution" -->

note:

**Time Elapsed:** `2:25:00`.

How will these features change your day-to-day life?
That depends on what kind of Java developer you are:

* Developer Advocate / Trainer
* Developer (Framework or Library)
* Developer (Jakarta EE)
* Developer (Spring)
* Developer (Quarkus)

---

## Developer Advocate / Trainer

You can just get started trying them out and talking about them! ☺️

---

## Developer (framework or library)

* You can start using the features to enhance your product; <!-- .element: class="fragment fade-in-then-semi-out" -->
* Keep in mind that Virtual Threads is the only feature that is finalized; <!-- .element: class="fragment fade-in-then-semi-out" -->
* Structured Concurrency and Scoped Values are both still in preview, so they're not permanent parts of the language yet. <!-- .element: class="fragment fade-in-then-semi-out" -->

note:

## Preview status

Ensures that features are "done right" before they become final and permanent parts of the language by providing a suitable window of time in which features can be improved based on user feedback.

* fully specified and implemented
* impermanent
* meant to provoke developer feedback based on real world use
* may lead to it becoming permanent (but can in theory also be removed in the end)

---

<!-- .slide: data-background-color="#222" -->

## Developer (Spring)

<div class="fragment">
<code>SimpleTaskExecutor.setVirtualThreads(boolean virtual)</code>
<small>(became available in Spring 6.1)</small>
</div>

<https://spring.io/blog/2022/10/11/embracing-virtual-threads> <!-- .element: class="attribution" -->

note:
When you use a Java application framsework, you rarely create threads yourself. 
It is done for you by the framework.
So it is important to know how you can configure the framework you're using to benefit from the new concurrency features.
This is how you can use Virtual Threads with Spring.

---

<!-- .slide: data-background-color="#222" -->

## Developer (Jakarta EE)

Replace 

```java
@Resource(lookup = "java:app/concurrent/myExecutor")
private ManagedExecutorService managedExecutor;
```
by

```java
@ManagedExecutorDefinition(name = "java:app/concurrent/myExecutor", maxAsync = 3, virtual = true)
private ManagedExecutorService managedExecutor;
```
to get a `ManagedExecutorService` with virtual threads.

<https://blog.payara.fish/a-look-at-virtual-threads-in-a-jakarta-ee-managed-context> <!-- .element: class="attribution" -->

---

<!-- .slide: data-background-color="#222" -->

## Developer (Jakarta EE)

<pre><code class="java" data-trim data-line-numbers="1-6|1">
try (var scope = new StructuredTaskScope&lt;Object&gt;("MyTaskScopeWithContext", managedThreadFactory) {
    var subtask1 = scope.fork(task1);
    var subtask2 = scope.fork(task2);
    scope.join();
    // ...
}
</code></pre>

<a href="https://blog.payara.fish/a-look-at-virtual-threads-in-a-jakarta-ee-managed-context" class="attribution">https://blog.payara.fish/a-look-at-virtual-threads-in-a-jakarta-ee-managed-context</a>

note:

Structured Concurrency in Jakarta EE is supported by using the `StructuredTaskScope` overloaded constructor, which allows you to (slide) pass a *scope name* and a custom *thread factory*.
Which should be Jakarta EE's `ManagedThreadFactory` in this case.

---

<!-- .slide: data-background-color="#222" -->

## Developer (Quarkus)

<ul>
    <li class="fragment fade-in-then-semi-out">Use the <code>@RunOnVirtualThread</code> annotation to invoke the annotated method on a virtual thread.</li>
</ul>

<https://quarkus.io/blog/virtual-thread-1> <!-- .element: class="attribution" -->
