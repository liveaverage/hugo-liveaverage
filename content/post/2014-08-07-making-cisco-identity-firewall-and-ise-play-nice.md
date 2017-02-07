---
title: Making Cisco Identity Firewall, CDA, and ISE play nice
author: JR
type: post
date: 2014-08-07T14:03:32+00:00
url: /features/coding/making-cisco-identity-firewall-and-ise-play-nice/
categories:
  - Coding
  - Infrastructure Management

---
As of Cisco CDA Patch 2, identity mappings provided via Cisco ISE are natively supported. This means you can authenticate against ISE, which may in turn authenticate against LDAP or Active Directory, and subsequently notify one or more Cisco CDA servers that a new user-to-IP mapping exists. Cisco accomplishes this exchange of authenticated identities via syslog messages. ISE is configured to forward syslog messages to the CDA server(s), and the CDA server(s) have the sending ISE server(s) configured as a syslog &#8220;client.&#8221;<figure id="attachment_7493" style="width: 300px" class="wp-caption alignleft">

[<img class="size-medium wp-image-7493" src="http://liveaverage.com/wp-content/themes/mimbo2.2/images/20140729_IDFW_Identity_Digestion_ISE-300x232.jpg" alt="Visio of Cisco IDFW Identity Digestion from ISE" width="300" height="232" srcset="http://liveaverage.com/wp-content/themes/mimbo2.2/images/20140729_IDFW_Identity_Digestion_ISE-300x232.jpg 300w, http://liveaverage.com/wp-content/themes/mimbo2.2/images/20140729_IDFW_Identity_Digestion_ISE.jpg 748w" sizes="(max-width: 300px) 100vw, 300px" />][1]<figcaption class="wp-caption-text">Cisco IDFW Identity Digestion from ISE via Syslog-NG</figcaption></figure> 

We didn&#8217;t wait for this native support at our organization since we needed identities consumed by Cisco WLC via ISE to be available before the general release of Patch 2. How&#8217;d we manage this? After successful authentication via ISE, we would forward syslog RADIUS accounting messages to a syslog-ng server. We&#8217;d filter only the messages we needed and pass off the necessary information to a Python app listening for input on stdin. This app digests the information and creates its own RADIUS accounting packet that gets forwarded to an array of CDA and/or AD agent servers. This method works with their legacy AD agent server _and_ non-Patch 2 CDA appliances.

Why aren&#8217;t we using the native solution? We recently encountered problems with proper user-to-IP mappings being overwritten by machine-to-IP mappings forwarded from ISE. Since we&#8217;re using ISE for user and machine auth, it does make sense to see user _and_ machine mappings, but because only one user can be affiliated with any one IP address, the desired user-to-IP mappings occasionally get overwritten with machine names or MAC addresses. We reverted back to our own in-house solution and I&#8217;m posting it here in hopes it will help anyone else experiencing the problem. There are a few requirements to get this working:

  * Forward syslog entries (filtered at your own discretion) from ISE to any syslog/syslog-ng server
  * Configure the necessary source, filter, and/or destination on your syslog server. This assumes you&#8217;re already listening for network sources.
  * Add your syslog server as a &#8220;Consumer Device&#8221; on all CDA servers
  * Install Python & <a title="PyRad fork with cisco-avpair support" href="https://github.com/andreynpetrov/pyrad/tree/a9ca1d1fe756a93300638eed144357a4a10f81a3" target="_blank">PyRad</a> on your syslog server
  * Configure the Python app, and associated modules, below:

Install <a href="https://github.com/andreynpetrov/pyrad/tree/a9ca1d1fe756a93300638eed144357a4a10f81a3" target="_blank">PyRad </a>first. I actually used a fork that had the correct support for cisc0-avpair. Next download the module used by the app being called from syslog-ng:

<div class="code-embed-wrapper">
  <pre class="language-python code-embed-pre line-numbers" ><code class="language-python code-embed-code">import random, socket, sys, logging
import pyrad.packet

from pyrad.client import Client
from pyrad.dictionary import Dictionary

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

#Logger Console Handler
ch = logging.StreamHandler() #StreamHandler logs to console
ch.setLevel(logging.DEBUG)
ch_format = logging.Formatter(&#039;%(asctime)s - %(message)s&#039;)
ch.setFormatter(ch_format)
logger.addHandler(ch)

#Logger File Handler
fh = logging.FileHandler(".\{0}.log".format(__name__))
fh.setLevel(logging.WARNING)
fh_format = logging.Formatter(&#039;%(asctime)s - %(name)s - %(levelname)-8s - %(message)s&#039;)</code></pre>
  
  <div class="code-embed-infos">
    <a href="https://gist.github.com/8944967" title="See mod_radacct.py" target="_blank" class="code-embed-name">mod_radacct.py</a> <a href="https://gist.githubusercontent.com/liveaverage/8944967/raw/b1439ee6e0b31f80f53636c2b9ac784582628d26/mod_radacct.py" title="Back to mod_radacct.py" class="code-embed-raw" target="_blank">view raw</a>
  </div>
</div>

Now grab the program listening for syslog-ng $MESSAGE input. I&#8217;ve split the file into a configuration module (mod\_cda.py), and the actual program (update\_cda.py). Modify to suite your environment:

mod_cda.py

<div class="code-embed-wrapper">
  <pre class="language-python code-embed-pre line-numbers" ><code class="language-python code-embed-code">import mod_radacct

# Dictionary path (can be relative or absolute):
dictionary = "dictionary"

# If missing a user domain in the syslog msg, this is provided:
default_domain = "DefaultDomain"

# List of CDA identity maintainers. Can include legacy AD Agent servers:
servers = [ "cda1", "cda2", "ada1", "ada2" ]

# RADIUS secret:
secret = "yourRadiusSecret"
</code></pre>
  
  <div class="code-embed-infos">
    <a href="https://gist.github.com/22139f9adf9423143cf2" title="See mod_cda.py" target="_blank" class="code-embed-name">mod_cda.py</a> <a href="https://gist.githubusercontent.com/liveaverage/22139f9adf9423143cf2/raw/d6277481b0e3edadc68e16935c49bf49819dd1a6/mod_cda.py" title="Back to mod_cda.py" class="code-embed-raw" target="_blank">view raw</a>
  </div>
</div>

update_cda.py

<div class="code-embed-wrapper">
  <pre class="language-python code-embed-pre line-numbers" ><code class="language-python code-embed-code">#!/usr/bin/env python

import logging, re, sys, time
import mod_cda

PYTHONUNBUFFERED = "true"

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

#Logger Console Handler
ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)
ch_format = logging.Formatter(&#039;%(asctime)s - %(message)s&#039;)
ch.setFormatter(ch_format)
logger.addHandler(ch)

#Logger File Handler; change to your desired path
fh = logging.FileHandler("/usr/local/cda/{0}.log".format("UpdateCDA"))
fh.setLevel(logging.DEBUG)</code></pre>
  
  <div class="code-embed-infos">
    <a href="https://gist.github.com/ac8a11658e773f46da40" title="See update_cda.py" target="_blank" class="code-embed-name">update_cda.py</a> <a href="https://gist.githubusercontent.com/liveaverage/ac8a11658e773f46da40/raw/e6f524a32105a7aafaa1e05588cb849a2ce3d4dd/update_cda.py" title="Back to update_cda.py" class="code-embed-raw" target="_blank">view raw</a>
  </div>
</div>

syslog-ng.conf (excerpt)

<div class="code-embed-wrapper">
  <pre class="language-bash code-embed-pre line-numbers" ><code class="language-bash code-embed-code">### Filter out ISE hosts that should be sending specific messages for Device/IP association:

filter f_ise_host 	{ 	(
				host("4.4.4.4") or
				host("8.8.8.8") or
				host("ise01") or
				host("ise02")
				);
			};

### The username and Framed-IP we&#039;re looking for are in the watchdog updates AFTER accounting "start" msgs:
filter f_ise_auth { match(".*RADIUS Accounting watchdog update.*" value ("MESSAGE")); };


destination d_NS_ISE { file("/var/log/syslog-ng/NS_logs/NS_ISE/$SOURCEIP/$SOURCEIP.log"); };

destination d_NS_pythonCDA 	{ 
					program("/usr/local/cda/update_cda.py"
					template("$MSG\n")
					flags(no_multi_line)
					flush_lines(1)
					flush_timeout(1000)
					); 
				};

log {	source(s_network);
	filter(f_ise_host);
	filter(f_ise_auth);
	destination(d_NS_ISE);
	destination(d_NS_pythonCDA);
};</code></pre>
  
  <div class="code-embed-infos">
    <a href="https://gist.github.com/50e30d49bb93151f0576" title="See syslog-ng.conf" target="_blank" class="code-embed-name">syslog-ng.conf</a> <a href="https://gist.githubusercontent.com/liveaverage/50e30d49bb93151f0576/raw/5e1df9e248fc1f7952e3b5eb4fdeb588ade0128b/syslog-ng.conf" title="Back to syslog-ng.conf" class="code-embed-raw" target="_blank">view raw</a>
  </div>
</div>

Now you should be able to restart syslog-ng and see your application running (ps -ef | grep update_cda). It will continually listen for new messages and process them as needed. Feel free to change any of the configured logging levels to your preferred verbosity.

 [1]: http://liveaverage.com/wp-content/themes/mimbo2.2/images/20140729_IDFW_Identity_Digestion_ISE.jpg