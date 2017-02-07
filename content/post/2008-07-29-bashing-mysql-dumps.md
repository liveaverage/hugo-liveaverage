---
title: Bashing MySQL Dumps
author: JR
type: post
date: 2008-07-29T14:45:14+00:00
excerpt: A quick set of scripts to dump specified databases from MySQL and mail a copy of the log to a specified e-mail address (requires Per
url: /features/coding/bashing-mysql-dumps/
Image:
  - bash.jpg
syntaxhighlighter_encoded:
  - 1
categories:
  - Coding
tags:
  - backup
  - bash
  - dumps
  - mysql
  - script
  - shell
  - sql

---
A quick set of batch scripts I wrote up (two of the three scripts, anyway) for dumping all of my (specified) MySQL databases into an archive for backup. The log mailing (**emailsql.pl**)requires Perl and the MIME:Lite module to correctly function. I&#8217;ve also utilized a wrapper script so the log outputs to a separate file [to be mailed]. There&#8217;s also a &#8216;dummy&#8217; log file I use in my crontab file, though this isn&#8217;t really necessary:

<!--more-->

<span style="color: #800000;"><strong>mysqlbackupwrapper.sh</strong></span>

[shell]
  
#!/bin/bash
  
#Wrapper script used to call the primary backup script and output to a specified file
  
sh /home/administrator/scripts/mysqlbackups > /home/administrator/scripts/sql.backup.log 2>&1
  
[/shell]

<span style="color: #800000;"><strong>mysqlbackup.sh</strong></span>:

[shell]
  
#!/bin/sh
  
#Timestamp for your logs:
  
date

#Dump the databases &#8211; Make sure to specify your root or user password following the -p switch:
  
mysqldump -uroot -p &#8211;opt intraforum > /home/administrator/scripts/sqldata/intra_apdforum.sql
  
mysqldump -uroot -p &#8211;opt joomla\_intranet > /home/administrator/scripts/sqldata/intra\_intranet.sql
  
mysqldump -uroot -p &#8211;opt mysql > /home/administrator/scripts/sqldata/intra_mysql.sql
  
tar -zcvf /home/administrator/scripts/sqldata.tgz /home/administrator/scripts/sqldata/*.sql
  
echo
  
echo "Backup completed successfully for: "
  
echo
  
echo "MySQL &#8211; PHPbb3 Forum"
  
echo "MySQL &#8211; Joomla 1.0.X Intranet"
  
echo "MySQL &#8211; Intranet MySQL Tables"
  
echo
  
echo "Copying to SERVER.yourdomain.local . . . ."
  
echo

#Use SCP to transfer to file so you can verify successful backups &#8212; Make sure to use identity/keys for SCP instead of a password:
  
scp -v -i /home/administrator/identity /home/administrator/scripts/sqldata.tgz administrator@this.host:/backup/sqldata_backup.tgz

perl /home/administrator/scripts/emailsql.pl
  
[/shell]

<span style="color: #800000;"><strong>emailsql.pl (I did not write this one)</strong></span>:

Instead of just sending the text of the log file, this script attaches the file and sends the message:

[shell]

#!/usr/bin/perl -w
  
use MIME::Lite;

$msg = MIME::Lite->new(
  
From    => &#8216;Backup Log&#8217;,
  
To      => &#8216;liveaverage@yourdomain.org&#8217;,
  
Subject => &#8216;MySQL Data Backup &#8211; Intranets&#8217;,
  
Type    => &#8216;text/plain&#8217;,
  
Data    => "See the attached log for details on the most recent MySQL Database Dumps.");

$msg->attach(
  
Type       =>&#8217;text/plain&#8217;,
  
Path       =>&#8217;/home/administrator/scripts/sql.backup.log&#8217;,
  
Filename   =>&#8217;sql.backup.log&#8217;,
  
Disposition        =>&#8217;attachment&#8217;);

$msg->send;

[/shell]