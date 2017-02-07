---
title: OpenFiler errors that only I seemed to experience…
author: JR
type: post
date: 2011-07-26T16:51:03+00:00
url: /features/coding/openfiler-errors-that-only-i-seemed-to-experience/
categories:
  - Coding
  - Infrastructure Management

---
Well, I&#8217;ve finally deployed some production Openfiler ESA 2.99.1 machines as home-brew iSCSI boxes, primarily used for backups or low-stress virtual storage. Yes, they&#8217;re great &#8212; my basic write speeds on a Core 2 Duo box (recycled Dell Precision 390 workstation with 2GB of RAM and a single 1TB drive \*no\* RAID):

[shell]
  
administrator@mail:/backup-iscsi$ sudo dd if=/dev/zero of=garbage bs=131072 count=20000
  
20000+0 records in
  
20000+0 records out
  
2621440000 bytes (2.6 GB) copied, 40.8493 s, 64.2 MB/s
  
[/shell]

I&#8217;m pretty happy with that. What I&#8217;m not please with is a 56MB log (/var/log/secure) filled with strange messages:

[shell]Jul 26 12:25:46 e0-002 sudo: openfiler : TTY=unknown ; PWD=/opt/openfiler/var/www/htdocs ; USER=root ; COMMAND=/usr/bin/uptime
  
Jul 26 12:25:46 e0-002 sudo: PAM unable to dlopen(/lib64/security/pam\_gnome\_keyring.so)
  
Jul 26 12:25:46 e0-002 sudo: PAM [error: /lib64/security/pam\_gnome\_keyring.so: cannot open shared object file: No such file or directory]
  
Jul 26 12:25:46 e0-002 sudo: PAM adding faulty module: /lib64/security/pam\_gnome\_keyring.so[/shell]

Okay, so the first message isn&#8217;t strange. In fact, it&#8217;s normal. But I couldn&#8217;t figure out why a non-GUI (yes, it has a web interface, but no default window manager) distro was trying to load the Gnome keyring shared object. I still don&#8217;t know why. What I do know is how to get rid of this message:

  1. Verify which PAM files reference this shared object:
  1. \[shell\]\[root@e0-002 pam.d\]# cd /etc/pam.d/
  
    [root@e0-002 pam.d]# grep -i -n &#8216;pam\_gnome\_keyring.so&#8217; *
  
    system-auth:5:# FL: Have (patched) pam\_gnome\_keyring.so grab the password before pam_unix.so
  
    system-auth:6:auth optional pam\_gnome\_keyring.so
  
    system-auth:16:password optional pam\_gnome\_keyring.so[/shell]

  2. In this instance, the &#8216;system-auth&#8217; file contained these troublesome pam\_gnome\_keyring.so entries.
  3. Fire-up your favorite text editor and comment those lines.
  4. Reboot & enjoy a log without an absurd amount of irrelevant errors.

<div>
  <a title="The original thread mentioning the problem." href="https://lists.openfiler.com/viewtopic.php?id=6228" target="_blank">I found this referenced <em>once</em> on the OpenFiler forums</a>, but no answer or explanation. I hope this helps someone else that might experience a similar problem.
</div>

<div>
  I&#8217;ll also be doing a write-up related to OpenFiler volume backups with LVM snapshots.
</div>