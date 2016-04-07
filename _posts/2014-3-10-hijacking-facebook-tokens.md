---
layout: post
title: Hijacking Facebook Apps' Access Tokens on the Fly!
---
_**TL;DR**: Before Facebook's migration to OAuth 2.0, it was possible to hijack a valid access token of any given pre-authorized Facebook app by injecting a specially-crafted iframe through a simple MITM attack._

Back in December of 2013, I've reported the following OAuth security issue to the Facebook security team:<br />
Given a logged in Facebook user, a Man-In-The-Middle attack and a user-authorized Facebook application (say, Skype), an attacker can easily obtain/hijack an access token—while being transmitted over HTTP—for such application even if both of the canvas URL and the website URL of that app are "HTTPS" URIs. Here's how it goes: <br />
1- An attacker first initiates a MITM attack against the victim's network.<br />
2- The attacker waits for the victim to browse to any HTTP website, then he (the attacker) injects such malicious iframe:
{% highlight html linenos %}
<iframe src="https://www.facebook.com/dialog/oauth?redirect_uri=http%3A%2F%2Flogin.skype.com%2Flogin%2Foauth%3Fapplication%3Daccount&client_id=260273468396&response_type=token" width="0" height="0" />
{% endhighlight %}
3- The victim's web browser initiates a `GET` request to the endpoint specified in the source attribute of the injected iframe, and gets a 302 redirect back pointing to:
{% highlight html linenos %}
http://login.skype.com/login/oauth?application=account#access_token=XXXXXXXXX
{% endhighlight %}
4- The attacker now intercepts that `GET` request (as it's transmitted over a regular HTTP connection, not HTTPS), and simply extracts the leaked access_token from the hash fragment.

The **flaw** here is that in spite of the presence of a secure canvas/website URL (for Skype, in this example), Facebook still regards the HTTP version of the canvas/website URL to be a valid value for the '**redirect\_uri**' parameter, causing a hole in the secure authentication flow....

***Responsible Disclosure:***<br />
I got this response when I first reported this issue to the Facebook security team:

> Hi Ahmed,
> 
> I sincerely apologize for the delay in responding to this report. We'd actually received an earlier report from another researcher regarding this same issue. In response to that report, we've been working on limiting this behavior when it comes to our official apps, since they're pre-authorized. For other apps, unfortunately, fully preventing this would mean requiring any site integrating with Facebook to use HTTPS, which simply isn't practical for right now.
> 
> Thanks,<br /> 
> [redacted]<br />
> Security<br />
> Facebook

That being said, though this issue hasn't been fully resolved until the actual deprecation of OAuth 1.x (and migrating to OAuth 2.x instead), there were some things to be done at least on the client-side end to mitigate such type of vulnerability, including but not limited to, using the [HTTPS Everywhere](https://www.eff.org/HTTPS-everywhere) browser extension and proactively not authorizing any FB app that doesn't provide a secure Canvas URL.
