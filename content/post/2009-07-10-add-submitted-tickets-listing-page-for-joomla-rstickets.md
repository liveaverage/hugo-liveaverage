---
title: Add ‘Submitted Tickets’ Listing Page for Joomla! RSTickets
author: JR
type: post
date: 2009-07-10T17:15:21+00:00
url: /features/coding/add-submitted-tickets-listing-page-for-joomla-rstickets/
syntaxhighlighter_encoded:
  - 1
categories:
  - Coding

---
<img class="alignright" title="Joomla!" src="http://cdn.joomla.org/images/logo.png" alt="" width="235" height="46" />

If you haven&#8217;t heard, <a title="Check out the RSTickets! extension from RSJoomla!" href="http://www.rsjoomla.com/joomla-components/rstickets.html" target="_blank">RSTickets!</a> is an advanced Joomla! Help Desk ticketing system that allows you (or a team of yous) to manage and keep track of your clients&#8217; issues. It&#8217;s actually one of the few effective, useful Help Desk systems available for the Joomla! 1.5+ framework that I would personally recommend. Unfortunately, it&#8217;s still under development and lacks certain features that one may desire, such as a read-only listing page that displays tickets already submitted to you or your department.

<!--more-->

**My Problem:**

I noticed internal network clients were submitting several duplicate tickets related to a shared problem (i.e. printer trouble, network outages, etc.). I couldn&#8217;t blame them since they couldn&#8217;t view previously submitted tickets, so I decided to create a quick + dirty page that pulls _&#8220;open,&#8221;_ or _&#8220;on-hold,&#8221;_ tickets from a specific department&#8217;s table of submitted [active] tickets and displays them on a Joomla! article page (using the Sourcerer plug-in to execute custom PHP with {source} tags). Pasting this code into an article without the Sourcerer plug-in [or some sort of plug-in for executing PHP] _will do nothing._ Also note the _include_ file for making a raw connection to your MySQL database. This is required (and should be stored in a directory with the appropriate permissions to prevent outside read access).

If you&#8217;d like to simply list the number (amount) of tickets (open, closed, on-hold) for _all_ departments, you might want to <a title="Try RSJoomla!'s RSTicket Module for a quick count of all tickets." href="http://www.rsjoomla.com/customer-support/forum/38-rstickets/7805-joomla-module-for-rstickets.html" target="_blank">check the unreleased version of RSJoomla!&#8217;s RSTicket Module.</a>

[php]
  
{source}
  
<?php

include ("includes/connect_custom.php");

$result2 = mysql\_query("SELECT * FROM jos\_rstickets\_tickets WHERE DepartmentId=3 AND (TicketStatus=&#8217;open&#8217; OR TicketStatus=&#8217;on-hold&#8217;) ORDER BY TicketTime ASC") or die(mysql\_error());

echo &#8216;<br />&#8217;;

echo &#8216;<tr><td style="padding-bottom: 10px;" colspan="6">Please check the open, pending, or on-hold tickets listed below before submitting a support ticket. The list below can be utilized as an informal gauge for IT response times. If a duplicate support ticket is submitted, you may cancel it yourself or it will be deleted by the IT Department upon review. Thank you for your cooperation.</td></tr>&#8217;;

echo &#8216;<tr><td style="font-size: 16px;font-weight: bold;padding-bottom: 10px;" colspan="6">SUBMITTED SUPPORT TICKETS:</td></tr>&#8217;;

echo &#8216;<tr><td style="text-align:left; padding-bottom:7px;"><strong>Submitted:</strong></td><td style="padding-bottom:7px;"><strong>Username:</strong></td><td style="padding-bottom:7px;"><strong>Ticket Code:</strong></td><td style="padding-bottom:7px;"><strong>Subject:</strong></td><td style="padding-bottom:7px;"><strong>Status:</strong></td><td style="padding-bottom:7px;"><strong>Priority:</strong></td></tr>&#8217;;

while($row = mysql\_fetch\_array($result2))

{

$custid = $row[&#8216;CustomerId&#8217;];
  
$userquery= mysql\_query("SELECT name FROM jos\_users WHERE id=$custid") or die(mysql_error());

$username = mysql\_fetch\_array($userquery);

echo &#8216;<tr style="font-size: 10px;vertical-align:top;"><td style="width: 102px; overflow: hidden;padding: 3px -10px 3px 3px;">&#8217; . date("Y-m-d H:m", $row[&#8216;TicketTime&#8217;]) . &#8216;</td><td>&#8217; . $username[&#8216;name&#8217;] . &#8216;</td><td>&#8217; . $row[&#8216;TicketCode&#8217;] . &#8216;</td><td style="width: 235px; overflow: hidden;">&#8217; . $row[&#8216;TicketSubject&#8217;] . &#8216;</td><td>&#8217; . strtoupper($row[&#8216;TicketStatus&#8217;]) . &#8216;</td>&#8217;;

if ($row[&#8216;TicketPriority&#8217;]==&#8217;high&#8217;){

echo &#8216;<td style="background-color: red; color: white; font-weight: bold;padding-left: 5px;">&#8217;;

} else if ($row[&#8216;TicketPriority&#8217;]==&#8217;normal&#8217;){

echo &#8216;<td style="background-color: blue; color: white; font-weight: bold;padding-left: 5px;">&#8217;;

} else if ($row[&#8216;TicketPriority&#8217;]==&#8217;low&#8217;){

echo &#8216;<td style="background-color: yellow; font-color: black; font-weight: bold;padding-left: 5px;">&#8217;;}

echo strtoupper($row[&#8216;TicketPriority&#8217;]) . &#8216;</td></tr>&#8217;;

}

echo &#8216;<tr><td style="font-size:16px; padding-top: 20px; padding-bottom: 20px" colspan="6"><strong><a href="index.php?option=com\_rstickets&Itemid=59"><img src="images/M\_images/onsite\_support1.jpg"></a><br /><a href="index.php?option=com\_rstickets&Itemid=59">SUBMIT A NEW SUPPORT TICKET</a></strong></td></tr>&#8217;;

echo &#8216;<br />&#8217;;

?>
  
{/source}
  
[/php]

**connect_custom.php :**

[php]
  
<?php

// Script: connect_custom.php
  
// Author: JR
  
// Date: 20080218
  
// Use: Utilized for custom DB connections to our current database for Fabrik Forms + Joomla 1.5

$hostname="localhost";
  
$mysql_login="thedude";
  
$mysql_password="sumpasswordhere";
  
$database="datablah";

if (!($db = mysql\_pconnect($hostname, $mysql\_login , $mysql_password))){
  
die("Can&#8217;t connect to database server.");
  
}else{
  
if (!(mysql\_select\_db("$database",$db))){
  
die("Can&#8217;t connect to database.");
  
}
  
}
  
?>
  
[/php]