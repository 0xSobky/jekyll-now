---
layout: post
title: Abusing Facebook Social Plugins (for token leakage)
---
_**TL;DR:** By abusing Facebook social plugins like the activity feed plugin and/or the recommendations plugin, an attacker could retrieve valid sensitive tokens (e.g. access/m_sess tokens), unwittingly shared publicly across the Facebook platform...._

Facebook does provide a variety of social plugins that web authors can deploy on third-party websites, but interestingly, a couple of these plugins were leaking sensitive tokens, including but not limited to, valid access tokens and mobile sessions (a.k.a m\_sess tokens).
<br />

Both of the two aforementioned social plugins (i.e. the activity feed plugin and the recommendations plugin) returned similarly interesting results/responses when embedded with certain content filters such as the "__connect__" filter. For instance, a `GET` request as follows:
{% highlight python linenos %}
GET https://www.facebook.com/plugins/activity.php?site=facebook.com&width=900&height=60000&header=true&skin=dark&filter=connect HTTP/1.1
{% endhighlight %}
used to return an HTML document containing dozens of access\_token values appended to the hash fragment of so many URLs in this form:
{% highlight python linenos %}
https://www.facebook.com/connect/login_success.html#access_token=AAWWxSSFWcsdf...krfrgdvDFfFF 
{% endhighlight %}

As you might have expected, these tokens were actually valid tokens originally issued for various apps including official ones like "Facebook for Blackberry", "Facebook for Android", etc. Each of these tokens was granted extended permissions with offline\_access too, where an attacker can easily exploit them to access private users' data and even turning the owner accounts into zombie ones for social spam campaigns!
<br />

Another interesting type of tokens—that was being leaked too—is the so called "m\_sess" token, which has been typically used for authentication all over the Facebook mobile site (m.facebook.com), and frankly, an attacker could have easily done something like: 
{% highlight python linenos %}
GET https://m.facebook.com/messages/thread/[user_id]/?m_sess=c2VzczoxMDAw...BOjI6MTxNjQyOQ HTTP/1.1
{% endhighlight %}
using a valid 'm\_sess' token, and voila, he's directly into some Facebook user's inbox! all you need to do to retrieve such tokens is use this content filter "thread/message", forwarding a `GET` request as follows:
{% highlight python linenos %}
GET https://www.facebook.com/plugins/recommendations.php?site=m.facebook.com&width=9003&height=10000&skin=dark&header=true&filter=thread/messages HTTP/1.1
{% endhighlight %}

Such request typically used to return an HTML document containing some juicy "m\_sess" tokens, though just a few of them were yet to get expired due to the short expiry date issued to such type of tokens....  
<br />
That being said, the root cause of this issue wasn't really obvious at first as such social plugins clearly have nothing to do with authentication tokens, so the best guess was that it's a result of some kind of a user error in the first place; as suggested in this follow-up email:

> Hi Ahmed,<br />
> 
> This might be related to an earlier issue that partially results from a user error.
> We might do a short term block for this or go straight to the root cause.
> 
> Thanks,<br />
> [redacted]<br />
> Security<br />
> Facebook

Unsurprisingly, it did turn out that a portion of Facebook users were unwittingly sharing their tokens in public posts all over the platform, unwary of how sensitive these tokens actually are! ...quite disappointing, isn't it?!
