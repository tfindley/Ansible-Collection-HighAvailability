# VRRP Instance Variables

Documentation for keys within each entry of the `keepalived_vrrp` list. For more information see the [KeepAliveD manpage](https://www.keepalived.org/manpage.html).

---

## name

**Role Variable key**: `keepalived_vrrp[*].name`
**Type:** string
**Required:** Yes

The unique name of the VRRP instance. Used as the identifier in the generated `vrrp_instance <name> { }` block.

---

## state

**Role Variable key**: `keepalived_vrrp[*].state`
**Type:** string (`MASTER` | `BACKUP`)
**Required:** No
**Default Role value:** `BACKUP`

Initial state of the VRRP instance. To prevent service flapping it is recommended to use `BACKUP` on all nodes and differentiate by `priority` instead.

---

## interface

**Role Variable key**: `keepalived_vrrp[*].interface`
**Type:** string
**Required:** Yes
**Recommended value:** `"{{ ansible_facts['default_ipv4']['alias'] }}"`

Network interface that VRRP binds to. The recommended value automatically uses the interface Ansible connects over.

---

## priority

**Role Variable key**: `keepalived_vrrp[*].priority`
**Type:** int
**Required:** Yes
**Recommended value:** `100`

VRRP election priority. The node with the highest priority becomes MASTER. Must be an integer — do not quote this value.

---

## virtual_router_id

**Role Variable key**: `keepalived_vrrp[*].virtual_router_id`
**Type:** int (1–255)
**Required:** Yes

Arbitrary unique number from 1 to 255 used to differentiate multiple instances of vrrpd running on the same network interface and address family and multicast/unicast (and hence same socket).

Note: using the same virtual_router_id with the same address family on different interfaces has been known to cause problems with some network switches; if you are experiencing problems with using the same virtual_router_id on different interfaces, but the problems are resolved by not duplicating virtual_router_ids, your network switches are probably not functioning correctly.

Whilst in general it is important not to duplicate a virtual_router_id on the same network interface, there is a special case when using unicasting if the unicast peers for the vrrp instances with duplicated virtual_router_ids on the network interface do not overlap, in which case virtual_router_ids can be duplicated. It is also possible to duplicate virtual_router_ids on an interface with multicasting if different multicast addresses are used (see mcast_dst_ip).

---

## advert_int

**Role Variable key**: `keepalived_vrrp[*].advert_int`
**Type:** int
**Required:** Yes
**Recommended value:** `1`

VRRP advert interval in seconds. All nodes in the same VRRP group must use the same value.

---

## garp

**Role Variable key**: `keepalived_vrrp[*].garp`
**Type:** dict
**Required:** No

Gratuitous ARP configuration. When defined, the following sub-keys are required:

### garp.master_refresh

**Type:** int
**Recommended value:** `5`

Minimum time interval in seconds for refreshing gratuitous ARPs while MASTER.

### garp.master_refresh_repeat

**Type:** int
**Recommended value:** `1`

Number of gratuitous ARP messages to send at a time while MASTER.

**Example:**
```yaml
garp:
  master_refresh: 5
  master_refresh_repeat: 1
```

---

## authentication

**Role Variable key**: `keepalived_vrrp[*].authentication`
**Type:** dict
**Required:** No
**VRRP version:** v2 only (incompatible with v3)

Authentication configuration for VRRP adverts. Use of this option is non-compliant with RFC 3768; avoid where possible. When defined, the following sub-keys are required:

### authentication.type

**Type:** string (`PASS` | `AH`)

Authentication method. `PASS` (simple password) is suggested. `AH` (IPSEC AH) is not recommended.

### authentication.pass

**Type:** string

Password for VRRP authentication. Must be the same on all nodes in the group. Only the first 8 characters are used.

**Example:**
```yaml
authentication:
  type: PASS
  pass: "mysecret"
```

---

## unicast_src_ip

**Role Variable key**: `keepalived_vrrp[*].unicast_src_ip`
**Type:** string (IPv4 address)
**Required:** No (required when `unicast_peer` is set)
**Recommended value:** `"{{ ansible_facts['default_ipv4']['address'] }}"`

Source IP address for unicast VRRP adverts. Only needed when operating in unicast mode (i.e. when `unicast_peer` is defined). Omit this key entirely when using multicast.

---

## unicast_peer

**Role Variable key**: `keepalived_vrrp[*].unicast_peer`
**Type:** list of IPv4 address strings
**Required:** No
**Recommended value:** `"{{ keepalived_iplist }}"`

List of peer IP addresses to send VRRP adverts to via unicast instead of multicast. Use `keepalived_iplist` (auto-populated from all hosts in the play) for the recommended value.

When this key is omitted, keepalived uses multicast (224.0.0.18 by default), which works on most on-premises networks but may not be supported in cloud/virtual environments.

**Example:**
```yaml
unicast_src_ip: "{{ ansible_facts['default_ipv4']['address'] }}"
unicast_peer: "{{ keepalived_iplist }}"
```

---

## vip

**Role Variable key**: `keepalived_vrrp[*].vip`
**Type:** list of IP address strings
**Required:** Yes

The virtual IP address(es) managed by this VRRP instance. Must always be a list, even for a single address. Maps to the `virtual_ipaddress { }` block in the keepalived configuration.

**Example:**
```yaml
vip:
  - 192.168.1.100
  - 192.168.1.101
```

---

## checkscript

**Role Variable key**: `keepalived_vrrp[*].checkscript`
**Type:** list of strings
**Required:** No

List of checkscript names to track for this VRRP instance. Each entry must exactly match a `name` value defined in `keepalived_checkscript_scripts`. When a tracked script enters FAULT state, the VRRP priority is adjusted by the script's `weight` value.

Only used when `keepalived_checkscript_enabled: true`.

Maps to the `track_script { }` block in the keepalived configuration.

**Example:**
```yaml
checkscript:
  - vault_active_node_script
  - check_grafana_health_api
```
