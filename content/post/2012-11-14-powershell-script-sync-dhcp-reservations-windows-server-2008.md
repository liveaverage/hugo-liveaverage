---
title: 'Powershell Script: Sync DHCP Reservations (Windows Server 2008)'
author: JR
type: post
date: 2012-11-14T14:11:56+00:00
url: /features/coding/powershell-script-sync-dhcp-reservations-windows-server-2008/
categories:
  - Coding
  - Infrastructure Management

---
If you have redundant Windows 2008 DHCP servers (likely with split-scope configurations), you&#8217;re familiar with the problem of keeping reservations between the servers synchronized. I figured I&#8217;d post a script I created to sync reservations between servers. Synchronization can be 1-to-1 or 1-to-many, depending on your redundant DHCP server configuration. This script can sync with ALL authorized DHCP servers in a domain if needed. Make sure to read the included Powershell help information. This doesn&#8217;t come with any warranty, but I&#8217;d be glad to answer any questions or consider suggestions for improvement. In my environment, this simply runs as a scheduled task on the secondary DHCP server.

Something to note: although it \*should\* work synchronizing to/from Windows Server 2003 servers, I had issues with remote netsh execution (noted in the  script comments below) from W7/2008 systems. I&#8217;ve commented out the offending section that handled 2003 DHCP servers since everyone should, realistically, be running a newer server OS. If there&#8217;s a major need to have 2003 DHCP reservation synchronization, ask and I might be able to spend some more time on the problem. I&#8217;ve since migrated from 2003 to 2008, so there wasn&#8217;t a need to handle these situations.

<a title="Sync-Dhcp-Reservations.ps1" href="https://gist.github.com/liveaverage/522aabcb2593854b70fd" target="_blank">https://gist.github.com/liveaverage/522aabcb2593854b70fd</a>

<div class="code-embed-wrapper">
  <pre class="language-bash code-embed-pre line-numbers" ><code class="language-bash code-embed-code">&lt;#
.Synopsis
Synchronizes DHCP reservations to all or select authorized DHCP servers in a given domain.

.Description
This script utilizes a user-specified source (master) DHCP server and synchronizes
all or some scope reservations between other, authorized DHCP servers for a given domain.
Netsh commands can be generated for manual execution or executed automatically.

.Parameter Source
The source, authorized DHCP server providing the most current scope reservations.

.Parameter SyncAll
Perform synchronization with all authorized domain DHCP servers

.Parameter SyncSelect
Perform synchronization with user-specified authorized domain DHCP servers. Requires destinations.

.Parameter Destination
The destination, authorized DHCP servers where source reservations will configured.
Use &#039;All&#039; switch to synchronize to all authorized DHCP servers in a given domain.

.Parameter Scoperanges
A comma separated list of scoperanges to include in synchronization. Used with -scope switch.
.Parameter Domain
The fully-qualified domain name for the domain hosting destination
DHCP servers (authorized).

.Parameter Domain
Specifies the domain; useful for multi-domain forests. Uses -match (regex)

.Parameter Purge
Purge reservations on all/destination DHCP servers that were removed from the source.

.Parameter Invoke
Invoke the netsh commands directly from this script. By default, this script will only
generate the required netsh commadns.

.Notes
Created by JR Morgan, 20120409

.Example
-------------------------- EXAMPLE 1 --------------------------

Sync-Dhcp-Reservations -Source WinSource.DhcpServername.ms.com -SyncAll -Domain ms.com -Invoke N

Description
-----------
This command generates commands (no invocation) to synchronize reservations between WinSource and WinDest DHCP servers.

-------------------------- EXAMPLE 2 --------------------------

Sync-Dhcp-Reservations -Source WinSource.DhcpServername.ms.com -Destination Server1,Server2,Server3 -Domain ms.com -Invoke N

Description
-----------
This command generates commands (no invocation) to synchronize reservations between WinSource and an array of destination
DHCP servers. There&#039;s no limit to destination servers. This is useful if you do not want to synchronize a source to ALL
authorized DHCP servers for a given domain.

-------------------------- EXAMPLE 3 --------------------------

Sync-Dhcp-Reservations -Source WinSource.DhcpServername.ms.com -SyncAll -Domain ms.com -Scope-Invoke Y

#&gt;

Param
(
[Parameter(
ValueFromPipeline = $True,
ValueFromPipelineByPropertyName = $True,
Mandatory = $True,
HelpMessage="Specify a source DHCP to query for reservations")]
[Alias("s")]
[String]
$Source
,
[Parameter(
ParameterSetName = "DhcpAll",
ValueFromPipeline = $True,
ValueFromPipelineByPropertyName = $True,
HelpMessage="Syncs with all authorized DHCP servers.")]
[switch]
$SyncAll
,
[Parameter(
ParameterSetName = "DhcpDest",
ValueFromPipeline = $True,
ValueFromPipelineByPropertyName = $True,
Mandatory = $True,
HelpMessage="Specify one or more destination DHCP server(s)")]
[Alias("d")]
[String[]]
$SyncSelect
,
[Parameter(
#ParameterSetName = "ScopeIncludes",
ValueFromPipeline = $True,
ValueFromPipelineByPropertyName = $True,
Mandatory = $False,
HelpMessage="Enter one or more scopes to be syncronized from source to destination server(s).")]
[ValidatePattern(&#039;^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$&#039;)]
[Alias("c")]
[String[]]
$Scoperanges
,
[Parameter(
#ParameterSetName = "ScopeExcludes",
ValueFromPipeline = $True,
ValueFromPipelineByPropertyName = $True,
Mandatory = $False,
HelpMessage="Enter one or more scopes to be excluded from synchronization.")]
#[ValidatePattern(&#039;^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$&#039;)]
[Alias("x")]
[String[]]
$ExcludedScopes
,
[Parameter(
ValueFromPipeline = $True,
ValueFromPipelineByPropertyName = $True,
Mandatory = $True,
HelpMessage="Specify the domain hosting authorized DHCP servers you&#039;d like sync")]
[Alias("m")]
[String]
$Domain
,
[Parameter(
ValueFromPipeline = $True,
ValueFromPipelineByPropertyName = $True,
HelpMessage="Specify whether reservations deleted on the source should be removed from destination server(s).")]
[Alias("p")]
[Switch]
$Purge
,
[Parameter(
ValueFromPipeline = $True,
ValueFromPipelineByPropertyName = $True,
HelpMessage="Specify whether the netsh commands should be invoked or only displayed")]
[Alias("i")]
[Switch]
$Invoke
,
[Parameter(
ValueFromPipelineByPropertyName = $True,
HelpMessage="Specify log file path (optional)")]
[Alias("l")]
[string]
$LogPath="$env:temp\Sync-Dhcp-Reservations-Log.txt"
)

function Get-Dhcp-Servers()
{

$dhcps = netsh dhcp show server
$dhcps = $dhcps | ?{ $_ -match $Domain} | %{ $a = $_.Split(" "); $b = $a[1].replace("[",""); $b = $b.replace("]",""); $b}

return $dhcps
}

function Get-Dump ($server)
{

$header= @("proto","server","sip","scope","scoperange","action","opt","ip","mac","name","description","opt2")
$dhcpConfigs = @()

#Backup each server config before proceeding:
#Invoke-Expression "netsh dhcp server \\$server dump &gt; $server-$(Get-Date -uformat "%d").dhcp"

#$dump = "$server-$(Get-Date -uformat "%d").dhcp"

#   $v = Get-WmiObject -ComputerName $server -Authentication PacketPrivacy -Impersonation Impersonate Win32_OperatingSystem

#There&#039;s a problem with remote netsh execution from W7/2008/newer to 2003/XP DHCP.
#Solved with a call to PsExec, but this condition is tested, then the presence of PsExec
#is tested.

$dump = Invoke-Expression "netsh dhcp server \\$server dump"

#    if ($v.version -gt 6)
#    {
#        $dump = Invoke-Expression "netsh dhcp server \\$server dump"
#    }
#    elseif (($v.version -lt 6) -and (Test-Path "PsExec.exe"))
#    {
#
#        $dump = Invoke-Expression "./PsExec.exe \\$server netsh dhcp server \\$server dump"
#        Start-Sleep -Seconds 20
#    }
#    else
#    {
#        throw "Problem determining destination DHCP server version for netsh command execution.`r`nYou may be missing PsExec.exe from the script directory."
#    }

$raw = $dump | Where {($_ -notmatch "#") -and ($_ -match "reservedip")} |
ForEach-Object { $_ }

if ($raw -ne $null)
{
#Import the content into a CSV array object:
$csvdump = $raw | ConvertFrom-Csv -Header $header -delimiter " "
}
else
{
throw "Problem retrieving configuration from $server.`r`nYou may not have appropriate permissions or you may be querying an older server version."
}

return $csvdump

}

function Compare-Dump ($sourcedump, $destdump)
{

Compare-Object -ReferenceObject $sourcedump -DifferenceObject $destdump -Property scoperange,action,opt,ip,mac -PassThru |
#?{ ($_.name -notmatch "unused")} |
Select -Property sip,name,scoperange,action,opt,ip,mac,@{Name="Configured on";Expression={if ($_.SideIndicator -eq "=&gt;"){ "Destination" } elseif ($_.SideIndicator -eq "&lt;="){ $source }}},description

}

function Set-ClearVars()
{
Remove-Variable [a..z]* -Scope Global
Remove-Variable [1..9]* -Scope Global
}

function Get-Destination-Sys()
{

$d = Get-Dhcp-Servers | ?{ ($_ -notmatch $source) -and ($_ -notmatch "gruprintpr01")}

if ($SyncAll)
{
return $d
}
elseif (($SyncSelect).count -ge 1)
{
foreach ($sys in $SyncSelect)
{
$dr += $d | ?{ $_ -match $sys }
}

return $dr
}
}

############## Main Block #####################

#Get source reservation config; this will be used frequently:
$sd = Get-Dump $source
$log = $null
$sourceIp = (Test-Connection -Count 1 $source).IPV4Address.IPAddressToString

#Debug:

$start = $(Get-Date -uformat "%Y%m%d%H%M%S")

$log += "(DEBUG) $start`r`n"
$log += "(DEBUG) User-provided source DHCP server:`t`t`t$source`r`n"
$log += "(DEBUG) User-provided domain name for DHCP sync:`t`t$Domain`r`n"
$log += "(DEBUG) User-provided SyncSelect server count:`t`t`t$(($SyncSelect).count)`r`n"
$log += "(DEBUG) User-provided SyncSelect server(s):`t`t`t$SyncSelect`r`n"
$log += "(DEBUG) Listing of all Dhcp-Servers detected for:`t`t$(Get-Dhcp-Servers)`r`n"
$log += "(DEBUG) Listing of desination server:`t`t`t`t$(Get-Destination-Sys)`r`n"

foreach ($s in Get-Destination-Sys)
{
Set-ClearVars
$nsh_dis = @()
$nsh_add = @()
$c = $null
$cr = $null
$xs = $null

#Get destination server IP (for comparisons):
$destIp = (Test-Connection -Count 1 $s).IPV4Address.IPAddressToString

$log += "`r`n`r`n(DEBUG) ######## Syncing $s ######################################`r`n`r`n"

#Output for this should be null or empty if dhcp is synchronized:

$c = Compare-Dump $sd (Get-Dump $s)

#$log += "(DEBUG) Initial dump comparision for $source and $($s):`r`n"
$c | ft -auto

#Parse the dump info to include only relevant scopes:

if (($Scoperanges).count -ge 1)
{
$log += "(DEBUG) User-provided Scoperange count:`t`t`t`t$($Scoperanges.count)"
$log += "(DEBUG) User-provided Scopes to be synchronized:`t`t$Scoperanges"

foreach ($sc in $Scoperanges)
{
#Remove discrepancies for unspecified scoperanges (on source and dest servers):
$log += "(DEBUG) Filtering diff set by user-provided scope $sc"
$cr += $c | ?{$_.scoperange -eq $sc}
}

$log += "(DEBUG) Revised Scoperange-specific diffs:`r`n"
$cr | ft -auto
$c = $cr

$log += "(DEBUG) Diffs after reassignment to initial dump comp array:`r`n"
$c | ft -auto
$log += "(DEBUG) Count of diffs after scoperange specifics:`t`t`t$($c.count)`r`n"
}

if (($ExcludedScopes).count -ge 1)
{
foreach ($xs in $ExcludedScopes)
{
#Remove discrepancies for excluded scoperanges (on source and dest servers):
$log += "(DEBUG) Filtering diff set by user-provided excluded scope $xs"
"Removing $($c | ?{$_.scoperange -notmatch $xs})"
$c = $c | ?{$_.scoperange -notmatch $xs}
}

#$log += "(DEBUG) Revised Scoperange-exclusion diffs:`r`n"
#$cr | ft -auto
#$c = $cr

$log += "(DEBUG) Diffs after reassignment to initial dump comp array:`r`n"
$c | ft -auto
$log += "(DEBUG) Count of diffs after scoperange specifics:`t`t`t$($c.count)`r`n"
}

#Handle discrepancies for reservations that require removal/addition from destination servers:

foreach ($r in $c)
{
#If the serverIP/name matches the destination server, that means the records no longer exists or has changed on the
#source server; removed these reservations since we&#039;re syncing from a SINGLE master DHCP sys:
if (($r.sip -match $s) -or ($r.sip -eq $destIp))
{
$nsh_dis += "netsh dhcp server \\$s scope $($r.scoperange) delete reservedip $($r.ip) $($r.mac)`r`n"
}

#If the serverIP/name matches the source server (master), that means it&#039;s missing on the destination server;
#add these reservations since we&#039;re syncing to destination servers FROM the master.
if (($r.sip -match $source) -or ($r.sip -eq $sourceIP))
{
#$desc = ($r.mac + " " + (Convert-DNStoCN $Name))
$nsh_add += "netsh dhcp server \\$s scope $($r.scoperange) add reservedip $($r.ip) $($r.mac) `"$($r.name)`" `"$($r.description)`" BOTH`r`n"
}
}
$log += "`r`n(DEBUG) Reservations being removed from destination $s :`r`n"
$log += $nsh_dis
$nsh_dis | Out-File "netsh_cmd_dis_res.txt"
$log += "`r`n(DEBUG) Reservations being added to destination $s :`r`n"
$log += $nsh_add
$nsh_add | Out-File "netsh_cmd_add_res.txt"

if ($Invoke)
{
$log += "`r`n(DEBUG) Received Invoke Switch; Executing reservation operations on $s :`r`n"
foreach ($cm in $nsh_dis)
{
Invoke-Expression $cm -ErrorVariable err
}
foreach ($ca in $nsh_add)
{
Invoke-Expression $ca -ErrorVariable err
}

if ($err.count -ge 1)
{
$log += "Errors: `r`n$err `r`n"
}
else
{
$log += "No errors encountered; moving on`r`n"
}
$log += "`r`n(DEBUG) Finished invoking/syncing reservations to $s :`r`n"
}

#Set-ClearVars
#$nsh_add = $null
#$nsh_dis = $null

#Remove-Variable $nsh_add
#Remove-Variable $nsh_dis
$end = $(Get-Date -uformat "%Y%m%d%H%M%S")

$log += "`r`n`r`n(DEBUG) ######## $end - End of sync for $s ######################################`r`n`r`n"
}

Out-File -Encoding ASCII -InputObject $log -FilePath "dhcpSync.log" -Append
$log</code></pre>
  
  <div class="code-embed-infos">
    <a href="https://gist.github.com/522aabcb2593854b70fd" title="See Sync-Dhcp-Reservations.ps1" target="_blank" class="code-embed-name">Sync-Dhcp-Reservations.ps1</a> <a href="https://gist.githubusercontent.com/liveaverage/522aabcb2593854b70fd/raw/1db0569e101ae3d6e8dc30205649073cbdf0d88e/Sync-Dhcp-Reservations.ps1" title="Back to Sync-Dhcp-Reservations.ps1" class="code-embed-raw" target="_blank">view raw</a>
  </div>
</div>