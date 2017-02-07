---
title: Nagios check_http problems with Microsoft System Center 2012 Endpoint Protection
author: JR
type: post
date: 2012-11-07T22:25:03+00:00
url: /news/nagios-check_http-problems-with-microsoft-system-center-2012-endpoint-protection/
categories:
  - Infrastructure Management
  - Nagios
  - News

---
Someone important suggested this get posted _somewhere_ so that anyone else experiencing** check_http** socket timeout problems with client/servers running Microsoft System Center 2012 Endpoint Protection would have a clue as to what could be causing issues:

If you&#8217;re receiving strange/unexpected timeouts from the check_http plug-in when running it against a server using Microsoft System Center 2012 Endpoint Protection, the Network Inspection Service may be blocking your check attempt.

Don&#8217;t bother trying to force an agent string or any other weird options; either switch to header-checks-only [no body] using the **-N** option with check_http, or disable your Network Inspection service entirely (not recommended). I couldn&#8217;t locate granular settings for the Network Inspection service included with Endpoint protection 2012, though I saw references to promising settings included in 2010 [applied/managed via GPOs and custom ADMX Administrative Templates].

I haven&#8217;t had time to troubleshoot this problem further, but I&#8217;m likely to encounter it again when I need layer-7 string checks for hosts using Endpoint Protection 2012 with NIS enabled.