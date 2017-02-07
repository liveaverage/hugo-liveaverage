---
title: Check_MK DFS Backlog Monitoring
author: JR
type: post
date: 2014-02-12T14:29:23+00:00
url: /news/check_mk-dfs-backlog-monitoring/
categories:
  - Infrastructure Management
  - News

---
<figure id="attachment_3968" style="width: 715px" class="wp-caption alignright">[<img class="size-large wp-image-3968 " alt="Screenshot of inventoried replicated folders and their corresponding backlog counts" src="http://liveaverage.com/wp-content/themes/mimbo2.2/images/20131111_CheckMK_DFSRBacklog-1024x195.jpg" width="715" height="136" srcset="http://liveaverage.com/wp-content/themes/mimbo2.2/images/20131111_CheckMK_DFSRBacklog-300x57.jpg 300w, http://liveaverage.com/wp-content/themes/mimbo2.2/images/20131111_CheckMK_DFSRBacklog-1024x195.jpg 1024w, http://liveaverage.com/wp-content/themes/mimbo2.2/images/20131111_CheckMK_DFSRBacklog.jpg 1232w" sizes="(max-width: 715px) 100vw, 715px" />][1]<figcaption class="wp-caption-text">Inventoried replicated folders and their corresponding backlog counts</figcaption></figure> 

Before you even get started, make sure your Powershell execution policy is set to **RemoteSigned **for your standard AND x86 Powershell console; Check_MK will generally execute PS scripts from the x86 console, so it&#8217;s critical to set the policy for both:

  1. Start > Accessories > Windows PowerShell
  2. Right-click &#8216;Windows PowerShell (x86)&#8217;, select &#8216;Run As Administrator&#8217;
  3. Execute: _Set-Execution Policy RemoteSigned_
  4. Repeat the same steps above, but for the standard &#8216;Windows PowerShell&#8217; console.

<span style="line-height: 1.6em;">Next you&#8217;ll need to configure the Check_MK_Agent Service along with WMI and Component Services Security settings:</span>

  1. Configure the &#8216;Check\_MK\_Agent&#8217; service to utilize a domain user account (non-administrative). Make certain the &#8216;Log On As&#8217; user is a member of a **security group** configured for WMI/COM access. Restart the service when modifying &#8216;Log On As&#8217; account
  2. <span style="line-height: 1.6em;">Each DFS Replicated folder has two or more Sending or Receiving members. These members can be determined by examining the replication group connections in the DFS management console.</span>
  3. <span style="line-height: 1.6em;">Perform the following security/access edits for <em>all</em> DFS Sending/Receiving members of a replication group (it will fail if you don&#8217;t do this for every replication member)</span> 
      1. <span style="line-height: 1.6em;">Update WMI Security</span> 
          1. <span style="line-height: 1.6em;">Start > Run > wmimgmt.msc</span>
          2. <span style="line-height: 1.6em;">Right-click &#8216;WMI Control&#8217; > Select &#8216;Properties&#8217;</span>
          3. <span style="line-height: 1.6em;">Select &#8216;Security&#8217; Tab</span>
          4. Navigate to the proper ROOT\MicrosoftDfs namespace and click the &#8216;Security&#8217; button
          5. Click &#8216;Adavanced&#8217;, Click &#8216;Add&#8217; and enter the user or security group used for WMI access
          6. Select the following &#8216;Allow&#8217; permissions: 
              1. Execute Method
              2. Enable Account
              3. Remote Enable
              4. Read Security
      2. Update Component Services Security 
          1. Start > Administrative Tools > Component Services
          2. From the Console Root, navigate to &#8216;Component Services&#8217; > &#8216;Computers&#8217; > &#8216;My Computer&#8217;
          3. Right-click &#8216;My Computer&#8217;,  select &#8216;Properties&#8217;
          4. Select the &#8216;COM Security&#8217; tab.
          5. For both &#8216;Access Permissions&#8217; and &#8216;Launch and Activation Permissions&#8217;, click &#8216;Edit Limits&#8217; and add the user/group. &#8216;Allow&#8217; all available permissions. Click OK and close all open windows.
  4. <span style="line-height: 1.6em;">Verify you can successfully poll DFS replication group counts by running a Powershell terminal as the &#8216;Log On As&#8217; account you specified for the check_mk_agent service. Execute the script below. If you&#8217;re receiving backlog counts for each and every RG connection then everything is configured and you&#8217;re ready to copy the PS script to the </span><em style="line-height: 1.6em;">check_mk/local </em><span style="line-height: 1.6em;">directory</span><em style="line-height: 1.6em;">.</em>
  5. <span style="line-height: 1.6em;">An additional note: if you have a TON of replication groups (like I do), then I highly suggest downloading the Check_MK agent Innovation release and tweaking the local check timeout & cache settings. This will help, but likely not solve, issues with backlog check timeouts. An alternative is using a scheduled task for backlog counts, output to a status file, and use a simple &#8216;Get-Content&#8217; in PS to output the status file contents when requested by check_mk</span>

<a title="Gist - check_DFS_Backlog.ps1" href="https://gist.github.com/liveaverage/6324046" target="_blank">https://gist.github.com/liveaverage/6324046</a>

<div class="code-embed-wrapper">
  <pre class="language-bash code-embed-pre line-numbers" ><code class="language-bash code-embed-code">$computer = [System.Net.Dns]::GetHostName()
$Computer = [System.Net.Dns]::GetHostName()

# Fix issue with console text wrap:
$Host.UI.RawUI.BufferSize = New-Object Management.Automation.Host.Size (500, 300)

$OK = 0
$Warn = 1
$Crit = 2
$Unk = 3

#Warning/Critical Backlog [File] Counts:
$w_count = 350
$c_count = 700

[string]$RGName = ""
[string]$RFName = ""

$DebugPreference = "SilentlyContinue"
$ErrorActionPreference = "SilentlyContinue"

#region DFSQuery

### These thresholds are irrelevant for the local Check_MK thresholds (native to Steve Grinker&#039;s script):
[int]$WarningThreshold = 50
[int]$ErrorThreshold = 500

Function PingCheck
{
    Param</code></pre>
  
  <div class="code-embed-infos">
    <a href="https://gist.github.com/6324046" title="See check_DFS_backlog.ps1" target="_blank" class="code-embed-name">check_DFS_backlog.ps1</a> <a href="https://gist.githubusercontent.com/liveaverage/6324046/raw/b99f1ee47125a93d431403e1789c7340acfc14c5/check_DFS_backlog.ps1" title="Back to check_DFS_backlog.ps1" class="code-embed-raw" target="_blank">view raw</a>
  </div>
</div>

 [1]: http://liveaverage.com/wp-content/themes/mimbo2.2/images/20131111_CheckMK_DFSRBacklog.jpg