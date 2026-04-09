# KeepAliveD

This role deploys and configures KeepAlived.

## Requirements

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

## Role Variables

| Variable Name                           | Type    | Required | default value         | Description |
| --------------------------------------- | ------- | -------- | --------------------- | ----------- |
| `keepalived_enabled`                    | boolean | True     | `false`               | Enables or disables KeepaliveD. This is used as a safeguard to prevent accidental deployment. This variable must be present on each system you require KeepAliveD installing |

### Helper Variables

These are designed to do very simple deployments of KeepAliveD. It is recommended to only use these in basic deployments.

| Variable Name                           | Type    | Required | default value         | Description |
| --------------------------------------- | ------- | -------- | --------------------- | ----------- |
| `keepalived_vip`                        | list    | True     | `["192.168.69.201"]`  | The IP Addresses you wish to set as Virtual IP |
| `keepalived_vrid`                       | int     | True     | `123`                 | Your VRRP Virtual Router ID. Do not duplicate these on the same network |
| `keepalived_priority`                   | int     | True     | `100`                 | for electing MASTER, highest priority wins |
| `keepalived_state`                      | string  | True     | `BACKUP`              | This can either be `MASTER`, or `BACKUP`. To prevent service flapping, it is suggested to use `BACKUP` on all nodes and set the priority equally on each node. |
| `keepalived_unicast_mode`               | boolean | False    | `false`               | When `true`, VRRP adverts are sent via unicast instead of multicast. Requires `unicast_src_ip` and `unicast_peer` to be set on each VRRP instance. Use multicast (default) unless your network does not support it. |

### Global Variables

| Variable Name                           | Type    | Required | default value         | Description |
| --------------------------------------- | ------- | -------- | --------------------- | ----------- |
| `keepalived_max_auto_priority`          | int     | True     | `99`                  | To limit the maximum increased automatic priority, specify the following. (0 doesn't use automatic priority increases, and is the default. -1 disables the warning message at startup). Omitting the priority sets the maximum value. |
| `keepalived_vrrp_version`               | int     | False    | (unset)               | Set the default VRRP version to use. (default: 2, but IPv6 instances will use version 3) |
| `keepalived_vrrp_strict`                | boolean | False    | `false`               | Enforce strict VRRP protocol compliance. Note: strict mode disallows unicast peers and authentication. See [docs/GLOBAL_VARIABLES.md](docs/GLOBAL_VARIABLES.md) for details. |
| `keepalived_vrrp_startup_delay`         | float   | False    | (unset)               | Delay in seconds before VRRP instances start after keepalived starts. Useful when bond interfaces are slow to initialise. |
| `keepalived_router_id`                  | string  | False    | (unset)               | String identifying the machine (doesn't have to be hostname) |
| `keepalived_checkscript_user`           | string  | True     | `keepalived_script`   | Specify the default username to run scripts under |
| `keepalived_checkscript_group`          | string  | True     | `keepalived_script`   | Specify the default groupname to run scripts under |
| `keepalived_checkscript_path`           | string  | True     | *See below*           | *See below* |

### Checkscript Variables

| Variable Name                           | Type    | Required | default value         | Description |
| --------------------------------------- | ------- | -------- | --------------------- | ----------- |
| `keepalived_checkscript_enabled`        | boolean | True     | `false`               | |

**keepalived_checkscript_path:** The location of where check scripts will be stored. This is variable is preset based on the host OS family that you're installing to, but can be overritten in your playbook if you wish.

The default for RHEL-based systems is `/usr/libexec/keepalived` - this directory is created by keepalived with the appropriate SELinux permissions to allow the Keepalived script user that is defined in global_defs to run scripts. The role will ensure that all scripts deployed to this directory will have the correct owner/group/mode set.

Information Ref: [https://opensource-db.com/working-with-keepalived-and-selinux-ensuring-ha-and-security/](https://opensource-db.com/working-with-keepalived-and-selinux-ensuring-ha-and-security/)

### Keepalived VRRP

variables for the `keepalived_vrrp` dictionary instances

| Variable Name              | Type    | Required | default value | Recommended Value | Description |
| -------------------------- | ------- | -------- | ------------- | ---------------------------------------------------------------------------- | ------------|
| `name`                     | string  | True     | (none)        | `VI_1`                                                                       | The unique name of the VRRP instance |
| `state`                    | string  | False    | `BACKUP`      | `BACKUP`                                                                     | This can either be `MASTER`, or `BACKUP`. To prevent service flapping, it is suggested to use `BACKUP` on all nodes and set the priority equally on each node. |
| `interface`                | string  | True     | (none)        | `"{{ ansible_facts['default_ipv4']['alias'] }}"`                                      | Interface bound by VRRP. The recommended value automatically uses the interface Ansible connects over. |
| `priority`                 | int     | True     | (none)        | `100`                                                                        | For electing MASTER, highest priority wins |
| `virtual_router_id`        | int     | True     | (none)        | (none)                                                                       | Your VRRP Virtual Router ID (1–255). Do not duplicate these on the same network! |
| `advert_int`               | int     | True     | (none)        | `1`                                                                          | VRRP advert interval in seconds |
| `garp`                     | dict    | False    | (none)        |                                                                              | Gratuitous ARP settings. See sub-keys below. |
|  - `master_refresh`        | int     | False    | (none)        | `5`                                                                          | Minimum time interval (seconds) for refreshing gratuitous ARPs while MASTER |
|  - `master_refresh_repeat` | int     | False    | (none)        | `1`                                                                          | Number of gratuitous ARP messages to send at a time while MASTER |
| `authentication`           | dict    | False    | (none)        |                                                                              | VRRP v2 only. Non-compliant with RFC; use with caution. |
|  - `type`                  | string  | False    | (none)        | `PASS`                                                                       | `PASS` — simple password (suggested), `AH` — IPSEC (not recommended) |
|  - `pass`                  | string  | False    | (none)        | `"{{ lookup('community.general.random_string', length=8, special=false) }}"` | Password for accessing vrrpd. Must be the same on all nodes. Only the first 8 characters are used. |
| `unicast_src_ip`           | string  | False    | (none)        | `"{{ ansible_facts['default_ipv4']['address'] }}"`                                       | Source IP for unicast VRRP adverts. Required when `unicast_peer` is set. Omit for multicast mode. |
| `unicast_peer`             | list    | False    | (none)        | `"{{ keepalived_iplist }}"`                                                  | List of peer IPs to send unicast VRRP adverts to instead of multicast. Use `keepalived_iplist` to auto-populate from all hosts in the play. Omit to use multicast. |
| `vip`                      | list    | True     | (none)        | (none)                                                                       | The IP address(es) to use as Virtual IP. Must be a list. |
| `checkscript`              | list    | False    | (none)        | (none)                                                                       | List of checkscript names to track for this VRRP instance. Each entry must match a `name` value in `keepalived_checkscript_scripts`. |

### Keepalived Checkscript Scripts

There are three modes for defining a checkscript. Use **exactly one** of `filename` or `command` per entry.

| Variable Name | Type    | Required        | default value | Description |
| ------------- | ------- | --------------- | ------------- | ----------- |
| `name`        | string  | True            | (none)        | Unique name for this script. Must be referenced in `vrrp[*].checkscript` to be tracked. |
| `filename`    | string  | Mode 1 & 2 only | (none)        | Filename of the script to deploy. Mutually exclusive with `command`. |
| `command`     | string  | Mode 3 only     | (none)        | Command or binary on the managed host to run directly. Can be a bare command name (`systemctl`) or full path (`/usr/bin/systemctl`). Mutually exclusive with `filename`. |
| `content`     | string  | Mode 1 only     | (none)        | File content to upload. If omitted when using `filename`, the file must exist in the role's `files/` directory (mode 2). |
| `exec`        | string  | False           | (none)        | Arguments or flags appended to the script/command. |
| `mode`        | string  | Mode 1 & 2 only | `0700`        | File permissions for the deployed script. |
| `interval`    | int     | True            | (none)        | Run the check every N seconds. |
| `fall`        | int     | True            | (none)        | Enter FAULT state after N consecutive non-zero exits. |
| `rise`        | int     | True            | (none)        | Exit FAULT state after N consecutive zero exits. |
| `timeout`     | int     | True            | (none)        | Seconds to wait before assuming non-zero exit. |
| `weight`      | int     | True            | (none)        | Adjust VRRP priority by this value on FAULT (negative reduces priority). |

## Checkscripts

### Mode 1 — Custom script upload

Upload a script from your deployment host to the managed node. Use `content` with a `lookup()` to read the file.

```yaml
vars:
  keepalived_checkscript_enabled: true

  keepalived_vrrp:
    - name: VI_1
      vip: "{{ keepalived_vip }}"
      checkscript:
        - check_vault

  keepalived_checkscript_scripts:
    - name: check_vault
      filename: check_vault.py
      content: "{{ lookup('ansible.builtin.file', 'hashicorp/vault/check_vault.py') }}"
      exec: --timeout=1 --url='https://127.0.0.1:8200'
      mode: '0700'
      interval: 2
      fall: 1
      rise: 3
      timeout: 2
      weight: 50
```

### Mode 2 — Bundled role script

Use a script already included in this role's `files/` directory. Omit `content` — the role copies it automatically.

**WARNING:** The Grafana script requires the `python3-click` package. Install it in a `pre_tasks` block:

```yaml
pre_tasks:
  - name: Install Python3 Click
    become: true
    ansible.builtin.package:
      name: python3-click
      state: present
```

```yaml
vars:
  keepalived_checkscript_enabled: true

  keepalived_vrrp:
    - name: VI_1
      vip: "{{ keepalived_vip }}"
      checkscript:
        - check_grafana

  keepalived_checkscript_scripts:
    - name: check_grafana
      filename: check_grafana_health_api.py
      exec: --url http://localhost:3000/api/health --field database --response ok --timeout 3 --quiet
      mode: '0700'
      interval: 2
      fall: 1
      rise: 3
      timeout: 2
      weight: 50
```

### Mode 3 — System command

Run a command or binary already present on the managed host. No file is copied. Use `command` instead of `filename`.

```yaml
vars:
  keepalived_checkscript_enabled: true

  keepalived_vrrp:
    - name: VI_1
      vip: "{{ keepalived_vip }}"
      checkscript:
        - check_haproxy

  keepalived_checkscript_scripts:
    - name: check_haproxy
      command: systemctl          # bare name or full path, e.g. /usr/bin/systemctl
      exec: is-active --quiet haproxy
      interval: 2
      fall: 2
      rise: 2
      timeout: 2
      weight: -20
```

## Dependencies

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

## License

BSD


## Author Information

**Tristan Findley**

Find out more about me [here](https://tfindley.co.uk).

If you're fan of my work and would like to show your support:

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/Z8Z016573P)