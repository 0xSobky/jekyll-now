---
layout: post
title: The Tricky Vine XSS (input sanitization EPICFAIL!)
---
_**TL;DR**: While doing some bug bounty hunting as usual, I ended up with a very cool reflected XSS vulnerability affecting <a href="https://vine.co">Vine</a>; here you can find the story behind it._

Vine is basically a short-form video sharing service acquired by Twitter and is part of Twitter's bug bounty program. So by heading to <a href="https://vine.co" target="_blank">vine.co</a>, you can notice that big search box over there on the top of the main page, right? That's exactly where our tricky XSS bug actually was!
<br />

Yet this one is not your typical search XSS; you may go and submit some XSS payload like <code class="code">&lt;img src=x onerror=alert(1)></code>, just to find out that a zillion other bug bounty hunters have tested this before on every single input source possible! So basically, if you don't come up with an outlandish idea here, you're mostly doomed to fail, and that's exactly what I did. But right before that, I tested out a few tricks beside tweaking the payload:

* Whilst being very unlikely and merely exploitable on unpatched legacy browsers like IE6/7/8, I did some testing and tweaks using UTF-7 payloads on IE, as perhaps that search page has got some issues with character encodings and its encoding isn't really fixated to UTF-8 ... but that didn't work, of course!

* Also tried to duplicate the search query---once with the payload and the other without---so we test out a parameter-pollution-like technique, but that just resulted in an error.
<br />

That being said, I then came up with the idea to try out a different HTTP method other than the default one (which is `POST`) to submit our XSS payload. And to my surprise, the magic eventually happened after forging a `GET` request containing this very XSS payload:
{% highlight html linenos %}
"><iframe/src=javascript:alert(document.cookie)>
{% endhighlight %}
Here's a screenshot:
<a href="/images/VineXSS.jpg" target="_blank"><img class="innerImg" src="/images/VineXSS-thumb.jpg" alt="XSS Screenshot"></a>

So, apparently, they were doing something like:
{% highlight PHP linenos %}
htmlspecialchars($_POST["query"]);
{% endhighlight %}
But no such thing as:
{% highlight PHP linenos %}
htmlspecialchars($_GET["query"]);
{% endhighlight %}
which results in filtering out our XSS payload when `POST`ed, but when sent via `GET`, you find no filter! And this particular unexpected behavior has managed to keep that issue totally hidden in plain sight from a bunch of very skillful hackers out there!



**Side Notes:**

[*] Just in case you are wondering, the bounty payout for this issue was 1400$.
<br />

**References:**

[*] <a href='https://hackerone.com/reports/39215' target='_blank'>https://hackerone.com/reports/39215</a>
