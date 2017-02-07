---
title: 'Powershell Script: Windows Event Log Retention and Compression'
author: JR
type: post
date: 2012-11-07T22:12:34+00:00
url: /features/coding/powershell-script-windows-event-log-retention-and-compression/
categories:
  - Coding
  - Infrastructure Management

---
Windows Security event logs fill up fast when you have Directory Service Access Auditing enabled, for <a title="Tracking DNS Deletions" href="http://blogs.technet.com/b/networking/archive/2011/08/17/tracking-dns-record-deletion.aspx" target="_blank">whatever reason</a>. If I want to retain any useful information, I need at least 7 to 14 days of logs to review, in my case, the DNS scavenging process. The built-in &#8216;Archive log when full&#8217; option doesn&#8217;t really help out as much as you&#8217;d think, particularly when I might have 2 or 3 logs (each 300MB) per day. I poked around Windows Server 2008 looking for options to increase the quantity of archive logs, but I couldn&#8217;t find any applicable settings to change. Even if I did find something, how many 300MB logs can I afford to keep around?

I opted for a retention script to compress and managed 30 days worth of logs (not just 30 logs, 30 days worth). This uses native compression (no extension or third-party utilities/binaries required). After compression, my log sizes drop from ~300MB to 29-30MB.  This roughly equates to 90MB per day, assuming 3x300MB raw archive logs are used.

See the included script below. The parameters are well documented, but if there are any questions, just let me know:

[shell]

############################
  
#AUTHOR: JR
  
#CREATED: 20121106
  
#MODIFIED: 20121106
  
############################

####################################################################################
   
#.Synopsis
   
\# Moves archive log files from $logDir to new subdirectory for storage.
   
\# Removes expired log data or backups sets from $logDir according to retention schedule (below).
   
#
   
#.Description
   
\# Checks the user-specified $logDir parameter for log archives
   
\# and removes stale or expired backup/export sets. Default retention is 14 days if not specified.
   
#
   
#.Parameter $logDir
   
\# This directory specifies the location of log files.
   
\# Using this location, the script traverses each subdirectory and determines whether the
   
\# backup has expired and requires deletion.
   
#
   
#.Parameter $logname
   
\# This is the name of the Windows log you&#8217;d like to archive and retain (e.g. Security, System, Application, DNS Server, etc.)
   
#
   
#.Parameter $retention
   
\# This is how long to keep log files in the sub-directory.
   
#
  
####################################################################################

Param
  
(
   
[Parameter(ValueFromPipeline = $True, Mandatory = $True)]
   
[String]
   
[Alias("ld")]
   
$logDir = "C:\Windows\System32\winevt\Logs"
   
,
   
[Parameter(ValueFromPipeline = $True, Mandatory = $True)]
   
[Alias("n")]
   
[String]
   
$logname = "Security"
   
,
   
[Parameter(ValueFromPipeline = $True, Mandatory = $False)]
   
[Alias("pw")]
   
[Int]
   
$retention = 14
  
)

function New-Zip
  
{
   
param([string]$zipfilename)
   
set-content $zipfilename ("PK" + [char]5 + [char]6 + ("$([char]0)" * 18))
   
(dir $zipfilename).IsReadOnly = $false
  
}

function Add-Zip
  
{
   
param([string]$zipfilename,$filetozip)

if(-not (test-path($zipfilename)))
   
{
   
set-content $zipfilename ("PK" + [char]5 + [char]6 + ("$([char]0)" * 18))
   
(dir $zipfilename).IsReadOnly = $false
   
}

$shellApplication = new-object -com shell.application
   
$zipPackage = $shellApplication.NameSpace($zipfilename)

$zipPackage.CopyHere($filetozip)
   
Start-sleep -milliseconds 500

}

function IsFileLocked(
   
[string] $path)
  
{

[bool] $isFileLocked = $true

$file = $null

Try
   
{
   
$file = [IO.File]::Open(
   
$path,
   
[IO.FileMode]::Open,
   
[IO.FileAccess]::Read,
   
[IO.FileShare]::None)

$isFileLocked = $false
   
}
   
Catch [IO.IOException]
   
{
   
If ($_.Exception.Message.EndsWith(
   
"it is being used by another process.") -eq $false)
   
{
   
Throw $_.Exception
   
}
   
}
   
Finally
   
{
   
If ($file -ne $null)
   
{
   
$file.Close()
   
}
   
}

return $isFileLocked
  
}

$archiveDir = "$logDir\Archive-$logname"

\# Verify $logDir has a valid path/value:

if ( ($logDir -eq $null) -or (-not(Test-Path $logDir)))
  
{
  
Write-Output "Missing or invalid parameter. Verify the parameter supplied contains a valid path."
  
exit 3
  
}

else
  
{

if (-not(Test-Path $archiveDir))
  
{
   
New-Item -Path $logDir -Name "Archive-$logname" -ItemType "Directory"
  
}

Out-File -filepath "$archiveDir\maint.txt" -InputObject $(Write-Output ((Get-Date -format g) + "\`tStarting archiving data retention check.")) -Append;

$list = Get-ChildItem "$logDir" | ?{ $\_.Name -like "Archive-$logname*" -and $\_.Mode -notmatch "d" }
  
$list

foreach ($i in $list)
  
{
   
if (-not(Test-Path "$archiveDir\$($i.Name).zip"))
   
{
   
$i.Name
   
Out-File -filepath "$archiveDir\maint.txt" -InputObject $((Write-Output ((Get-Date -format g) + "\`tCompressing " + $\_.Name + " to $archiveDir (ZIP). Last modified: " + $\_.CreationTime))) -Append;
   
New-Zip "$archiveDir\$($i.Name).zip";

$zipname = ((Get-Item "$archiveDir\$($i.Name).zip").FullName);
   
$fil = $i.FullName;

Add-Zip $zipname $fil;

#Wait for file zip before moving on:

While (IsFileLocked $zipname)
   
{
   
Start-sleep -milliseconds 1000;
   
}

#Reset timestamps to those matching the original archive logs; this is important for retention periods:

$zip = Get-Item "$archiveDir\$($i.Name).zip"
   
$zip.LastWriteTime = $i.LastWriteTime
   
$zip.CreationTime = $i.CreationTime
   
}
  
}

######
  
\# Handle retention based on preferred retention schedule.
  
######

Get-ChildItem "$archiveDir\*" -Include *.zip | \`
  
\`
  
Where-Object { ( $_.CreationTime -lt ((Get-Date).AddDays(-($retention))) ) } | \`
  
\`
  
ForEach-Object {\`
   
Out-File -filepath "$archiveDir\maint.txt" -InputObject $((Write-Output ((Get-Date -format g) + "\`tRemoving " + $\_.FullName + " from " + $\_.CreationTime))) -Append;\`
   
Remove-Item $_.FullName }

Out-File -filepath "$archiveDir\maint.txt" -InputObject $(Write-Output ((Get-Date -format g) + "\`tDone.")) -Append;

exit 0
  
}

[/shell]