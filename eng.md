##Implementing Off-Canvas Navigation For A Responsive Website

The varying viewports that our websites encounter on a daily basis continue to demand more from responsive design. Not only must we continue to tackle the issues of [content choreography][1] — the art of maintaining order and context throughout the chaotic ebb and flow of the Web browser — but we must also **meet the expectations of users**. They’re not sitting still.

With the likes of [Firefox OS][2] (Boot to Gecko), [Chrome OS][3] and now [Ubuntu for phones][4] — an OS that makes “Web apps” first-class citizens — delivering native app-like experiences on the Web may become a necessity if users begin to expect it. Many in our field have argued for a degree of separation between the Web and native platforms for both technical and philosophical reasons. They’re certainly wise to heed caution, but as consumer devices continue to blur the boundaries, it’s worth thinking about what we can learn from native app design.

###A Demonstration

In this article, I’ll be walking through a build demo that centers on two topics. The first is [responsive design patterns][5] that embrace the viewport and that improve content discoverability beyond the basic hyperlink; in this case, **off-canvas navigation**. The second is the complexities of implementing such ideas in an accessible and highly performant manner. These are two topics that I believe are at the heart of the Web’s future.

With that in mind, let’s get building.

###The Accessible Base

All good things begin with a solid foundation of semantic HTML and widely supported CSS. In theory, this baseline should function as a usable experience for all browsers that visit our website. (It might also be the *final* experience in less-capable browsers.)

As a starting point, I’ll use a technique very similar to Aaron Gustafson’s “[Smart Mobile Navigation Without Hacks][6].” It requires no JavaScript to function.

![Responsive Off-Canvas Menu Demo 1](img/demo-1-500.png?raw=true&repo=off-canvas-navigation-for-responsive-website "Demo 1")

*Step 1 of the responsive off-canvas menu. Short link: [bit.ly/offcanvas1][7]*

- [View demo 1][8]<br>
Be sure to view on a mobile or small screen, and take a while to inspect the code. Although our final design will be significantly different, starting simple is vital; retrofitting accessibility isn’t trivial.

The HTML body looks like this (I’ve stripped a few attributes for semantic clarity):

~~~~ .language-html
<header id="top" role="banner">
    <h1>Book Title</h1>
    <a href="#nav">Book navigation</a>
</header>
<nav id="nav" role="navigation">
    <h2>Chapters</h2>
    <ul>
        <li><a href="#">Chapter 1</a></li>
        <li><a href="#">Chapter 2</a></li>
        <li><a href="#">Chapter 3</a></li>
        <li><a href="#">Chapter 4</a></li>
        <li><a href="#">Chapter 5</a></li>
    </ul>
    <a href="#top">Return to content</a>
</nav>
<article role="main">
    <!-- [main content here] -->
</article>
~~~~

You could consider the HTML alone, with little to no styling, as being “breakpoint zero.” If it’s not logical at this stage, then accessibility will not improve.

####DEMO 1 BREAKDOWN

- Media queries are based on a viewport width of `45em` (that’s content-dependent). Above this breakpoint, the navigation is permanently visible. I prefer em units because they allow breakpoints to maintain a relationship with text size. Lyza Gardner explains in detail in her post “[The EMs Have It: Proportional Media Queries FTW!][9]”
- I’m using both `min-width` and `max-width` media queries to scope CSS. This adds a bit of complexity. Most people prefer a “mobile-first” build, using only progressively larger `min-width` queries. The downside with that technique is the amount of **resetting** required if an element has noticeably different visual states. Neither method is right or wrong.
- The crux of this initial stage is the [:target pseudo-class][10] selector, utilized to show and hide the navigation. Only IE8 and lower lack support. However, this is a non-issue if you serve a semi-fluid desktop style sheet to old IEs. [Jake Archibald][11], [Nicolas Gallagher][12] and [Stuart Robson][13] can tell you more.

As the demo takes shape, I’ll continue to introduce the main development principles. There’s a long way to go yet…

###Going Off-Canvas

For some websites, the above may suffice — but not for us! We’re experimenting with off-canvas patterns and striving for that native experience. Because we cannot ignore older browsers, it’s now time to **progressively enhance**.

![Responsive Off-Canvas Menu Demo 2](img/demo-2.png?raw=true&repo=off-canvas-navigation-for-responsive-website "Demo 2")

*Step 2 of the responsive off-canvas menu. Short link: [bit.ly/offcanvas2][14]*

- [View demo 2][15]<br>
You will see the restyled navigation with basic functionality.

####DEMO 2 BREAKDOWN

- I’m adding the class `js-ready` to the document element after the [DOMContentLoaded][16] event fires. The selector `.js-ready` is used as a hook to safely restyle the navigation off-canvas. If for whatever reason JavaScript *doesn’t* load, then the original functionality from demo 1 still exists.
- To show and hide the navigation, I’m toggling a class of `js-nav` on the document element when the user clicks (or taps) the relevant buttons. This simply applies a style of `left: 70%` to the `#inner-wrap` element (`#outer-wrap` is used to hide any overflow and to avoid scrollbars).

This is a fairly basic enhancement, but importantly it remains usable before JavaScript is ready. It’s also notable that no inline styles are written with JavaScript; only classes are used to manage states.

Jumping between open and closed navigation states makes for a jarring user experience. Users need to understand — or even see — how an interface has changed. This is often the point where developers let the Web down. To be fair, building user interfaces is incredibly difficult. What I’m going to show below is far from perfect, but it’s certainly a step in the right direction.

So, we have the set-up. Now let’s add transitions.

###Transitioning (The Wrong Way)

I’ll start by getting it all wrong, because this is how I would have done it a few years ago, and learning from mistakes is important.

![Responsive Off-Canvas Menu Demo 3 (jQuery)](img/demo-jquery.png?raw=true&repo=off-canvas-navigation-for-responsive-website "Demo jQuery")

*The responsive off-canvas menu using jQuery .animate for transitions. Short link: [bit.ly/offcanvas3][17]*

[jQuery][18] is a resource that many front-end developers begin with to learn JavaScript. Personally, I am a big fan (never blame the tools), but unfortunately jQuery has a habit of making things look deceptively simple. It masks complexity and, with that, *understanding*.

Transitioning our off-canvas navigation with jQuery is very easy:

~~~~ .language-javascript
$('#nav-open-btn').on('click', function() {
    $('#inner-wrap').animate({ left: '70%' }, 500);
});

$('#nav-close-btn').on('click', function() {
    $('#inner-wrap').animate({ left: '0' }, 500);
});
~~~~

Visually, this achieves the effect we’re after, but test it on mobile and watch the frame rate stutter. Performance is dreadful.

This method is bad for several reasons:

![An animation of jQuery updating the DOM](img/jquery-animate.gif?raw=true&repo=off-canvas-navigation-for-responsive-website)

- jQuery’s `.animate` increments the element’s style attribute on every animation frame (as can be seen in the GIF above). This forces the browser to [recalculate the layout][19].
- It leaves inline styles in the DOM that have very **high specificity** and that will override our well-maintained CSS. This is a big issue if the viewport is resized and triggers different breakpoints.
- A **separation of concerns** is lost because styles are defined in JavaScript files.

Overall, it’s a performance and maintainability nightmare. There is a better way.

###Transitioning With CSS

Putting the jQuery experiment aside, I’m now building from the second demo again, this time using **CSS transforms and transitions**. These enable us to smoothly animate the off-canvas navigation with great performance.

![Responsive Off-Canvas Menu Demo 4](img/demo-final1.png?raw=true&repo=off-canvas-navigation-for-responsive-website)

*The final responsive off-canvas menu using CSS transforms and transitions. Short link: [bit.ly/offcanvas4][20]*

- [View final demo][21]<br>
Performance has drastically improved compared to the jQuery example.

####FINAL DEMO BREAKDOWN

- Returning to CSS, I’m once again using the `.js-nav` class to toggle the navigation, while making no actual style alterations with JavaScript.
- I’m progressively enhancing with the classes `.csstransforms3d` and `.csstransitions` (applied to the document element by [Modernizr][22]).
- Instead of moving the navigation with negative positioning (`left: -100%`), I’m using the `transform` property: `transform: translate3d(-100%, 0, 0)`.
- CSS transitions are used to animate `transform` changes: `transition: transform 500ms ease`. And I’ve added a few more transforms to enhance the visual effect.

With the help of Modernizr, this will fall back to [demo 2][23] if browser support is lacking for CSS transforms and transitions. (In theory, I could fall back to jQuery animation, but it’s really not worth it.) When you [download Modernizr][24], you can include only the feature detection that you need. It also includes the HTML5 shiv for IE. All in all, it’s a very useful script.

The benefits here are immense. First and foremost, by transitioning elements with 3-D transforms, the browser can generate render layers that are [hardware-accelerated][25]. Secondly, there’s no need for JavaScript to worry about style or media-query breakpoints. JavaScript need only interpret user interaction and apply classes to maintain states — CSS defines the visual changes.

We can look deeper into performance by using Chrome’s Developer Tools. The results below are from the desktop browser, but for mobile performance you could use [USB Remote Debugging][26] on Android devices.

####JQUERY ANIMATION PERFORMANCE

![jQuery animation performance in Chrome Developer Tools](img/demo-perf-jquery.png?raw=true&repo=off-canvas-navigation-for-responsive-website)

The four events above represent the opening and closing of the navigation twice. In the diagram, yellow represents the JavaScript running, purple is the rendering (recalculating the style and layout), and green is the painting to the screen. On mobile, we’d be shooting way below that 30 FPS line. I also tested on an iPhone 3GS and could literally count 3 FPS with my own eyes — it’s that slow!

####CSS TRANSITION PERFORMANCE

![CSS transition performance in Chrome Developer Tools](img/demo-perf-css.png?raw=true&repo=off-canvas-navigation-for-responsive-website)

What a difference! The only JavaScript that exists is there to manage user interaction before and after the transition. The green we’re seeing is the minimum that the browser needs to repaint the transition at a respectable frame rate, using [GPU acceleration][27].

####POSSIBLE CONCERNS

As with all new Web standards, nothing is inherently perfect.

In WebKit-based browsers, the [font smoothing][28] may switch to antialiased from the default subpixel-antialiased when CSS transforms or transitions are applied. This could result in visually thinner text, which many designers actually prefer — but [not something you should “fix.”][29]

Flickering can also occur if an element goes between transform and no-transform. To avoid this, always start with a default like `translate3d(0,0,0)` on an element that will move later, so that the render layer is composed and ready.

###The Next Step

Enhancing further, we could detect and take advantage of touch gestures, like swiping, to really bring this implementation closer to par with its native counterparts — “closer” being the operative word, but I do believe the gap is closer than many would have us believe.

It’s also — in my opinion — a gap that we *need* to bridge.

There are other clever ways to increase interaction speed. Mobile browsers tend to wait around 300 ms to fire click events. Google’s Ryan Fioravanti has a great article on “[Creating Fast Buttons][30]” to reduce this latency.

[Transition easing][31] can radically change the visual effect and the user’s perception of what’s happening. Whether elements need to spring, bounce or accelerate into action, Lea Verou has a useful [cubic bezier][32] resource to generate custom easing functions.

Hopefully, this article has shown the improvements that can be made if we take extra care to understand the technology we’re using.

####DOWNLOAD THE CODE

So, there we have it: a three-tiered responsive, interactive, off-canvas menu. I’ve set up a [GitHub repository][33] where you can view all the code. Have a play with it; there are a few bits and pieces I haven’t covered to avoid bloating this article.

####FURTHER READING

To really understand why the techniques highlighted above are my preferred solution, I point you in the direction of these resources:

- “[All You Need to Know About CSS Transitions][34],” Alex MacCaw
MacCaw gets into the specifics of CSS transitions and JavaScript.
- “[Why Moving Elements With translate() Is Better Than pos:abs top/left][35],” Paul Irish
A superb video and article examining the different movement techniques.
- “[Improving the Performance of Your HTML5 App][36],” Malte Ubl
Looks deep into the intricacies of performance.
- “[CSS3 vs jQuery Animations][37],” Siddharth Rao
This article puts both methods head to head.
- “[Understanding Hardware Acceleration on Mobile Browsers][38],” Ariya Hidayat
Goes under the hood with a technical review.
- “[Let’s Play With Hardware-Accelerated CSS][39],” Martin Kool
This article also introduces us to hardware acceleration.
- “[Scrolling Performance][40],” Paul Lewis
Looks at a related scenario.

[1]: http://trentwalton.com/2011/07/14/content-choreography/
[2]: http://www.mozilla.org/en-US/firefoxos/
[3]: http://www.google.com/intl/en/chrome/devices/
[4]: http://www.ubuntu.com/devices/phone
[5]: http://bradfrost.github.com/this-is-responsive/patterns.html
[6]: http://www.netmagazine.com/tutorials/build-smart-mobile-navigation-without-hacks
[7]: bit.ly/offcanvas1
[8]: http://dbushell.github.com/Responsive-Off-Canvas-Menu/step1.html
[9]: http://blog.cloudfour.com/the-ems-have-it-proportional-media-queries-ftw/
[10]: https://developer.mozilla.org/en-US/docs/Using_the_:target_selector
[11]: http://jakearchibald.github.com/sass-ie/
[12]: http://nicolasgallagher.com/mobile-first-css-sass-and-ie/
[13]: http://www.alwaystwisted.com/post.php?s=2012-08-06-a-sass-mixin-for-media-queries-and-ie
[14]: bit.ly/offcanvas2
[15]: http://dbushell.github.com/Responsive-Off-Canvas-Menu/step2.html
[16]: https://developer.mozilla.org/en-US/docs/Mozilla_event_reference/DOMContentLoaded_(event)
[17]: bit.ly/offcanvas3
[18]: http://jquery.com/
[19]: https://developers.google.com/speed/articles/reflow
[20]: bit.ly/offcanvas4
[21]: http://dbushell.github.com/Responsive-Off-Canvas-Menu/step4.html
[22]: http://modernizr.com/
[23]: http://dbushell.github.com/Responsive-Off-Canvas-Menu/step2.html
[24]: http://modernizr.com/download/
[25]: http://mobile.smashingmagazine.com/2012/06/21/play-with-hardware-accelerated-css/
[26]: https://developers.google.com/chrome/mobile/docs/debugging
[27]: http://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome
[28]: http://www.usabilitypost.com/2012/11/05/stop-fixing-font-smoothing/
[29]: http://www.usabilitypost.com/2012/11/05/stop-fixing-font-smoothing/
[30]: https://developers.google.com/mobile/articles/fast_buttons
[31]: http://www.w3.org/TR/css3-transitions/#transition-timing-function-property
[32]: http://cubic-bezier.com/
[33]: https://github.com/dbushell/Responsive-Off-Canvas-Menu
[34]: http://blog.alexmaccaw.com/css-transitions
[35]: http://paulirish.com/2012/why-moving-elements-with-translate-is-better-than-posabs-topleft/
[36]: http://coding.smashingmagazine.com/2013/01/15/off-canvas-navigation-for-responsive-website/Improving%20the%20Performance%20of%20your%20HTML5%20App
[37]: http://dev.opera.com/articles/view/css3-vs-jquery-animations/
[38]: http://www.sencha.com/blog/understanding-hardware-acceleration-on-mobile-browsers/
[39]: http://mobile.smashingmagazine.com/2012/06/21/play-with-hardware-accelerated-css/
[40]: http://coding.smashingmagazine.com/2013/01/15/off-canvas-navigation-for-responsive-website/Scrolling%20Performance
