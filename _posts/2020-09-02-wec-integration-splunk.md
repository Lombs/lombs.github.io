---
layout: post
author: Lombs
title: Splunkin' Windows Event Collection
date: 2020-09-02
tags: splunk
date_readable: 2020-09-02
---

# Goal
 
We want to achieve the followings extractions / configs for all WinEventLogs coming from the WEC:
- Store/Remember the WEC host name in a field called host_uf (at index-time)
- Replace host field with value from field ComputerName / Computer (at index-time)
- Store/Remember the original channel name in a field called original_channel (at index-time)
- Make it work for sourcetype WinEventLog as well as XmlWinEventLog
- Extract a new field called bcd_domain from host field by cutting of the trailing domain name (at search-time)
- Get this additional extractions only for Events from WEC and cause interference with WinEventLogs coming in directly from UFs
 
# Universal Forwarder Configuration (WEC)
 
As a requirement for the configurations to apply correctly the inputs on the WEC have to be tagged with the sourcetype WinEventlog:ForwardedEvents oder XmlWinEventlog:ForwardedEvents (if renderXml is set to true). Please do NOT set a host or source attribute in inputs.conf.

### inputs.conf with custom channels

```
[WinEventLog://WEC-Other/LDAP]
current_only = 1
disabled = 0
evt_ad_cache_exp = 1200
evt_ad_cache_exp_neg = 1200
evt_ad_cache_max_entries = 40000
evt_dc_name = 
evt_dns_name = 
evt_resolve_ad_obj = 0
evt_sid_cache_exp = 300
evt_sid_cache_exp_neg = 300
evt_sid_cache_max_entries = 4000
host = CA-APL3001
index = sec_ad_bcd
interval = 60
renderXml = 1
sourcetype = XmlWinEventLog:ForwardedEvents
start_from = oldest
 
[WinEventLog://WEC-Other/SMB]
current_only = 1
disabled = 0
evt_ad_cache_exp = 1200
evt_ad_cache_exp_neg = 1200
evt_ad_cache_max_entries = 40000
evt_dc_name = 
evt_dns_name = 
evt_resolve_ad_obj = 0
evt_sid_cache_exp = 300
evt_sid_cache_exp_neg = 300
evt_sid_cache_max_entries = 4000
host = CA-APL3001
index = sec_ad_bcd
interval = 60
renderXml = 1
sourcetype = XmlWinEventLog:ForwardedEvents
start_from = oldest
 
[WinEventLog://WEC-Other/System]
current_only = 1
disabled = 0
evt_ad_cache_exp = 1200
evt_ad_cache_exp_neg = 1200
evt_ad_cache_max_entries = 40000
evt_dc_name = 
evt_dns_name = 
evt_resolve_ad_obj = 0
evt_sid_cache_exp = 300
evt_sid_cache_exp_neg = 300
evt_sid_cache_max_entries = 4000
host = CA-APL3001
index = sec_ad_bcd
interval = 60
renderXml = 0
sourcetype = WinEventLog:ForwardedEvents
start_from = oldest
whitelist1 = 6005,6006,6008,6009,6013,
whitelist2 = 1001,5136,5137,5138,5139,5141,7034,7035,7036,7040,60006
whitelist9 = 582
```