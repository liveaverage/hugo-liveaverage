---
title: 'Check_MK Environmental Monitoring: Room Alert 4E'
author: JR
type: post
date: 2014-08-13T16:32:32+00:00
url: /features/infrastructure-management/check_mk-envrionmental-monitoring-room-alert-4e/
categories:
  - Check_MK
  - Infrastructure Management

---
One of our Network Engineers needed some monitoring (internal temp, external temp, and external humidity) added for an [AVTech Room Alert 4E unit][1], preferably using a check_MK plugin. Couldn&#8217;t find one in any repos or in git, so decided to add this to a growing collection of plugins I&#8217;ve been working on. Another request from the same team: MinuteMan UPS units. I&#8217;ll be adding to my <a title="Check_MK_Bag" href="https://github.com/liveaverage/check_mk_bag" target="_blank">check_mk_bag g</a>it repo periodically with any new, custom check plugins. I&#8217;ll also try to get these on <a href="http://exchange.check-mk.org/" target="_blank">Check_MK&#8217;s Exchange</a>. For now, here&#8217;s some environmental monitors, of course with perfdata output &#8212; gotta have the metrics available for trending!

As typical with check_MK checks, this supports auto-inventory (scan function configured):

<div class="code-embed-wrapper">
  <pre class="language-python code-embed-pre line-numbers" ><code class="language-python code-embed-code">ra4e_sensor_temp_defaultlevels = (28, 32)

def inventory_ra4e_sensor_temp(info):
    inventory = []
    for tempc, tempf, desc in info:
        inventory.append( (desc, "ra4e_sensor_temp_defaultlevels") )
    import pprint ; pprint.pprint(info)
    return inventory

def check_ra4e_sensor_temp(item, params, info):
    for tempc, tempf, desc in info:
        if desc == item:
            warn, crit = params
	    temp = (int(tempc)/100)
	    tempf = (int(tempf)/100)
            if tempc != "" and tempf != "":
                infotext = "%.1f " % tempf + "F (%.1f C)" % temp + " (warn/crit at %.1f/%.1f " % (warn, crit) + "C)"
                perfdata = [ ( "temperature", temp, warn, crit ) ]
                if temp &gt;= crit:
                    return (2, "Temperature is: " + infotext, perfdata)
                elif temp &gt;= warn:
                    return (1, "Temperature is: " + infotext, perfdata)
                else:
                    return (0, "Temperature is: " + infotext, perfdata )
            else:
                return (3, "Sensor is offline")
    return (3, "Sensor not found")

check_info["ra4e_sensor_temp"] = {
    &#039;check_function&#039;:          check_ra4e_sensor_temp,
    &#039;inventory_function&#039;:      inventory_ra4e_sensor_temp,
    &#039;service_description&#039;:     &#039;Temperature %s&#039;,
    &#039;has_perfdata&#039;:            True,
    &#039;snmp_info&#039;:               (
        ".1.3.6.1.4.1.20916.1.6.1.2.1", [
            1, #digital-sen1 - Degrees in Celsius
            2, #digital-sen1 - Degrees in Fahrenheit
            6, #digital-sen1 - Description 
        ],
    ),
    &#039;snmp_scan_function&#039;:      \
         lambda oid: "Room Alert" in oid(".1.3.6.1.2.1.1.1.0"),
    &#039;group&#039;:                   &#039;room_temperature&#039;,</code></pre>
  
  <div class="code-embed-infos">
    <a href="https://github.com/liveaverage/check_mk_bag/blob/master/ra4e_sensor_temp" title="See ra4e_sensor_temp" target="_blank" class="code-embed-name">ra4e_sensor_temp</a> <a href="https://raw.github.com/liveaverage/check_mk_bag//ra4e_sensor_temp" title="Back to ra4e_sensor_temp" class="code-embed-raw" target="_blank">view raw</a>
  </div>
</div>

 [1]: http://avtech.com/Products/Environment_Monitors/Room_Alert_4E.htm "Room Alert 4e"