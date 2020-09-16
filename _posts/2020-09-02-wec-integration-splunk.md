---
layout: post
author: Lombs
title: Splunkin' Windows Event Collection
date: 2020-09-02
tags: splunk wec winevenlog windows wef
date_readable: 2020-09-02
---

# Goal
 
We want to achieve the following extractions / configs for all WinEventLogs coming from the Windows Event Collection (WEC) - Server:
- Store/Remember the WEC host name in a field called host_wec (at index-time)
- Replace host field with value from field ComputerName / Computer (at index-time)
- Store/Remember the original channel name in a field called original_channel (at index-time)
- Make it work for sourcetype WinEventLog as well as XmlWinEventLog
- Extract a new field called ad_domain from host field by cutting of the trailing domain name (at search-time)
- Get this additional extractions only for Events from WEC and cause interference with WinEventLogs coming in directly from UFs
 
# Configuration
## Universal Forwarder Configuration (WEC)
 
As a requirement for the configurations to apply correctly the inputs on the WEC have to be tagged with the sourcetype WinEventlog:ForwardedEvents oder XmlWinEventlog:ForwardedEvents (if renderXml is set to true). Please do NOT set a host or source attribute in inputs.conf.

### inputs.conf with custom channels

inputs.conf
```conf
[WinEventLog://Custom-Channel/LDAP]
current_only = 1
disabled = 0
index = domain_controllers
interval = 60
renderXml = 1
sourcetype = XmlWinEventLog:ForwardedEvents
start_from = oldest
 
[WinEventLog://Custom-Channel/SMB]
current_only = 1
disabled = 0
index = domain_controllers
interval = 60
renderXml = 1
sourcetype = XmlWinEventLog:ForwardedEvents
start_from = oldest
 
[WinEventLog://Custom-Channel/System]
current_only = 1
disabled = 0
index = domain_controllers
interval = 60
renderXml = 0
sourcetype = WinEventLog:ForwardedEvents
start_from = oldest
whitelist1 = 6005,6006,6008,6009,6013,
whitelist2 = 1001,5136,5137,5138,5139,5141,7034,7035,7036,7040,60006
whitelist9 = 582
```


## Indexer Configuration

Most of the extractions have to be applied at index-time when data first reaches a full splunk instance, the indexer has to be the first place to do some changes. On the indexer we need to enhance the already existing Splunk Windows TA (Splunk_TA_windows) by adding configs in the local folder. Currentl the TA runs with version 8.0.0 and we add the following config files:

…/Splunk_TA_windows_idx/local/props.conf
```conf
############################################################################
## WEC WinEventLog Config at index-time
############################################################################

## Host override for WinEventLog events collected using WEF
#[host::WEC-*]
#TRANSFORMS-change_host_for_windows_wef = WinEventRememberWECHost, WinEventHostOverride
#TRANSFORMS-change_xml_host_for_windows_wef = WinEventRememberWECHost, WinEventXmlHostOverride

# Hint: Data is collected by Windows Event Forwarding and sent to Splunk by one UF with this sourcetype:
[WinEventLog:ForwardedEvents]
TRANSFORMS-change_host_for_windows_wef = WinEventRememberHost, WinEventHostOverride

[XmlWinEventLog:ForwardedEvents]
TRANSFORMS-change_xml_host_for_windows_wef = WinEventRememberHost, WinEventXmlHostOverride

############################################################################
## Custom Event Cleanup (Trimming and Storing Channel Names)
############################################################################
[source::WinEventLog:*]
SEDCMD-trim_event-text = s/This event is generated[\S\s\r\n]+$/<Trimmed>/g

# These stanzas apply on sources AND sourcetypes. Copied from Windows TA
[(?::){0}WinEventLog:*]
TRANSFORMS-1-SaveOrigChannel = WinEventSetOrigChannelName

[(?::){0}XmlWinEventLog:*]
TRANSFORMS-1-XmlSaveOrigChannel = WinEventSetOrigChannelName

[source::WinEventLog:ForwardedEvents]
SEDCMD-remove_ffff = s/::ffff://g
SEDCMD-cleansrcipxml = s/<Data Name='IpAddress'>(\:\:1|127\.0\.0\.1)<\/Data>/<Data Name='IpAddress'><\/Data>/
SEDCMD-cleansrcportxml=s/<Data Name='IpPort'>0<\/Data>/<Data Name='IpPort'><\/Data>/
SEDCMD-clean_rendering_info_block = s/<RenderingInfo Culture='.*'>(?s)(.*)<\/RenderingInfo>//

############################################################################
## Event Cleanup Best Practice by Splunk https://docs.splunk.com/Documentation/WindowsAddOn/8.0.0/User/Configuration
## Does not apply for ForwardedEvents from WECs sourcetype, only for events coming in directly from UFs
############################################################################

[source::WinEventLog:Security]
SEDCMD-windows_security_event_formater = s/(?m)(^\s+[^:]+\:)\s+-?$/\1/g
SEDCMD-windows_security_event_formater_null_sid_id = s/(?m)(:)(\s+NULL SID)$/\1/g s/(?m)(ID:)(\s+0x0)$/\1/g
SEDCMD-cleansrcip = s/(Source Network Address:    (\:\:1|127\.0\.0\.1))/Source Network Address:/
SEDCMD-cleansrcport = s/(Source Port:\s*0)/Source Port:/
SEDCMD-remove_ffff = s/::ffff://g
SEDCMD-clean_info_text_from_winsecurity_events_certificate_information = s/Certificate information is only[\S\s\r\n]+$//g
SEDCMD-clean_info_text_from_winsecurity_events_token_elevation_type = s/Token Elevation Type indicates[\S\s\r\n]+$//g
#SEDCMD-clean_info_text_from_winsecurity_events_this_event = s/This event is generated[\S\s\r\n]+$//g

#For XmlWinEventLog:Security
SEDCMD-cleanxmlsrcport = s/<Data Name='IpPort'>0<\/Data>/<Data Name='IpPort'><\/Data>/
SEDCMD-cleanxmlsrcip = s/<Data Name='IpAddress'>(\:\:1|127\.0\.0\.1)<\/Data>/<Data Name='IpAddress'><\/Data>/

[WMI:WinEventLog:System]
SEDCMD-clean_info_text_from_winsystem_events_this_event = s/This event is generated[\S\s\r\n]+$//g

[WMI:WinEventLog:Security]
SEDCMD-windows_security_event_formater = s/(?m)(^\s+[^:]+\:)\s+-?$/\1/g
SEDCMD-windows_security_event_formater_null_sid_id = s/(?m)(:)(\s+NULL SID)$/\1/g s/(?m)(ID:)(\s+0x0)$/\1/g
SEDCMD-cleansrcip = s/(Source Network Address:    (\:\:1|127\.0\.0\.1))/Source Network Address:/
SEDCMD-cleansrcport = s/(Source Port:\s*0)/Source Port:/
SEDCMD-remove_ffff = s/::ffff://g
SEDCMD-clean_info_text_from_winsecurity_events_certificate_information = s/Certificate information is only[\S\s\r\n]+$//g
SEDCMD-clean_info_text_from_winsecurity_events_token_elevation_type = s/Token Elevation Type indicates[\S\s\r\n]+$//g
SEDCMD-clean_info_text_from_winsecurity_events_this_event = s/This event is generated[\S\s\r\n]+$//g</li>
```

…/Splunk_TA_windows/local/transforms.conf
```conf 
## Overriding host to identify system from which events are generated
[WinEventHostOverride]
DEST_KEY = MetaData:Host
REGEX = (?m)ComputerName=(.*)?\b
FORMAT = host::$1

[WinEventXmlHostOverride]
DEST_KEY = MetaData:Host
REGEX = <Computer>(.*).*?<\/Computer>
FORMAT = host::$1

[WinEventRememberHost]
SOURCE_KEY = MetaData:Host
REGEX = host::(.+)
FORMAT = host_WEC::$1
WRITE_META = true

[WinEventSetOrigChannelName]
REGEX = WinEventLog:(.*)
SOURCE_KEY = MetaData:Source
FORMAT = original_channel::$1
WRITE_META = true

[WinEventRememberWECHost]
SOURCE_KEY = MetaData:Host
REGEX = host::WEC-(.+)
FORMAT = host_WEC::$1
WRITE_META = true
```

…/Splunk_TA_windows_idx/local/fields.conf
```conf
[host_WEC]
INDEXED = true

[original_channel]
INDEXED = true
```

Apply the new cluster-bundle to the indexer cluster members:
```bash
$ splunk show cluster-bundle-status
$ splunk apply cluster-bundle
```

## Search Head Configuration

The work is done in the indexers and now it's time to switch to the Search Head and verify if the extractions work correctly. But before we start to query the windows events we first have to add the extraction for the ad_domain field on the SH at search-time.

…/my_app/local/props.conf
```conf
[WinEventLog]
EXTRACT-ad_domain = \.(?<ad_domain>.*)$ in host

[XmlWinEventLog]
EXTRACT-ad_domain = \.(?<ad_domain>.*)$ in host
```

…/my_app/local/fields.conf
```conf
[ad_domain]
INDEXED_VALUE = false
```

After a quick …/debug/refresh call to reload the config, we can jump to the search bar and start searching our events. 

# Troubleshooting

Here are some queries to verify the results:

Verify extractions for events indexed on indexer cluster (without ad_domain, as it is not an indexed field)
```
| tstats count as Eventcount, distinct_count(host) as Hostcount, values(source) as Sources, values(original_channel) as "Original Channels"  where index=domain_controllers_* splunk_server="indexer_system" sourcetype=*WinEventLog by sourcetype, host_WEC, index
index=domain_controllers_* splunk_server="indexer_system" sourcetype=*WinEventLog | stats count, distinct_count(host), values(source), values(original_channel) by sourcetype, host_WEC
```

Which sources are leaving the UF on the WEC?
```
index=_internal host=WEC-host group=per_source_thruput | stats count by series
```

Which sourcetypes are leaving the UF on the WEC?
```
index=_internal host=WEC-host group=per_sourcetype_thruput | stats count by series
```

# Lessons Learned / ToDo's
- SEDCMD runs at index-time (https://docs.splunk.com/Documentation/Splunk/8.0.5/Data/Anonymizedata)
- Be aware of execution order of TRANSFORMS-* attributes of stanzas in props.conf
- Using a WEC is not Splunk Best Practice
- Splunk will indeed not process props again after changing the sourcetype (https://community.splunk.com/t5/Getting-Data-In/What-are-the-precedence-of-stanza-and-option-in-props-conf/td-p/303042)
- ad_domain can be turned into a indexed extraction, then it can also be used within tstats commands etc.
		
# Sources

- https://community.splunk.com/t5/All-Apps-and-Add-ons/WinEventLog-ForwardedEvents-override/td-p/129032#answer-330822
- https://community.splunk.com/t5/Getting-Data-In/Windows-Event-Forwarding-custom-channels-renaming-sources-adding/m-p/516977#M87495
- https://wiki.splunk.com/Community:HowIndexingWorks
- https://docs.splunk.com/Documentation/Splunk/8.0.5/Indexer/Indextimeversussearchtime

[back](./)