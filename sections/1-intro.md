#### Java's Concurrency Journey Continues! Exploring
## Structured Concurrency 
#### and
## Scoped Values

<table>
    <tr>
        <td style="text-align: right; vertical-align: middle;" width="45.3%">Hanno Embregts</td>
        <td style="text-align: left; padding: 0 0 0 0; vertical-align: middle;"><img width="16%" data-src="img/logos/ace-associate-spade.png" class="no-background" style="margin-top: 30px; vertical-align: middle;"/><img width="22%" data-src="img/logos/java-champion.png" class="no-background" style="margin-top: 30px; vertical-align: middle;"/></td>
        <td style="text-align: right;"><img width="45%" data-src="img/icons/twitter-white.png" class="no-background" style="margin-top: 35px"/></td>
        <td style="vertical-align: middle; padding: 0 0 0 0"><a href="https://www.twitter.com/hannotify">@hannotify</a></td>
    </tr>
</table>
<img data-src="img/logos/java-community-logo.png" width="9%" class="no-background" style="margin-right: 2em">
<img data-src="img/logos/arnhemjug.png" width="30%" class="no-background"/>
<br/>

note:
**Time Elapsed:** `0:00`.

*Preparations*

* Watch your pronounciation (British English is your _superpower_, so act like it)
* Set the appropriate browser zoom level (80% will fit all code onto the MacBook screen).
* Slides and IDE on projector screen
* Speaker notes on second screen
* Code in IntelliJ checked out to the right Git tag (`TODO`)

*Intro*

* Hello, my name is Hanno, welcome to this talk!
* Today I want to talk about concurrency.
* A topic that has always interested me, but that has also *scared* me from time to time.
* Because of:
  * The fact that you typically don't need intricate concurrency knowledge in the workplace.
    * Unless you're a library developer
    * Unless you run into race conditions
    * Unless ... you name it.
    * So you typically don't need intricate concurrency knowledge, until you do.
  * Concurrency also scares me because of its ability to improve things, but also to make things *a lot* worse.
* Introducing concurrency can even make things *slower*.

---

<!-- .slide: data-background="img/background/the-illusion-of-multitasking.jpeg" data-background-color="black" data-background-opacity="1.0" data-background-size="contain" -->

note:

*Storytelling*

* Earlier this year, the Scrum Master of an agile team I was a part of hosted an exercise he called "The Illusion of Multitasking".
* With the purpose of making the team learn about focus span, disturbing factors and limit active work.
* One *worker* (me), five *managers* (them).
* Three scenarios:
  * All managers shouted letters at me and use everything in their power to get my attention. 
  * All managers formed a line and, when it was their turn, demanded the writing of a single letter on the sticky note that they bring.
  * All managers formed a line and, when it was their turn, spelled out the entire name they wanted me to write.
* Metrics:
  * Time-to-first (T 1e.)
  * Time-to-done (T tot.)
* Conclusions:
  * Almost nothing gets done when the manager:worker ratio is 5:1. :)
  * A lot of time can be lost on context switching (see scenario 2).
  * When only a single task can be executed at the same time, then concurrency will *definitely* make things slower.
* Fortunately Java's concurrency features are a lot more sophisticated!
