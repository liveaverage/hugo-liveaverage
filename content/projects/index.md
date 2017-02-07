---
title: Projects
author: JR
type: page
date: 2008-04-25T20:04:38+00:00
syntaxhighlighter_encoded:
  - 1

---
[ad name=&#8221;LA2&#8243;]

I&#8217;m usually working on something. If it&#8217;s not a home project, it&#8217;s a fun tech-project for SMB applications. This is where to get a nice breakdown of all current projects, tech or otherwise, that I&#8217;m managing. Every now and then the two sections may converge on a single project, such as those involving the implementation of monetary projects from work with personal household projects. On-going biggies:

#### **Zimbra Collaboration Suite &#8211; _Completed_** 

  * Testing
  * Implementation as a virtual machine/appliance on Linux Host OS (Ubuntu 6.06 LTS)
  * Implementation with 100+ users
  * Integration with [existing] Active Directories (Windows Server 2003 Platforms only) 
      * Leverage existing usernames and passwords via OpenLDAP
      * Leverage existing domain- & forest-wide Global Account Listings (GAL)
  * Migration of existing domain(s)
  * Initiation of recently registered domain(s)

#### **MailArchiva OSS E-mail Archiving Solution &#8211; _Completed_** 

  * What&#8217;s an e-mail system without an archiving solution to complement it? Everyone should know e-mail record compliance is a hot topic among SMBs, but anyone should know it&#8217;s even more vital & strict in government applications.
  * Zimbra utilizes a modded version of Postfix as its internal MTA (Mail Transfer Agent) and MailArchiva, much like Zimbra, provides a FOSS version of their software that plays nice with Postfix (2.4.6 or greater).

#### **Telco Carrier Migration (PRI + 250 DID No.&#8217;s) &#8211; _Completed_** 

  * Move a single PRI from Windstream to Cox Communictions.
  * Port 250+ DID Numbers.
  * Partition new PRI service using Adtran Atlas 550; Route sub-blocks of DIDs to multipe PBX systems.
  * Minimize downtime associated with service migration and number porting.

#### **Untangle UTM Deployments (with ntop monitoring) &#8211; _Completed_**

  * Utilize FOSS to create a transparent proxy/gateway for multiple VLANs/Subnets.
  * Configure protocol, web, spyware, and spam filters on each UTM.
  * Configure detailed client reporting (Untangle).
  * Configure traffic and network usage analysis (using ntop).

#### **Replacement of Cisco equipment with Open Source alternatives (Cisco &#8211; > pfSense) &#8211; _Completed_**

  * Migrate/Translate all ACLs and rules from an overworked Cisco 1800 Series router to a pfSense box.
  * Redeploy Cisco VPN services on the new box using PPTP (128-bit encryption) and OpenVPN (256-bit encryption).
  * Enable remote syslog logging and monitoring.