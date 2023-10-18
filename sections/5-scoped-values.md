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

<!-- .slide: data-background="img/background/binary-code.jpg" data-background-color="black" data-background-opacity="0.3" -->

## Demo, Part 3

- Let's migrate `AnnouncementId` from a Thread Local to a Scoped Value

<https://pxhere.com/en/photo/1458897> <!-- .element: class="attribution" -->

note:
[TODO] Rehearse demo and sets appropriate Git tags.

Let's see scoped values in action!

- Let's migrate `AnnouncementId` from a Thread Local to a Scoped Value
- TODO: explain some stuff

---

## Wait A Minute!

TODO (1-x times)

---

## To Summarize

<ul>
    <li class="fragment fade-in-then-semi-out"><em>Thread-locals</em> might perform badly when many (virtual) threads are involved;</li>
    <li class="fragment fade-in-then-semi-out"><em>Scoped values</em> TODO.</li>
</ul>

notes: 

In summary, TODO
