# ansible-role-bootstrap
![GitHub tag (latest SemVer)](https://img.shields.io/github/v/tag/chrecht/ansible-role-bootstrap?sort=semver&label=version)

Bootstraps a fresh Ubuntu server. Each task is individually toggled via a boolean variable — set a variable to `true` to enable it, or `false` (the default) to skip it.

## Requirements

- Ubuntu (xenial, focal, jammy, noble)
- Ansible 2.2+
- `ansible.posix` collection (for `authorized_key`, `mount`)
- `community.general` collection (for `shutdown`)

## Role Variables

### Enabled by default

| Variable | Default | Description |
|---|---|---|
| `bootstrap_set_hostname` | `true` | Set hostname from `inventory_hostname` |
| `bootstrap_hosts_file` | `true` | Add all hostnames and ip addresses from your inventroy to `/etc/hosts` |
| `bootstrap_os_updates` | `true` | Run `apt upgrade`, `dist-upgrade`, `autoremove`, and remove `unattended-upgrades` |
| `bootstrap_packages` | `true` | Install packages defined in `bootstrap_packages_list` |
| `bootstrap_ssh` | `true` | Harden SSH: disable empty passwords, root login, password auth; enable PAM |
| `bootstrap_admin_user` | `true` | Manage admin user sudo access and authorized SSH keys |

### Opt-in (disabled by default)

| Variable | Default | Description |
|---|---|---|
| `bootstrap_install_iscsi` | `false` | Install and enable `open-iscsi` / `iscsid` |
| `bootstrap_disable_swap` | `false` | Disable swap for current session and persist via `/etc/fstab` |
| `bootstrap_disable_multipathd` | `false` | Disable and stop the `multipathd` service |
| `bootstrap_grub_splash` | `false` | Remove `splash` from GRUB cmdline and run `update-grub2` |
| `bootstrap_longhorn_disks` | `false` | Partition, format, and mount a disk for Longhorn at `/var/lib/longhorn` |

### Package list

`bootstrap_packages_list` defines the packages installed when `bootstrap_packages: true`. Override or extend it in your host/group vars.

Default list:

```yaml
bootstrap_packages_list:
  - curl
  - wget
  - nano
  - htop
  - git
  - unzip
  - net-tools
  - ca-certificates
  - gnupg
  # - cifs-utils
```

### Other variables (required when tasks are enabled)

The role will fail early with a clear error message if any of these are missing when their associated task is enabled.

| Variable | Required when | Description |
|---|---|---|
| `domainname` | `bootstrap_set_hostname: true` | Domain appended to hostname in `/etc/hosts` |
| `admin_user` | `bootstrap_admin_user: true` | Username for admin user tasks |
| `ssh_authorized_keys` | `bootstrap_admin_user: true` | List of SSH public keys to add for the admin user |
| `ssh_authorized_keys_absent` | `bootstrap_admin_user: true` | List of SSH public keys to remove from the admin user |
| `longhorn_device` | `bootstrap_longhorn_disks: true` | Block device path for Longhorn disk (e.g. `/dev/sdb`) |

## Example Playbook

```yaml
- hosts: servers
  roles:
    - role: ansible-role-bootstrap
      vars:
        domainname: example.com
        bootstrap_disable_swap: true
        bootstrap_disable_multipathd: true
        bootstrap_grub_splash: true
        bootstrap_packages_list:
          - curl
          - wget
          - nano
          - htop
          - git
          - unzip
          - net-tools
          - ca-certificates
          - gnupg
          - cifs-utils
