<!-- .slide: data-background="img/background/destination-arrows.jpg" data-background-color="black" data-background-opacity="0.3"-->

# Have We Come To Journey's End?  <!-- .element: class="stroke" -->

<https://www.pexels.com/photo/sign-arrow-direction-travel-52526/> <!-- .element: class="attribution" -->

note:

**Time Elapsed:** `42:00`.

So is Java's concurrency journey over now?
Have we come to journey's end?
What new features are expected to be developed after these?

---

## Possible Future Features

- sharing streams of data among threads ('channels') <!-- .element: class="fragment fade-in-then-semi-out" -->
- a new thread cancellation mechanism <!-- .element: class="fragment fade-in-then-semi-out" -->
- solving thread pinning when virtual threads are executing a synchronized code block <!-- .element: class="fragment fade-in-then-semi-out" -->
- TODO: coroutines?
- TODO: unfinished stuff from Project Loom?
- TODO: even more?

note:

- **sharing streams of data among threads** ('channels')

> It is not a goal to define a means of sharing streams of data among threads (i.e., [channels](https://en.wikipedia.org/wiki/Channel_(programming) "‌")). We might propose to do so in the future.

- **a new thread cancellation mechanism**

> It is not a goal to replace the existing thread interruption mechanism with a new thread cancellation mechanism. We might propose to do so in the future.

- **solving thread pinning when virtual threads are executing a synchronized code block**

> Virtual threads not releasing the underlying operating system thread when working on a synchronized method 
> is a limitation in JDK 19. It could be addressed in a future release of Java.


- what alternative features do languages like Go, C# and Kotlin offer, that might inspire Java language designers?
  - in what ways are coroutines and virtual threads / structured concurrency different & similar?
  - see: https://auroratide.com/posts/understanding-kotlin-coroutines
- TODO: even more?

