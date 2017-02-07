---
title: Lenovo RS110 Review
author: JR
type: post
date: 2009-10-07T15:34:01+00:00
url: /news/lenovo-rs110-review/
syntaxhighlighter_encoded:
  - 1
categories:
  - News

---
<img class="size-medium wp-image-260     alignleft" style="margin: 15px;" title="Lenovo RS110 Server" src="http://liveaverage.com/wp-content/themes/mimbo2.2/images/74840064-300x300-0-0_lenovors110xeonqcx33604gbsas331.jpg" alt="Lenovo RS110 Server for SMBs" width="210" height="210" srcset="http://liveaverage.com/wp-content/themes/mimbo2.2/images/74840064-300x300-0-0_lenovors110xeonqcx33604gbsas331-150x150.jpg 150w, http://liveaverage.com/wp-content/themes/mimbo2.2/images/74840064-300x300-0-0_lenovors110xeonqcx33604gbsas331.jpg 300w" sizes="(max-width: 210px) 100vw, 210px" />

I recently had the [unfortunate] opportunity of rolling-out (2) Lenovo RS110 servers targeted for SMBs. We seemed to fit the market for this relatively new Lenovo offering, but the product failed to meet the needs and expectations of my environment. The intention was to launch one of these two units with an Ubuntu/Debian-based Linux distribution to host our Zimbra mail server (currently residing virtual Ubuntu 6.06 LTS machine, with a host OS of Ubuntu 8.04 LTS 64-bit). Here&#8217;s some brief hardware highlights:

<div class="bulletListWrapper">
  <ul id="PO_ctl02_blmObjText__bulletList" class="bulletList">
    <li>
      Intel® Xeon® Dual Core Processor E3110
    </li>
    <li>
      3.00Ghz
    </li>
    <li>
      6MB cache
    </li>
    <li>
      2GB (we upgraded to 4Gb of PC26400 RAM)
    </li>
    <li>
      Rack(2&#215;2) &#8212; Rails and mounting hardware included
    </li>
    <li>
      LSISAS1064e Raid Controller (0,1,10) &#8212; Hardware RAID, NOT fakeRAID
    </li>
    <li>
      16x Max DVD-ROM
    </li>
    <li>
      Dual Gigabit Ethernet, +1 Ethernet Management Port
    </li>
    <li>
      no preload for OS
    </li>
  </ul>
</div>

<div class="bulletListWrapper">
  Yes, it looks pretty decent for a $800-$900 price-tag. But note some caveats: to obtain hard-disk carriers for this RS110, you <em>have</em> purchase Lenovo hard-disks. You cannot purchase the carriers independent of a hard-disk. You can, however, make your own brackets or use hard-disk brackets from another manufacture that fit in the hot-swap bay(s). This was unknown to me before my purchase. Had I know this, I would have kept looking at other options. Another snag: the hardware RAID controller is/was only [fully] supported by one of the four different Linux distributions I attempted to install. The comprehensive list of attempted distro installs:
</div>

<div class="bulletListWrapper">
  <strong><br /> </strong>
</div>

<table style="height: 129px;" border="0" cellspacing="0" cellpadding="5" width="319">
  <tr>
    <td>
      <strong>Debian Server 5.03 Lenny</strong>
    </td>
    
    <td>
      <strong>[Joy]</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Ubuntu Server 6.06.1 LTS</strong>
    </td>
    
    <td>
      <strong>[No go]</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Ubuntu Server 6.06.2 LTS</strong>
    </td>
    
    <td>
      <strong>[No go]</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Ubuntu Server 8.04 LTS</strong>
    </td>
    
    <td>
      <strong>[No go]</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Ubuntu Server 9.04</strong>
    </td>
    
    <td>
      <strong>[No go]</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>Red Hat Fedora 11</strong>
    </td>
    
    <td>
      <strong>[No go]</strong>
    </td>
  </tr>
  
  <tr>
    <td>
      <strong>CentOS Server 5.3</strong>
    </td>
    
    <td>
      <strong>[No go]</strong>
    </td>
  </tr>
</table>

I won&#8217;t lie &#8212; I did complete successful installs on Ubuntu 8.04 and Ubuntu 9.04; however, when attempting to copy large files to a Samba share (or via SCP, didn&#8217;t matter what protocol), the RS110 would crash&#8230; hard&#8230; with perpetual disk i/o errors until a hard reset was completed. Debian 5.03 was the only distribution that installed successfully _and_ operated normally under a commonplace workload. Unfortunately, Zimbra offers no support for Debian 5.03 at this time. The RS110 claims support for Red Hat Enterprise Linux and Suse, but I&#8217;m not in the mood to start mixing too many Linux distros in my environment (I&#8217;m already running Debian, Ubuntu, and flavors of OpenBSD), nor do I feel up for paying a yearly RHEL subscription/support fee. So, I executed my last option for utilizing an RS110 as my physical mail server: throw in a supported RAID card. I installed a 3Ware (AMCC) 9650SE-2LP card in the available PCI-E riser-card slot. Note that the hot-swap back-plane does NOT use generic SATA/SAS data or power connections, which means I had use a molex to SATA Y-connector for power and two separate, standard SATA data cables. This meant the back-plane was not being used at all&#8230;

Apparently this &#8220;configuration,&#8221; is what Lenovo Support staff called an &#8220;unsupported,&#8221; hardware configuration. A generic PCI-E RAID controller in a PCI-E slot on the RS110 is unsupported. The 3Ware RAID bios never did post, regardless of several Lenovo BIOS setting changes and physically removing the LSI card from the RS110. Following this last bit of frustration, I contacted the vendor I purchased from and indicated my dissatisfaction with the Lenovo RS110. I&#8217;ve decided to keep one as a Windows 2003 R2 VMWare host, but I&#8217;ve already packaged the other 110 for return.

**Bottom-line:** If you&#8217;re running a low-load Windows 2003 server, this product _might_ be ideal for your environment. If you intend to run anything else on the hardware, _stay away_. Support was unclear about the &#8220;unsupported,&#8221; configuration, so I wouldn&#8217;t plan on even using the one available PCI-E slot. The forced purchase of Lenovo hard-disks versus just the disk carriers feels like extortion&#8230;