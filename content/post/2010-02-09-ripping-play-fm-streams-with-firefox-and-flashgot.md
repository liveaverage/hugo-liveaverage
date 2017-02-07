---
title: Ripping Play.FM or SoundCloud Streams with Firefox and FlashGot
author: JR
type: post
date: 2010-02-09T14:44:50+00:00
url: /news/ripping-play-fm-streams-with-firefox-and-flashgot/
syntaxhighlighter_encoded:
  - 1
categories:
  - News

---
<img class="ngg-singlepic ngg-left alignleft" alt="Play.FM Logo" src="http://liveaverage.com/wp-content/gallery/article-images/logo_beta1.jpg" width="150" height="41" />I&#8217;ve recently stumbled across Play.FM, a flash-based music streaming service similar to Pandora but exclusively featuring Electronic and Dance Music. Friendly site with sensational tunes, but many of the songs, particularly those uploaded by independent DJs or artists, are not available for sale or download, making it difficult to listen on portable devices or offline. I tried the usual methods of download/capture for Flash-based music, but nothing was saved to my cache directory, and FreeMusicZilla detected no active streams&#8230; I gave the popular FlashGot Firefox plug-in a try and found it to work _great_. There was one catch: you have to hit play within the Flash player to capture the stream URL, then hit pause for the download to start. I assume Play.FM limits the number of active simultaneous connections from one computer/IP, so you&#8217;ll need to toggle the player so FlashGot detects the MP3 URL, then download the file. Here&#8217;s the quick breakdown on pulling a Play.FM stream:

  1. Fire up your Firefox web browser and navigate to the Play.FM or SoundCloud player streaming the music you&#8217;d like to save.
  2. Start playing the stream until you see the FlashGot icon show up in Firefox status bar (see image below).
  3. [singlepic id=13 w=320 h=240 float=left]Right-click the icon and select the Flash stream you&#8217;d like to download. There is likely only one entry to select.
  4. [singlepic id=16 w=320 h=240 float=right]FlashGot will automatically add this stream download to your &#8220;Downloads,&#8221; window/queue. However, it won&#8217;t start downloading until you pause the stream or close the window.