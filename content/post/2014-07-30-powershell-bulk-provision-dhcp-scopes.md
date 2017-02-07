---
title: 'Powershell: Bulk provision DHCP Scopes'
author: JR
type: post
date: 2014-07-30T21:08:10+00:00
url: /features/coding/powershell-bulk-provision-dhcp-scopes/
categories:
  - Coding
  - Powershell

---
Thought I&#8217;d share a nice wrapper for netsh and dnscmd calls to allow easy, bulk provisioning of new DHCP scopes. It&#8217;s nice being able to provision a ton of these at once by piping the output from Import-CSV!

&nbsp;

<div class="code-embed-wrapper">
  <pre class="language-bash code-embed-pre line-numbers" ><code class="language-bash code-embed-code">############################
#AUTHOR:       JR Morgan
#CREATED:      20120417
#MODIFIED:     20140611
############################

&lt;#
    .Synopsis 
    Adds DHCP Scope to ALL specified DHCP servers. If split-scope is desired
	the script uses IP Math to automatically add the desired exlcude ranges.
	

    .Description 
    Creates a DHCP based on user-provided parameters. For 50/50 split-scope config,
	the ordering of the DHCP Servers determines the upper/lower designation. The first
	DHCP server specified will host upper scope portion, while the second DHCP server will
	host the lower scope portion. Upper/Lower &#039;tags&#039; are added to the description when using
	the Split50 switch.

    .Parameter DhcpServer
        An array of DHCP Server IP names or addresses that will host the new DHCP scopes
        
    .Parameter IPScope
        The desired IP Scope

    .Parameter IPMask
        The desired IP mask for the scope

    .Parameter Description
        A brief scope description. Upper/Lower tags will
		automatically be added if using the Split50 switch
    
    .Parameter Gateway
        The IP address of the router or gateway for this scope (Option 3)
    
	.Parameter Dns
        An array of DNS server IP addresses utilized by scope clients (Option 6).
    
    .Parameter Domain
     	The fully-qualified domain name for scope clients (Option 15)</code></pre>
  
  <div class="code-embed-infos">
    <a href="https://gist.github.com/f61799a360039990087b" title="See Set-Dhcp-Scope.ps1" target="_blank" class="code-embed-name">Set-Dhcp-Scope.ps1</a> <a href="https://gist.githubusercontent.com/liveaverage/f61799a360039990087b/raw/11e25c9381428702a9fa23fd72a7debfc37b6f41/Set-Dhcp-Scope.ps1" title="Back to Set-Dhcp-Scope.ps1" class="code-embed-raw" target="_blank">view raw</a>
  </div>
</div>