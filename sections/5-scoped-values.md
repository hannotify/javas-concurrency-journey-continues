<!-- .slide: data-background="img/background/upcoming-station.jpg" data-background-color="black" data-background-opacity="0.5"-->

# Scoped Values  <!-- .element: class="stroke" -->

<https://www.pexels.com/photo/photography-of-railway-894312/> <!-- .element: class="attribution" -->

note: 

**Time Elapsed:** `32:00`.

Upcoming Station: Scoped Values

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
Note that you need to pass the compiler option `--enable-preview` to be able to work with it.

---

## What Problem Are We Trying To Solve?

note:

* But let's take a step back first..
* Let's remind ourselves of the problem that we're trying to solve here.

(slide)

TODO

---

<!-- .slide: data-background="img/background/spider-web.jpg" data-background-color="black" data-background-opacity="0.3"-->

## Scoped Values <!-- .element: class="stroke" -->

<blockquote class="explanation">
TODO
</blockquote>

<https://www.pexels.com/photo/spider-web-in-closeup-photo-1038278/> <!-- .element: class="attribution" -->

note:
(first read the definition out loud)
> TODO

---

## Scoped Values

TODO:

* a <!-- .element: class="fragment fade-in-then-semi-out" -->
* few <!-- .element: class="fragment fade-in-then-semi-out" -->
* characteristics <!-- .element: class="fragment fade-in-then-semi-out" -->

note:

TODO

---

## Useful for:

- hidden method arguments
- re-entrant code
- nested transactions
- graphics contexts

> In general, we advise migration to scoped values when the purpose of a thread-local variable aligns with the goal of a scoped value: one-way transmission of unchanging data.
>
> Thread-local variables retain their values until explicitly removed using the "remove" method or until the thread terminates. Developers often forget to remove them, causing data to persist longer than necessary. This can be a security risk, especially in thread pools where data may unintentionally transfer between tasks. For programs relying on mutable thread-local variables, finding a safe point to call "remove" can be challenging, potentially causing long-term memory leaks. It is preferable to limit data access to a specific execution period to prevent these issues.

---

<!-- .slide: data-background="img/background/binary-code.jpg" data-background-color="black" data-background-opacity="0.3" -->

## Demo, Part 3

- Let's migrate `AnnouncementId` from a thread-local to a scoped value

<https://pxhere.com/en/photo/1458897> <!-- .element: class="attribution" -->

note:
[TODO] Rehearse demo and sets appropriate Git tags.

Let's see scoped values in action!

- Let's migrate `AnnouncementId` from a thread-local to a scoped value
- TODO: explain some stuff

---

## Wait A Minute!

- How To Use In Conjunction With Structured Concurrency?
TODO (1-x times)

---

## To Summarize

<ul>
    <li class="fragment fade-in-then-semi-out"><em>Thread-locals</em> might perform badly when many (virtual) threads are involved;</li>
    <li class="fragment fade-in-then-semi-out"><em>Scoped values</em> TODO.</li>
</ul>

notes: 

In summary, TODO
