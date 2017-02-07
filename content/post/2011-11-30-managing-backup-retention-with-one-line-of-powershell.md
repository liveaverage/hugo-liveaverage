---
title: Managing backup retention… with one line of Powershell
author: JR
type: post
date: 2011-11-30T18:09:08+00:00
url: /features/coding/managing-backup-retention-with-one-line-of-powershell/
box_set:
  - default-boxes
box_col_number:
  - 3
box_thumb_type:
  - top_thumbs
categories:
  - Coding

---
Ok, I used four lines, but my needs for retention might be a bit more complex than most. I also spaced each pipeline command, so it looks longer than it should, but readability is important. Additionally, there&#8217;s a good half-page of comments in the script than can safely be ignored, unless you _want_ to know what was going through my mind. Most of these related directly to my desired retention periods.

For testing purposes, the last two &#8220;lines&#8221; only print out the listing of files that would be deleted.

<!--more-->

[shell]
  
Param ($backupDir)

######
  
\# Handle 30-day backup recyclables first (no directory recursion since each backup is in its own directory)
  
\# The current backup schedule performs weekly configuration backups with 1-month retention.
  
######

Get-ChildItem "$backupDir\backup_data" | \`
  
\`
  
Where-Object {(($\_.FullName -match "configuration\_isxpr01\_") -and ( $\_.CreationTime -lt ((Get-Date).AddMonths(-1)) ) )} | \`
  
\`
  
ForEach-Object {\`
                      
Out-File -filepath "$backupDir\log" -inputobject $((Get-Date -format g) + "Removing " + $\_.FullName + " from " + $\_.CreationTime) -append\`
                      
Remove-Item $_.FullName -recurse}

######
  
\# Handle 7-day backup recyclables next (no directory recursion since each backup is in its own directory)
  
\# The current backup schedule performs weekly configuration backups with 1-week retention.
  
\# No "overwrite option available, so this script manages removal
  
######

Get-ChildItem "$backupDir\backup_data" | \`
  
\`
  
Where-Object {(($\_.FullName -match "backup\_isxpr01\_") -and ( $\_.CreationTime -lt ((Get-Date).AddDays(-7)) ) )} | \`
  
\`
  
ForEach-Object {(Remove-Item $_.FullName -recurse)}

######
  
\# Handle daily trend data recyclables next (use recursion parameter).
  
\# The current daily trend export schedule requires 1-month retention for 24-hr trend data.
  
######

Get-ChildItem -include *.zip -recurse -path "$backupDir\trend\_data\_export" | \`
  
\`
  
Where-Object {(($\_.FullName -match "24-hr") -and ( $\_.CreationTime -lt ((Get-Date).AddMonths(-1)) ) )} | \`
  
\`
  
ForEach-Object {($_.FullName)}

######
  
\# Handle monthly trend data recyclables next (use recursion parameter).
  
\# The current monthly trend export schedule requires 5-year retention for 30-day trend data.
  
######

Get-ChildItem -include *.zip -recurse -path "$backupDir\trend\_data\_export" | \`
  
\`
  
Where-Object {(($\_.FullName -match "30-day") -and ( $\_.CreationTime -lt ((Get-Date).AddYears(-5)) ) )} | \`
  
\`
  
ForEach-Object {($_.FullName)}
  
[/shell]