# Ansible Collection - tfindley.highavailability

This collection replaces my previous tfindley.keepalived collection.

This HighAvailability colletion wraps up HA tooling into a single collection for easy consumption.

## Compatibility

| Ansible Core | Tested |
| ------------ | ------ |
| 2.18         | ✅     |
| 2.19         | ✅     |


| Ansible Feature | Working | Notes               |
| --------------- | ------- | ------------------- |
| `--diff` mode   | ✅      |                     |
| `--check` mode  | ✅      |                     |
| Idempotency     | ✅      |                     |
| Linting         | ✅      |                     |

## Roles

- tfindley.highavailability.keepalived

## Playbooks

- tfindley.highavailability.keepalived_basic

This deploys a basic version if KeepAliveD. As a minimum you need to pass the following variables: `keepalived_enabled`, `keepalived_vip`, `keepalived_vrid`

- tfindley.highavailability.keepalived_pass

This will deploy a basic version of KeepAliveD but with basic PASSWORD authentication between the two instances. As a minimum you need to pass the following variables: `keepalived_enabled`, `keepalived_vip`, `keepalived_vrid`, `keepalived_pass`

- tfindley.highavailability.keepalived_hashivault

This will deploy KeepAliveD but with basic PASSWORD authentication enabled and the checkscript required to monitor a Hashicorp Vault installation on the default port (8200). As a minimum you need to pass the following variables: `keepalived_enabled`, `keepalived_vip`, `keepalived_vrid`, `keepalived_pass`

## Variables

| Variable                         | Type      | Required | Description                     | Default Value     |
| -------------------------------- | --------- | -------- | ----------------------------------------------------------------------------------------------- | ----------------- |
| `keepalived_enabled`             | bool      | Yes      | Boolean value to enable the use of the keepalived role as a safeguard.                          |                   |
| `keepalived_vip`                 | list      | Yes      | `xyz.xyz.xyz.xyz` format IP address in a list format `[]`. Must specify at least one IP address |                   |
| `keepalived_vrid`                | int       | Yes      | Your virtual router ID. IMPORTANT: On a network you cannot have conflicting Virtual Router IDs  | `123`             |
| `keepalived_pass`                | string    | No       | Specify a password. Only used in the `keepalived_pass` and `keepalived_hashivault` sample plays |                   |
| `keepalived_state`               | string    | No       | The state of the node in the KeepAliveD cluster. Recommended to leave all nodes as backup       | `BACKUP`          |
| `keepalived_vrrp_version`        | int       | No       | Set the VRRP version you wish to use. I would not recommend changing from 2                     | `2`               |
| `keepalived_vrrp`                | dict      | No       | See below - Required if you want to control how KeepAliveD is deployed beyond the sample plays  |                   |

```yaml
keepalived_vrrp:
    - name: VI_1
      state: "{{ keepalived_state | default('BACKUP') }}"
      interface: "{{ ansible_default_ipv4['alias'] }}"
      priority: "{{ keepalived_priority | default(100) }}"
      virtual_router_id: "{{ keepalived_vrid }}"
      advert_int: 1
      garp:
        master_refresh: 5
        master_refresh_repeat: 1
      authentication:
        type: PASS  # or AH
        pass: "{{ keepalived_pass }}"  # to set a random pass; "{{ lookup('community.general.random_string', length=8, special=false) }}" - this is not idempotent
      unicast_src_ip: "{{ ansible_default_ipv4.address }}"  # Shouldn't need to adjust this
      unicast_peer: "{{ keepalived_iplist }}" # Shouldn't need to adjust this
      vip: "{{ keepalived_vip }}"
      checkscript:
        - random_check_script
```

| Variable                         | Type      | Required | Description                     | Default Value     |
| -------------------------------- | --------- | -------- | ----------------------------------------------------------------------------------------------- | ------------------  |
| `keepalived_checkscript_enabled` | bool      | No       | set to true to enable checkscript support and incldue the following vars                        | `False`             |
| `keepalived_checkscript_user`    | string    | No       | The user that will be used to execute check scripts with KeepAliveD (Cannot be a root user)     | `keepalived_script` |
| `keepalived_checkscript_group`   | string    | No       | The group for the user that will be used to execute check scripts with KeepAliveD               | `keepalived_script` |
| `keepalived_checkscript_path`    | string    | No       | The default path for the checkscript storage location. use this to override the role default    | `"{{ keepalived_checkscript_dir }}"` |
| `keepalived_checkscript_scripts` | dict      | No       | See below - Required if you want to control how KeepAliveD is deployed beyond the sample plays  |                   |

```yaml
keepalived_checkscript_enabled: true  # set to true to enable checkscript support and incldue the following vars
keepalived_checkscript_scripts:
  - name: random_check_script
    filename: check_vault.py
    exec: --timeout=1 --url='https://127.0.0.1:8200'
    content: "{{ lookup('ansible.builtin.file', 'hashicorp/vault/check_vault.py') }}"
    mode: '0700'
    interval: 2  # Run script every 2 seconds
    fall: 1  # If script returns non-zero 2 times in succession, enter FAULT state
    rise: 3  # If script returns zero r times in succession, exit FAULT state
    timeout: 2  # Wait up to t seconds for script before assuming non-zero exit code
    weight: 50  # Reduce priority by 50 on fall
```

### Checkscripts

Some checkscripts have been pacakged with the keepalived role. These are uploaded or installed during the KeepAlived Deployment. 

- **check_json_api.py** - Generic JSON API check used to pull a key/value pair
  - content: `"{{ lookup('ansible.builtin.file', 'hashicorp/vault/check_vault.py') }}"`
  - exec:
    - `-u` / `--url` - URL to query (required)
    - `-f` / `--field` - JSON field you wish to query (required)
    - `-r` / `--response` - JSON value you wish to check for (required)
    - `-t` / `--timeout` - Timeout for the HTTP request (default='5') (optional)
    - `-q` / `--quiet` - Suppress output except for errors (opional)

- check_vault.py - Hashicorp Vault specific check to ensure the service status and unseal status of the local Hashicorp Vault service
  - content: `"{{ lookup('ansible.builtin.file', 'generic/check_json_api.py') }}"`
  - exec:
    - `--timeout=1`
    - `--url='https://127.0.0.1:8200'`
  - Note: Full credit to [madrisan](https://github.com/madrisan) for this script!

You can also build and upload a check from your own filesystem by replacing the 'content' with a lookup to a local file path: `"{{ lookup('ansible.builtin.file', '/path/to/your/chec)k_script.py') }}"`

## Environmental Variables

None required

## License

AGPL

## Author Information

**Tristan Findley**

Find out more about me [here](https://tfindley.co.uk).

If you're fan of my work and would like to show your support:

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/Z8Z016573P)
