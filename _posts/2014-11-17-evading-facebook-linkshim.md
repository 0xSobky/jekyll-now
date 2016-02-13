---
layout: post
title: On Evading Facebook's Linkshim Mechanism
---
*TL;DR: Here I'll be talking about an interesting bypass for the so called «linkshim system», which Facebook mainly relies upon to protect its users from malicious URLs shared across the whole platform....*

In case you're not familiar with the term "Linkshim"---it's basically a mechanism that has been deployed since 2008, which whenever a user clicks an external link (from within Facebook), it checks that URL against an internal list of malicious domain names along with a bunch of other outsourced lists from McAfee, Google, Web of Trust (<a href="https://www.mywot.com/" target="_target">WoT</a>), et al.
<br />

That being said, and as any other blacklist-based protection, it always comes with its own pitfalls. So generally speaking, the typical way to break such mechanisms is by masking or obfuscating blacklisted data ... which would be "domain names" in this particular case. But how could we obfuscate a domain name?

Let's take this---innocuous?---domain name "[www.xx.com](#)" for instance, which used to be blacklisted. So, even if you've managed to get it posted or shared on Facebook, it'd get eventually detected by the linkshim as soon as a click had occured. This applies also to shortened URLs (e.g. goo.gl/xxx, is.gd/xxx and bit.ly/xxx), so that shortening such a blocked URL—no matter how many times—wouldn't make it pass through the linkshim as the destination URL is being checked too! So, how might we break that?

Well, before we get to the actual bypass, it's worth stating that easy tricks like "[facebook.com.xx.com](#)", "[www.facebook.com@xx.com](#)" or even "[www..xx..com](#)" didn't work ... it wasn't until I stumbled upon an interesting browser-dependent behavior, which is that on most webkit-based browsers—like Chrome, Opera and Safari—the whole URL is being URL-decoded right before attempting to resolve the domain name, which has turned out to be an unexpected sloppy-behavior for Facebook's linkshim. So by simply applying URL-encoding to a few characters of a blocked domain name and then shortening it, like this:
{% highlight bash linenos %}
http://www.xx.com >> 'http://www.xx.%63%6F%6D' >> http://goo.gl/0JeMLG
{% endhighlight %}
We would be able to bypass that very linkshim! And if our encoded domain name gets also blocked sometime later, we could simply re-encode it again but in a different form, as such:
{% highlight python linenos %}
http://%78%68%61%6D%73%74%65%72%2E%63%6F%6D
{% endhighlight %}
And it passes through, over and over again! 
<br />

***Side Notes:***<br />
[\*] Just in case you are wondering, the bounty payout for this issue was 2000$
<br />

***References:***<br />
[\*] <a href="https://www.facebook.com/notes/facebook-security/link-shim-protecting-the-people-who-use-facebook-from-malicious-urls/10150492832835766" target="_blank">https://www.facebook.com/notes/facebook-security/link-shim-protecting-the-people-who-use-facebook-from-malicious-urls/10150492832835766</a>
