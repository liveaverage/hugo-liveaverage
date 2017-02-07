---
title: 'Check_MK: Local Checks Grab Bag!'
author: JR
type: post
date: 2014-07-29T15:32:29+00:00
url: /features/coding/check_mk-local-checks-grab-bag/
categories:
  - Coding
  - Infrastructure Management
  - Nagios

---
Some local Check\_MK checks that were created to execute check\_MK local check scripts (Powershell) in 64-bit context, monitor Exchange 2007 health (Storage Group replication status, Log Truncation after backups, etc.), and monitor DNS scavenging on Windows servers:

<div class="code-embed-wrapper">
  <pre class="language-bash code-embed-pre line-numbers" ><code class="language-bash code-embed-code">@echo off
REM Note that SysNative is available on x86 2008, and on x86 2003 with KB942589 applied

set CONSOLE_WIDTH=500
CD %ProgramFiles(x86)%\check_mk\local-64
FOR /R %%X IN ("*") DO ( %WINDIR%\SysNative\windowspowershell\v1.0\powershell.exe -File "%%X")</code></pre>
  
  <div class="code-embed-infos">
    <a href="https://gist.github.com/ff3d31fcc6bde71d8d2f" title="See Execute-Local-64.bat" target="_blank" class="code-embed-name">Execute-Local-64.bat</a> <a href="https://gist.githubusercontent.com/liveaverage/ff3d31fcc6bde71d8d2f/raw/5972e835d811ec92a0378c70bfc9030c72ce9a5f/Execute-Local-64.bat" title="Back to Execute-Local-64.bat" class="code-embed-raw" target="_blank">view raw</a>
  </div>
</div>

<div class="code-embed-wrapper">
  <pre class="language-bash code-embed-pre line-numbers" ><code class="language-bash code-embed-code">$Host.UI.RawUI.BufferSize = New-Object Management.Automation.Host.Size(900,900)

Add-PSSnapin Microsoft.Exchange.Management.PowerShell.Admin

$check_name = "Exchange_SG_LogTrunc"

$OK = 0
$Warn = 1
$Crit = 2
$Unk = 3</code></pre>
  
  <div class="code-embed-infos">
    <a href="https://gist.github.com/c09b3417e4a7fb1e46aa" title="See check_Exchange_SG_LogTruncate.ps1" target="_blank" class="code-embed-name">check_Exchange_SG_LogTruncate.ps1</a> <a href="https://gist.githubusercontent.com/liveaverage/c09b3417e4a7fb1e46aa/raw/c76fe839cd15931d9a45bfa58fe3dbc651afdce6/check_Exchange_SG_LogTruncate.ps1" title="Back to check_Exchange_SG_LogTruncate.ps1" class="code-embed-raw" target="_blank">view raw</a>
  </div>
</div>

<div class="code-embed-wrapper">
  <pre class="language-bash code-embed-pre line-numbers" ><code class="language-bash code-embed-code">$Host.UI.RawUI.BufferSize = New-Object Management.Automation.Host.Size(900,900)

Add-PSSnapin Microsoft.Exchange.Management.PowerShell.Admin



$check_name = "Exchange_SG_CopyStatus"

$OK = 0
$Warn = 1</code></pre>
  
  <div class="code-embed-infos">
    <a href="https://gist.github.com/a042417f78e84a6d0259" title="See check_Exchange_StorageGroups.ps1" target="_blank" class="code-embed-name">check_Exchange_StorageGroups.ps1</a> <a href="https://gist.githubusercontent.com/liveaverage/a042417f78e84a6d0259/raw/3f65e665a279a4a5da222c65004d5057c23e94d0/check_Exchange_StorageGroups.ps1" title="Back to check_Exchange_StorageGroups.ps1" class="code-embed-raw" target="_blank">view raw</a>
  </div>
</div>

<div class="code-embed-wrapper">
  <pre class="language-bash code-embed-pre line-numbers" ><code class="language-bash code-embed-code">$Host.UI.RawUI.BufferSize = New-Object Management.Automation.Host.Size(500,300)

$server = $env:COMPUTERNAME


$global:debug = ""
$global:n_stat = ""
$global:n_stattext = ""

</code></pre>
  
  <div class="code-embed-infos">
    <a href="https://gist.github.com/9114265640b97cabc14e" title="See check_DNS_Scavenge.ps1" target="_blank" class="code-embed-name">check_DNS_Scavenge.ps1</a> <a href="https://gist.githubusercontent.com/liveaverage/9114265640b97cabc14e/raw/5ec2e28f338549dae347741103e0841d08dcf6b8/check_DNS_Scavenge.ps1" title="Back to check_DNS_Scavenge.ps1" class="code-embed-raw" target="_blank">view raw</a>
  </div>
</div>

&nbsp;