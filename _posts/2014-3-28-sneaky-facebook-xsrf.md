---
layout: post
title: The Sneaky Facebook XSRF/CSRF!
---
_**TL;DR**: A CSRF vulnerability that could reset a Facebook user's post-by-email address was hidden deep inside the Facebook mobile site, where you have to first trigger some kind of legacy browser fallback support and then to tweak with some parameter(s) to catch it!_

As you know, Facebook (and many other web services) does provide a post-by-email address that you can use to upload photos, videos or to post status updates to your account right from your email easily. That address is particularly useful in cases in which direct access to your Facebook account is not possible at the time....

So, while I was messing around on (<a href="https://m.facebook.com" target="_blank">m.facebook.com</a>), specifically with this image uploading endpoint (<a href="https://m.facebook.com/photos/upload/" target="">https://m.facebook.com/photos/upload/</a>), I forwarded a malformed form submission to that endpoint where some `POST` key/value pairs were omitted and some others were duplicated and tampered with, just to see how Facebook would handle such `POST` request ... and to my surprise, Facebook responded with a 302 HTTP status code pointing to:
{% highlight css linenos %}
Location: https://m.facebook.com/photos/upload/?email_fallback
{% endhighlight %}
Where an error message was suggesting me to upload my photos using my post-by-email address instead, as my browser doesn't seem to be supporting posting photos directly! Here's how that page looked like at that time:
<br>
<a href="/images/FB-XSRF.jpg" target="_blank"><img class="innerImg" src="/images/FBXSRF-thumb.jpg" alt="fallback"></a>
And as you can see from the screenshot above, there's a "Reset Email Address" button which is when clicked, it submits a `POST` request containing an anti-CSRF token named "__fb\_dtsg__" and a parameter named "__reset\_email__" assigned the value of "__1__"; the obvious hunt here is to omit that "fb\_dtsg" parameter—or to alter its value—and then forward the request, right? 

Unfortunately, that didn't work and the token was being properly validated ... not until this idea popped on my mind "What if I submit that very parameter using a `GET` request instead of `POST`?", so I eventually forged this simple `GET` request:
{% highlight css linenos %}
GET https://m.facebook.com/photos/upload/?email_fallback&reset_email HTTP/1.1
{% endhighlight %}
And guess what? My post-by-email address got simply reset, thanks to 12 chars added to a `GET` request!
