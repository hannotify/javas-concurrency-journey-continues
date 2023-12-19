#### Java's Concurrency Journey Continues! Exploring
## Structured Concurrency 
#### and
## Scoped Values

<table>
    <tr>
        <td style="text-align: right; vertical-align: middle;" width="25%">Hanno Embregts</td>
        <td style="text-align: left; padding: 0 0 0 0; vertical-align: middle;"><img width="16%" data-src="img/logos/ace-associate-spade.png" class="no-background" style="margin-top: 30px; vertical-align: middle;"/><img width="22%" data-src="img/logos/java-champion.png" class="no-background" style="margin-top: 30px; vertical-align: middle;"/></td>
        <td style="text-align: right;"><img width="45%" data-src="img/icons/twitter-white.png" class="no-background" style="margin-top: 35px"/></td>
        <td style="vertical-align: middle; padding: 0 0 0 0"><a href="https://www.twitter.com/hannotify">@hannotify</a></td>
    </tr>
</table>
<img data-src="img/logos/java-community-logo.png" width="9%" class="no-background" style="margin-right: 2em">
<img data-src="img/logos/info-support.png" width="15%" class="no-background"/>
<br/>

note:
**Time Elapsed:** `0:00`.

*Preparations*

* Watch your pronounciation (British English is your _superpower_, so act like it)
* Set the appropriate browser zoom level (`80%` will fit all code onto the MacBook screen).
* Slides and IDE on projector screen
* Speaker notes on second screen
* Code in IntelliJ checked out to the right Git tag (`0-demo-start`)
* Terminal armed with `mirror` 
* Logitech Spotlight ready to go

*Intro*

* Hello, my name is Hanno, welcome to this talk!
* Today I want to talk about concurrency.
* I actually got the idea for this talk on a Friday night, while I was out for drinks with a few friends.

---

<!-- .slide: data-background="img/background/bar-with-drinks.jpg" data-background-color="black" data-background-opacity="1.0"-->

<https://www.pexels.com/photo/stylish-interior-of-bar-in-restaurant-5490965/> <!-- .element: class="attribution" -->

note: 

**Storytelling**

Ordering a drink with my Java Community coworkers, but no menu available.

...tell the story...

In Java 21, a few features were introduced that could very elegantly solve this particular situation.
