---
layout: post
title: Messing Around with Web Browsers (Part I)
---
*TL;DR: Some interesting URL Spoofing attacks, some functionality bugs and a neat exploit to blow your favourite browser up ... just got excited after reading "The Browser Hacker's Handbook"!*

First things first, If you haven't read "<a href = "https://browserhacker.com" target="_target">The Browser Hacker's Handbook</a>" yet or haven't even heard about it before, you should definitely go and give it a read! Seriously, if "The Web Application Hacker's Handbook" is said to be the bible of web hackers, then this is to be---with all due respect---Jesus of web hackers!
<br />

Now, let's say you have an endpoint on a target origin domain name that typically returns a "204" HTTP status code like this one "<https://www.google.com/csi>" on Google's main domain, How can we make use of such endpoint to conduct a URL spoofing attack against that same domain? 

Well, it all depends on how a browser handles such 204 'no content' HTTP response. Luckily, almost all browsers from the webkit browser family (i.e. Chrome/Chromium, Opera and Safari) handle such response in an exploitable manner whenever a user navigates to an endpoint similar to the one mentioned before, as none of them resets the URL nor the DOM of the current document when a "204" HTTP status code is returned. So, here's a simple exploit code example:
{% highlight javascript linenos %}
window.onbeforeunload = function() {
    var currentDomain = document.domain ? document.domain : 'null';
    var HTML = '<html>' + 'Spoofed Content!' + '<br>' + 'Current Domain: ' + currentDomain + '</html>';
    setTimeout(function(){document.write(HTML)}, 1000);
};
{% endhighlight %}
To better understand what goes wrong, you may take a look at this short video: <br />
<iframe class="frames" width="420" height="315" src="https://www.youtube.com/embed/R6XlWHzzFpI" frameborder="0" allowfullscreen="true"></iframe><br />
Yet after some googling around, it turned out that this issue is actually a variant of <a href="https://code.google.com/p/chromium/issues/list?can=1&q=label%3ACVE-2013-2916" target="_blank">CVE-2013-2916</a>. Though, the exploitation in our case here requires some sort of social engineering as it only works against browser-initiated navigations (i.e. the victim has to initiate the navigation manually)

Furthermore, another interesting URL-spoofing issue I've encountered during some semi-manual fuzzing with super long URLs—particularly with RTL URLs—on both of Firefox and Chrome is a one that occurs due to a bug in the scrolling function of the browser's navigation bar, so if it happens that you are such a RTL language user and subsequently you adjust the alignment of your URL bar to right from time to time, then you are prone to such spoof:<br />
*Firefox example:*
<a href="/images/FirefoxSpoof.jpg" target="_blank"><img class="innerImg" src="/images/FirefoxSpoof-thumb.jpg" alt="Firefox Spoof"></a>
*Chrome example:*
<a href="/images/ChromeSpoof.jpg" target="_blank"><img class="innerImg" src="/images/ChromeSpoof-thumb.jpg" alt="Chrome Spoof"></a>

And as obvious as it seems, neither of Firefox nor Chrome handle such a long RTL-aligned URL properly, resulting in getting the real origin shifted away into the right corner of the navigation bar! Here's a simple PoC code:
{% highlight javascript linenos %}
(function {
    location.href = (/firefox/i.test(navigator.userAgent)) ? 'http://www.example.com/https://www.mozilla.org/XxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxXXXXXxxxxxxxxxxxxxxxxXXXxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxXxxxxxxxxXxxxxxUUxxxxxxxxxxxxx' : 'http://example.com/dd/%D8%A7%D8%A7%D8%A7zzzzzz%D8%A7%D8%A7%D8%A7%D8%A7%D8%A7Spoofedzzz%D8%A7zxzlllzzzzxzzzllSpoofedSpoofedSpoofHASHSpoofedspoofzzz%D8%A7%D8%A7%D8%A7%D8%A7zzz%D8%A7%D8%A7%D8%A7Hashlz%D8%A7%D8%A7Spoofed/Spoofed.php?query=xxHASHspoofzz%D8%A7l%D8%A7/https://www.google.com';
}());
{% endhighlight %}
Yet this is fairly hard to exploit in the general case, as it's a prerequisite for the alignment of the navigation bar to be adjusted to right which is not such a common practice.
<br />

Well, I'd say enough for now but just a last---but not least---little issue to mention:
Firefox seems to have a very interesting behaviour when it comes to navigating from one webpage to another. Typically, it doesn't reset the contents of the current webpage until a feasible part of the requested webpage is fully loaded, then it starts rendering the new webpage. This doesn't seem to be a secure behaviour, as an attacker can simply take advantage of such behavior to emulately spoof the URL with the help of a MITM attack. Just like what happens in this quick demo: <a href="https://bugzilla.mozilla.org/attachment.cgi?id=8387576" target="_blank">https://bugzilla.mozilla.org/attachment.cgi?id=8387576</a>
<br />

**Update:** You can now read part two here:  [https://0xsobky.github.io/messing-with-browsers2](https://0xsobky.github.io/messing-with-browsers2).

***Notes:***

[\*] As Opera was the first browser to get patched from that '204 No Content'-based spoof, I got credited here:* <a href="http://blogs.opera.com/security/2014/08/security-changes-opera-23/" target="_blank">http://blogs.opera.com/security/2014/08/security-changes-opera-23/</a>

***References:***

[\*] <a href="https://code.google.com/p/chromium/issues/detail?id=361434" target="_blank">https://code.google.com/p/chromium/issues/detail?id=361434</a><br />
[\*] <a href="https://bugzilla.mozilla.org/show_bug.cgi?id=991503" target="_blank">https://bugzilla.mozilla.org/show_bug.cgi?id=991503</a>
