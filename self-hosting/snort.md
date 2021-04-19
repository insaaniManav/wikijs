---
title: Snort IDS 
description: An intrusion detection system for your home or enterprise network 
published: 1
date: 2021-04-18T16:19:10.065Z
tags: self-hosting
editor: markdown
dateCreated: 2021-04-18T16:12:39.967Z
---

# Intial steps

use opkg to install and install snort from the repos

```shell
opkg update
opkg install snort
```

# Configuration 
The configuration files for snort are saved in /etc/snort

Here is a sample config file with most basic options higlighted that you can change or tweak to make snort behave a certain way

```squidconf
---------------------------------------------------------------------------
-- Snort++ configuration
---------------------------------------------------------------------------

-- there are over 200 modules available to tune your policy.
-- many can be used with defaults w/o any explicit configuration.
-- use this conf as a template for your specific configuration.

-- 1. configure defaults
-- 2. configure inspection
-- 3. configure bindings
-- 4. configure performance
-- 5. configure detection
-- 6. configure filters
-- 7. configure outputs
-- 8. configure tweaks

---------------------------------------------------------------------------
-- 1. configure defaults
---------------------------------------------------------------------------

-- HOME_NET and EXTERNAL_NET must be set now
-- setup the network addresses you are protecting
HOME_NET = 'any'

-- set up the external network addresses.
-- (leave as "any" in most situations)
EXTERNAL_NET = 'any'

include 'snort_defaults.lua'
include 'file_magic.lua'

---------------------------------------------------------------------------
-- 2. configure inspection
---------------------------------------------------------------------------

-- mod = { } uses internal defaults
-- you can see them with snort --help-module mod

-- mod = default_mod uses external defaults
-- you can see them in snort_defaults.lua

-- the following are quite capable with defaults:

stream = { }
stream_ip = { }
stream_icmp = { }
stream_tcp = { }
stream_udp = { }
stream_user = { }
stream_file = { }

arp_spoof = { }
back_orifice = { }
dnp3 = { }
dns = { }
http_inspect = { }
http2_inspect = { }
imap = { }
modbus = { }
netflow = {}
normalizer = { }
pop = { }
rpc_decode = { }
sip = { }
ssh = { }
ssl = { }
telnet = { }

dce_smb = { }
dce_tcp = { }
dce_udp = { }
dce_http_proxy = { }
dce_http_server = { }

-- see snort_defaults.lua for default_*
gtp_inspect = default_gtp
port_scan = default_med_port_scan
smtp = default_smtp

ftp_server = default_ftp_server
ftp_client = { }
ftp_data = { }

-- see file_magic.lua for file id rules
file_id = { file_rules = file_magic }

-- the following require additional configuration to be fully effective:

appid =
{
    -- appid requires this to use appids in rules
    --app_detector_dir = 'directory to load appid detectors from'
}

--[[
reputation =
{
    -- configure one or both of these, then uncomment reputation
    --blacklist = 'blacklist file name with ip lists'
    --whitelist = 'whitelist file name with ip lists'
}
--]]

default_variables = {
    nets = {
        HOME_NET = HOME_NET,
        EXTERNAL_NET = EXTERNAL_NET,
        DNS_SERVERS = DNS_SERVERS,
        FTP_SERVERS = FTP_SERVERS,
        HTTP_SERVERS = HTTP_SERVERS,
        SIP_SERVERS = SIP_SERVERS,
        SMTP_SERVERS = SMTP_SERVERS,
        SQL_SERVERS = SQL_SERVERS,
        SSH_SERVERS = SSH_SERVERS,
        TELNET_SERVERS = TELNET_SERVERS,
        AIM_SERVERS = AIM_SERVERS,
},
    paths = {
        RULE_PATH = RULE_PATH,
        BUILTIN_RULE_PATH = BUILTIN_RULE_PATH,
        PLUGIN_RULE_PATH = PLUGIN_RULE_PATH,
        WHITE_LIST_PATH = WHITE_LIST_PATH,
        BLACK_LIST_PATH = BLACK_LIST_PATH,
    },
    ports = {
        FTP_PORTS = FTP_PORTS,
        HTTP_PORTS = HTTP_PORTS,
        MAIL_PORTS = MAIL_PORTS,
        ORACLE_PORTS = ORACLE_PORTS,
        SIP_PORTS = SIP_PORTS,
        SSH_PORTS = SSH_PORTS,
        FILE_DATA_PORTS = FILE_DATA_PORTS,
    }

}



---------------------------------------------------------------------------
-- 3. configure bindings
---------------------------------------------------------------------------

wizard = default_wizard

binder =
{
    -- port bindings required for protocols without wizard support
    { when = { proto = 'udp', ports = '53', role='server' },  use = { type = 'dns' } },
    { when = { proto = 'tcp', ports = '53', role='server' },  use = { type = 'dns' } },
    { when = { proto = 'tcp', ports = '111', role='server' }, use = { type = 'rpc_decode' } },
    { when = { proto = 'tcp', ports = '502', role='server' }, use = { type = 'modbus' } },
    { when = { proto = 'tcp', ports = '2123 2152 3386', role='server' }, use = { type = 'gtp_inspect' } },

    { when = { proto = 'tcp', service = 'dcerpc' }, use = { type = 'dce_tcp' } },
    { when = { proto = 'udp', service = 'dcerpc' }, use = { type = 'dce_udp' } },
    { when = { proto = 'udp', service = 'netflow' }, use = { type = 'netflow' } },

    { when = { service = 'netbios-ssn' },      use = { type = 'dce_smb' } },
    { when = { service = 'dce_http_server' },  use = { type = 'dce_http_server' } },
    { when = { service = 'dce_http_proxy' },   use = { type = 'dce_http_proxy' } },

    { when = { service = 'dnp3' },             use = { type = 'dnp3' } },
    { when = { service = 'dns' },              use = { type = 'dns' } },
    { when = { service = 'ftp' },              use = { type = 'ftp_server' } },
    { when = { service = 'ftp-data' },         use = { type = 'ftp_data' } },
    { when = { service = 'gtp' },              use = { type = 'gtp_inspect' } },
    { when = { service = 'imap' },             use = { type = 'imap' } },
    { when = { service = 'http' },             use = { type = 'http_inspect' } },
    { when = { service = 'http2' },            use = { type = 'http2_inspect' } },
    { when = { service = 'modbus' },           use = { type = 'modbus' } },
    { when = { service = 'pop3' },             use = { type = 'pop' } },
    { when = { service = 'ssh' },              use = { type = 'ssh' } },
    { when = { service = 'sip' },              use = { type = 'sip' } },
    { when = { service = 'smtp' },             use = { type = 'smtp' } },
    { when = { service = 'ssl' },              use = { type = 'ssl' } },
    { when = { service = 'sunrpc' },           use = { type = 'rpc_decode' } },
    { when = { service = 'telnet' },           use = { type = 'telnet' } },

    { use = { type = 'wizard' } }
}

---------------------------------------------------------------------------
-- 4. configure performance
---------------------------------------------------------------------------

-- use latency to monitor / enforce packet andRules rule thresholds
--latency = { }

-- use these to capture perf data for analysis and tuning
--profiler = { }
--perf_monitor = { }

---------------------------------------------------------------------------
-- 5. configure detection
---------------------------------------------------------------------------

references = default_references
classifications = default_classifications

ips =
{
    -- use this to enable decoder and inspector alerts
    --enable_builtin_rules = true,

    -- use include for rules files; be sure to set your path
    -- note that rules files can include other rules files
    include = './rules/snort3-community.rules',
    variables = default_variables
}

rewrite = { }

-- use these to configure additional rule actions
-- react = { }
-- reject = { }

-- use this to enable payload injection utility
-- payload_injector = { }

---------------------------------------------------------------------------
-- 6. configure filters
---------------------------------------------------------------------------

-- below are examples of filters
-- each table is a list of records

--[[
suppress =
{
    -- don't want to any of see these
    { gid = 1, sid = 1 },

    -- don't want to see these for a given server
    { gid = 1, sid = 2, track = 'by_dst', ip = '1.2.3.4' },
}
--]]

--[[
event_filter =
{
    -- reduce the number of events logged for some rules
    { gid = 1, sid = 1, type = 'limit', track = 'by_src', count = 2, seconds = 10 },
    { gid = 1, sid = 2, type = 'both',  track = 'by_dst', count = 5, seconds = 60 },
}
--]]

--[[
rate_filter =
{
    -- alert on connection attempts from clients in SOME_NET
    { gid = 135, sid = 1, track = 'by_src', count = 5, seconds = 1,
      new_action = 'alert', timeout = 4, apply_to = '[$SOME_NET]' },

    -- alert on connections to servers over threshold
    { gid = 135, sid = 2, track = 'by_dst', count = 29, seconds = 3,
      new_action = 'alert', timeout = 1 },
}
--]]

---------------------------------------------------------------------------
-- 7. configure outputs
---------------------------------------------------------------------------

-- event logging
-- you can enable with defaults from the command line with -A <alert_type>
-- uncomment below to set non-default configs
--alert_csv = { }
--alert_fast = { }
--alert_full = { }
--alert_sfsocket = { }
--alert_syslog = { }
--unified2 = { }
  alert_syslog = {
                facility = local7,
                level = alert,
                options = pid
                }

-- packet logging
-- you can enable with defaults from the command line with -L <log_type>
--log_codecs = { }
--log_hext = { }
--log_pcap = { }

-- additional logs
--packet_capture = { }
--file_log = { }

---------------------------------------------------------------------------
-- 8. configure tweaks
---------------------------------------------------------------------------

if ( tweaks ~= nil ) then
    include(tweaks .. '.lua')
end
```

# Some basic commands
Now here are some basic snort commands

* Gets the version of snort and some other basic information
```shell
snort -V
```

* Lets you create a snort config file and run it 
```shell
snort -c /etc/snort/snort.lua # or path to your own config file
```
*  Helps you test out your config file for errors
```shell
snort -c /etc/snort/snort.lua -T
```

* Lets you set an interface for snort
```shell
snort -i eth1 # replace eth1 with the name of your interface
```

# Adding and configuring Rules
* Two places where I would recommend getting rules from are as follows
	** https://www.snort.org/downloads/community/snort3-community-rules.tar.gz	
  ** https://rules.emergingthreats.net/open/snort-$version/emerging.rules.tar.gz
	now just wget these, put these in the rules/ directory and then you are good to go

# To test out these rules 
* Now the best way I can recommend to test these out is by adding a sample rule which will basically be triggered everytime so you can properly see snort in action

so create a test.rules file and include it in your config
Add this in the file

```shell
alert icmp any any -> any any (msg:"ICMP Test"; sid:10000001; rev:1; classtype:icmp-event;)

```

```shell
snort -i eth1 -L dump --daq-dir /usr/lib/daq
```

# To run snort on multiple CPU cores instead of one 
Snort is an intensive programm especially if its running on the router of a big company which has hundreds of computers pinging it at once so another beautiful thing that came with snort3 was running it on multiple cores and just using the -z flag can help you do that

```shell
snort -i eth1 -L dump --daq-dir /usr/lib/daq -z 4 # This will run snort on 4 threads

```

# To redirect all logging to syslog
```shell
alert_syslog =
{
 facility = local7,
 level = alert,
 options = pid
}
```



