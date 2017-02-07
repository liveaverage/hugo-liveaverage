---
title: 'New fun with old toys: Adtran Atlas 550 + Free 411'
author: JR
type: post
date: 2009-07-08T21:10:26+00:00
url: /features/infrastructure-management/new-fun-with-old-toys-adtran-atlas-550-free-411/
syntaxhighlighter_encoded:
  - 1
categories:
  - Infrastructure Management

---
![][1]<img class="alignleft" style="margin: 10px;" title="Goog-411" src="http://www.google.com/images/logos/goog-411_logo.png" alt="Goog-411 is a spectacular, free alternative to traditional directory svc." width="209" height="40" />

As a result of my frugality, I got my hands on an Adtran Atlas 550 to be used for partitioning a single PRI into two PRIs at my workplace. In addition to the partitioning, this system provides the added bonus of dynamic number substitution. What&#8217;s this mean for me? Substituting costly 411 directory service for free Goog-411 service (which I prefer over traditional 411). I&#8217;m also able to create a number rejection list to block those NSFW 900* calls [or variation thereof], but the number substitution templates seem much more interesting&#8230;

How to do it:

>   * Telnet to **yohost.yodomain** and log-in
>   * _Dial Plan_ > _Network Term_ > _Interface #_ (1 in my case) > Enter
>   * Select _Substitution Template_
>   * Enter the Original DNIS number (the phone number originally dialed)
>   * Enter the Substitution DNIS number (the phone number you&#8217;d like to dial)
>   * Go back to the main menu and log-out

Remember, this change is transparent (and instant &#8212; no need to write/commit the config to startup), so the next time callers decide to hit up the 411 directory service, their call should be automagically routed to 1-800-goog411 (1-800-4664411). Since we&#8217;re a fairly small office I don&#8217;t think this number substituion presents a problem with Googles Terms of Service, but if you&#8217;re considering this change on a massive scale  I&#8217;d consider reading over <a title="Review Google's Terms of Service" href="http://www.google.com/accounts/TOS" target="_blank">Google&#8217;s TOS agreement</a> (particularly section 5.3).

 [1]: file:///C:/DOCUME~1/ADMIN~1.IT0/LOCALS~1/Temp/moz-screenshot-11.jpg