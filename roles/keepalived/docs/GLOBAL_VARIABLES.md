# Global Variables

Below is documentation for implemented global variables. For more informatino see: [KeepAliveD manpage](https://www.keepalived.org/manpage.html).

## router_id

**Role Variable**: `keepalived_router_id`
**Expected value:** `new_hostname`
**Required:** No
**Default KeepAliveD value:** (local host name)
**Default Role value:** 

String identifying the machine (doesn't have to be hostname).

## vrrp_version

**Role Variable**: `keepalived_vrrp_version`
**Expected value:** `2`
**Required:** No
**Default KeepAliveD value:** 2, but IPv6 instances will use version 3
**Default Role value:** (unset)

Set the default VRRP version to use

## vrrp_strict

**Role Variable**: `keepalived_vrrp_strict`
**Expected value:** `true`|`false`
**Required:** No
**Default KeepAliveD value:** (unset)
**Default role value:** false

Enforce strict VRRP protocol compliance. This currently includes enforcing the following. Please note that other checks may be added in the future if they are found to be missing:
- 0 VIPs not allowed
- unicast peers not allowed
- IPv6 addresses not allowed in VRRP version 2
- First IPv6 VIP is not link local
- State MASTER can be configured if and only if priority is 255
- Authentication is not supported
- Preempt delay is not supported
- Accept mode cannot be set for VRRPv2
- If accept/no accept is not specified, accept is set if priority
-  is 255 aand cleared otherwise
- Gratuitous ARP repeats cannot be enabled
- Cannot clear lower_prio_no_advert
- Cannot set higher_prio_send_adver

## vrrp_startup_delay

**Role Variable**: `keepalived_vrrp_startup_delay`
**Expected value:** `5.5`
**Required:** No
**Default KeepAliveD value:** (unset)
**Default role value:** (unset)

On some systems when bond interfaces are created, they can start passing traffic and then have a several second gap when they stop passing traffic inbound. This can mean that if keepalived is started at boot time, i.e. at the same time as bond interfaces are being created, keepalived doesn't receive adverts and hence can become master despite an instance with higher priority sending adverts. This option specifies a delay in seconds before vrrp instances start up after keepalived starts,

## max_auto_priority

**Role Variable**: `keepalived_max_auto_priority`
**Expected value:** [<-1 to 99>]  # 99 is really sched_get_priority_max(SCHED_RR)
**Required:** No
**Default KeepAliveD value:** (unset, disables automatic priority increases)
**Default role value:** `99`

The following options can be used if vrrp, checker or bfd  processes are timing out. This can be seen by a backup vrrp instance becoming master even when the master is still running, because the master or backup system is too busy to process vrrp packets.
--
keepalived can, if it detects that it is not running sufficiently soon after a timer should expire, increase its priority, first of all switching to realtime scheduling, and if that is not sufficient, it will then increase its realtime priority by one each time it detects a further delay in running. If the event that realtimescheduling is enabled, RLIMIT_RTTIME will be set, using the values for {bfd,checker,vrrp}_rlimit_rttime (see below). These values may need to be increased for slower processors.
--
To limit the maximum increased automatic priority, specify the following (0 doesn't use automatic priority increases, and is the default. -1 disables the warning message at startup). Omitting the priority sets the maximum value.

## script_user

**Expected value:** (unix_user_name) (unix_group_name)
**Required:** No
**Default KeepAliveD value:** (none)
**Default role value:** `{{ keepalived_checkscript_user }} {{ keepalived_checkscript_group }}`


Specify the default username/groupname to run scripts under. If this option is not specified, the user defaults to `keepalived_script` if that user exists, or the user keepalived runs as otherwise.

For this role, we have two role variables that combine to fill this Keepalived variable. If they do not exist, then the role will create both the username and group.

### checkscript_user

**Role Variable**: `keepalived_checkscript_user`
**Default role value:** `keepalived_script`

### checkscript_group

**Role Variable**: `keepalived_checkscript_group`
**Default role value:** `keepalived_script`
