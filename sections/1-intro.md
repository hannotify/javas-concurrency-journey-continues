#### Java's Concurrency Journey Continues! Exploring
## Structured Concurrency 
#### and
## Scoped Values

<br/>
<table>
    <tr>
        <td style="text-align: right; vertical-align: middle;" width="36%">Hanno Embregts</td>
        <td style="text-align: left; padding: 0 0 0 0; vertical-align: middle;">
            <img width="16%" data-src="img/logos/ace-pro-spade.png" class="no-background" style="margin-top: 30px; vertical-align: middle;"/>
            <img width="20%" data-src="img/logos/java-champion.png" class="no-background" style="margin-top: 30px; vertical-align: middle;"/>
            <img width="15%" data-src="img/logos/nljug.png" class="no-background" style="margin-top: 30px; vertical-align: middle;"/>
        </td>
        <td style="vertical-align: middle; text-align: right;"><img width="35%" data-src="img/icons/bluesky.png" class="no-background" style="margin-top: 35px"/></td>
        <td style="vertical-align: middle; padding: 0 0 0 0"><a href="https://bsky.app/profile/hanno.codes">@hanno.codes</a></td>
    </tr>
</table>
<br/>
<img data-src="img/logos/jfokus-2026.svg" width="40%" class="no-background"/>


note:
**Time Elapsed:** `0:00`.

*Preparations*

* append `?controls=false` to the URL
* Watch your pronunciation
* Set the appropriate browser zoom level (`80%` will fit all code onto the MacBook screen).
* Slides and IDE on projector screen
* Speaker notes on second screen
* Dark mode activated in IDE
* Code in IDE checked out to the right Git tag (`0-demo-start`)
* Cmd+F1 to switch from mirror to extend
* Logitech Spotlight ready to go

*Intro*

* Hello, my name is Hanno, welcome to this talk!
* Today I want to talk about Java's concurrency features.
* How it all started in the 90s, how Java's support for concurrency improved over time.
* And what is currently happening in Java's latest releases.
* It's a topic that has always interested me, but that has also *scared* me from time to time.
* Because of:
  * The fact that you typically don't need intricate concurrency knowledge in the workplace.
    * Unless you're a library developer
    * Unless you run into race conditions
    * Unless ... you name it.
    * So you typically don't need intricate concurrency knowledge, until you do.
* So, concurrency sometime scares me. Not because it can improve things. Because it can make things *a lot* worse.
* Introducing concurrency can even make things *slower*.

---

<!-- .slide: data-background="img/background/the-illusion-of-multitasking.jpeg" data-background-color="black" data-background-opacity="1.0" data-background-size="contain" -->

note:

*Storytelling*

> A workshop that leads to a discussion about focus, focus span, disturbing factors, limit active work.

* Earlier this year, the Scrum Master of an agile team I was a part of hosted an exercise he called "The Illusion of Multitasking".
* With the purpose of making the team learn about focus span, disturbing factors and why you should limit active work.
* One *worker* (me), armed with a Sharpie, and five *managers* (them), armed with sticky notes.
* This exercise had three scenarios:
  * All managers shouted letters at me and use everything in their power to get my attention to write letters on sticky notes.
  * All managers formed a line and, when it was their turn, demanded the writing of a single letter on the sticky note that they brought.
  * All managers formed a line and, when it was their turn, spelled out the entire name they wanted me to write.
* Metrics:
  * Time-to-first (T 1e.)
  * Time-to-done (T tot.)
* Conclusions:
  * Almost nothing gets done when the manager:worker ratio is 5:1. :-)
  * A lot of time can be lost on context switching (see scenario 2).
  * When only a single task can be executed at the same time, then concurrency will *definitely* make things slower.
  * Concurrency can have both positive and negative effects.

---

<!-- .slide: data-background="img/background/bar-with-drinks.jpg" data-background-color="black" data-background-opacity="1.0"-->

<https://www.pexels.com/photo/stylish-interior-of-bar-in-restaurant-5490965/> <!-- .element: class="attribution" -->

note: 
* I actually got the idea for this talk on a Friday night, while I was out for drinks with a few friends.

**Storytelling**

Ordering a drink with my Java Community coworkers, but no menu available.

...tell the story...

In recent Java versions, a few features were introduced that could very elegantly solve this particular situation.
