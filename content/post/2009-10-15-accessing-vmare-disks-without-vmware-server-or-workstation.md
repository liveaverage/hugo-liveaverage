---
title: Accessing VMare Disks — without VMware Server or Workstation
author: JR
type: post
date: 2009-10-15T14:23:59+00:00
url: /news/accessing-vmare-disks-without-vmware-server-or-workstation/
syntaxhighlighter_encoded:
  - 1
categories:
  - News

---
I spent more time than necessary looking for a VMware disk mount utility to use on a Linux-based distribution. For windows, there&#8217;s the [VMWare Workstation Disk Mount Utility (5.5)][1] that can installed with the [VMWare DiskMount GUI][2], but I couldn&#8217;t find the simple **vmware-mount.pl** program for Linux distros. Instead, I had to dig it out of a compatible VMWare Server package [for Linux], copy it to /usr/bin/ and create the necessary symlinks for operation. Here&#8217;s a quick breakdown of the steps:

  1. If you don&#8217;t already have it, grab a release of the VMWare Server for Linux &#8212; make sure you download the correct version for your distribution (32-bit or 64-bit). _You do not need to install the package, simply download it and extract the contents. Alternatively, you could download the package to a Windows workstation and transfer the **vmware-mount** utility to your Linux workstation/server via SFTP, SCP, FTP, etc.._
  2. After downloading the VMWare Server package, extract the **vmware-mount** utility from the following directory in the archive: [shell]\vmware-server-distrib\bin\vmware-mount[/shell]
  3. Copy **vmware-mount** to /usr/bin
  
    [shell]cp ~/vmware-mount /usr/bin/[/shell]
  4. The disk mount utility requires a couple of dependencies to run. You can try and run **vmware-mount** to determine what it needs. In my case, on Ubuntu 8.04LTS 64-bit, it required _libcrypto.so.0.9.8_ and _libssl.so.0.9.8_. The utility was looking for these files in a non-existent directory structure: [shell]/usr/bin/libdir/lib/lib*[/shell] 
    Go ahead and create this directory structure (beats installing the VMWare server package), then provide symlinks to the actual lib files required by vmware-mount:
    
    [shell]sudo mkdir -p /usr/bin/libdir/lib/libcrypto.so.0.9.8; sudo mkdir -p /usr/bin/libdir/lib/libssl.so.0.9.8</pre>
    
    sudo ln -s /usr/lib/libcrypto.so.0.9.8 /usr/bin/libdir/lib/libcrypto.so.0.9.8/libcrypto.so.0.9.8
  
    sudo ln -s /usr/lib/libssl.so.0.9.8 /usr/bin/libdir/lib/libssl.so.0.9.8/libssl.so.0.9.8[/shell]</li> 
    
      * That&#8217;s it! Now you should be able to run **./vmware-mount** to determine proper command usage.</ol>

 [1]: http://www.vmware.com/download/eula/diskmount_ws_v55.html "Download the VMWare DiskMount Utility"
 [2]: http://vmxbuilder.com/vmware-diskmount-gui/ "Download the GUI for the VMWare DiskMount Utility"