---
title: Perl my way to happiness
author: JR
type: post
date: 2008-04-25T19:00:49+00:00
url: /features/coding/perl-my-way-to-happiness/
Image:
  - bash.jpg
categories:
  - Coding
tags:
  - perl zimbra zmprov provisioning

---
If only everything was as easy & straight-forward as account provisioning in Zimbra:

[shell]!/usr/bin/perl

\# ZCS IMPORT SCRIPT
  
\# Provided by : ZCS Wiki
  
\# Modified by : J.R.
  
\# Last Change : 2008.04.09
  
#
  
\# Lookup the valid COS (Class of Service) ID in the interface or like this
  
my $cosid = \`su &#8211; zimbra -c &#8216;zmprov gc apd |grep zimbraId:&#8217;\`;
  
$cosid =~ s/zimbraId:\s\*|\s\*$//g;

while (<>) {
  
chomp;

\# CHANGE ME: To the actual fields you use in your CSV file
  
my ($email, $password, $first, $last) = split(/\,/, $_, 4);

my ($uid, $domain) = split(/@/, $email, 2);

print qq{ca $uid\@$domain $password\n};
  
print qq{ma $uid\@$domain zimbraCOSid "$cosid"\n};
  
print qq{ma $uid\@$domain givenName "$first"\n};
  
print qq{ma $uid\@$domain sn "$last"\n};
  
print qq{ma $uid\@$domain cn "$uid"\n};
  
print qq{ma $uid\@$domain displayName "$first $last"\n};

#Set the user&#8217;s reply or canonical address
  
print qq{ma $uid\@$domain zimbraMailCanonicalAddress $uid\@cityof\*****.org\n};

#Add e-mail account alias for multiple domains
  
#Verify domain is correctly working for provisioning aliases

print qq{aaa $uid\@$domain $uid\@cityof\*****.com\n};

#Add all users to a general distribution list and terminate
  
#Add multiple distro-lists if desired
  
print qq{adlm dept.all\@cityof**\*\\*\*.org $uid\@cityof\*\*\***.org\n};
  
print qq{\n};

}[/shell]