---
title: Altermime, Postfix/Zimbra, and Headaches
author: JR
type: post
date: 2009-06-24T19:43:11+00:00
url: /features/coding/altermime-postfixzimbra-and-headaches/
categories:
  - Coding

---
> _EDIT: I have since removed altermime after installing a MailScanner spam relay for our Zimbra server to use. Because, by default, MailScanner appends a default signature to all outbound email, it was very simple to modify the signature rules to accomodate our mandatory disclaimers for different domains._

I had the pleasure of applying mandatory disclaimers to all [outbound] emails at my workplace today&#8230; ~Joy~ &#8230; I had the assumption it&#8217;d be rather easy, but Altermime and Postfix were a bit finicky to work with. After editing the **master.cf** I ended up customizing my own &#8216;disclaimer&#8217; shell script.

<!--more-->

[shell]
  
#!/bin/sh
  
INSPECT_DIR=/var/spool/filter
  
SENDMAIL=/opt/zimbra/postfix/sbin/sendmail

\# Exit codes from
  
EX_TEMPFAIL=75
  
EX_UNAVAILABLE=69

\# Clean up when done or when aborting.
  
trap "rm -f in.$$" 0 1 2 3 15

\# Start processing.
  
cd $INSPECT\_DIR || { echo $INSPECT\_DIR does not exist; exit $EX_TEMPFAIL; }

cat > in.$$ || { echo Cannot save mail to file; exit $EX_TEMPFAIL; }

\# Verify this mail is not incoming or internal-only
  
\# We don&#8217;t need disclaimers for either one of these cases.

\# Debug:
  
#echo "output: $from_address" >> /tmp/tempoutput.txt

#Grab the from address:

from_address=\`grep -m 1 "From:" in.$$ | cut -d "<" -f 2 | cut -d ">" -f 1\`

#Verify whether your domain is in the from address.
  
#If it is, proceed to distinguish WHICH domain is sending outgoing mail and tag it appropriately:
  
#If not, then that would be incoming mail, so leave it alone:

#Additional (else if) conditional checks will be added to determine if the email is inner-office comm:
  
\# to_address= \`grep -m 1 "To:" in.$$ | cut -d "<" -f 2 | cut -d ">" -f 1\`
  
\# if [[ $from\_address == \*domain\* && to\_address == \*domain\* ]]; then &#8230;.
  
#
  
\# This additional condition requires more debugging&#8230;

if [[ $from_address == \*domain\* ]]; then

\# Debug:
  
#echo
  
#echo "FROM: $from_address" >> /tmp/tempoutput.txt
  
#echo "THIS GETS ALTERMIMED" >> /tmp/tempoutput.txt
  
#echo

#Check to see which domain is sending outgoing email,
  
#then tag it with the appropriate disclaimer:

if [[ $from_address == \*subdomain.domain\* ]]; then

\# Debug echo "THIS GETS APDD" >> /tmp/tempoutput.txt
  
/usr/bin/altermime &#8211;input=in.$$ \
  
&#8211;disclaimer=/opt/zimbra/postfix/conf/disclaimers/apd-disclaimer.txt \
  
&#8211;disclaimer-html=/opt/zimbra/postfix/conf/disclaimers/apd-disclaimer.txt \
  
&#8211;xheader="X-Public-Record:" || { echo Message content rejected; exit $EX_UNAVAILABLE; }
  
else
  
\# Debug echo "THIS GETS COAD" >> /tmp/tempoutput.txt
  
/usr/bin/altermime &#8211;input=in.$$ \
  
&#8211;disclaimer=/opt/zimbra/postfix/conf/disclaimers/coa-disclaimer.txt \
  
&#8211;disclaimer-html=/opt/zimbra/postfix/conf/disclaimers/coa-disclaimer.txt \
  
&#8211;xheader="X-Public-Record:" || { echo Message content rejected; exit $EX_UNAVAILABLE; }

fi
  
fi

\# Might need to remove -i switch for truncation problems depending on your MTA version&#8230;

$SENDMAIL -i "$@" < in.$$

exit $?
  
[/shell]