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
        (both mentioned in <a href="https://openjdk.org/jeps/525">https://openjdk.org/jeps/525</a>)
        </small>    
    </li>
</ul>

note:

- **sharing streams of data among threads** ('channels')

> It is not a goal to define a means of sharing streams of data among threads (i.e., [channels](https://en.wikipedia.org/wiki/Channel_(programming))). We might propose to do so in the future.

- **a new thread cancellation mechanism**

> It is not a goal to replace the existing thread interruption mechanism with a new thread cancellation mechanism. We might propose to do so in the future.

Both topics were mentioned in JEP 505 ('Structured Concurrency') as possible future additions to the language.
