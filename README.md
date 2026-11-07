# Ansible Role: Tarantool

[![CI](https://github.com/halif/ansible-role-tarantool/actions/workflows/ci.yml/badge.svg?branch=dev)](https://github.com/YOUR_GITHUB_USERNAME/ansible-role-tarantool/actions/workflows/ci.yml)
[![Ansible Galaxy](https://img.shields.io/badge/Ansible%20Galaxy-Tarantool-blue?logo=ansible)](https://galaxy.ansible.com/)
[![Galaxy Downloads](https://img.shields.io/ansible/role/d/halif/tarantool?label=Galaxy%20downloads)](https://galaxy.ansible.com/ui/standalone/roles/YOUR_GALAXY_NAMESPACE/tarantool/)
[![Galaxy Quality](https://img.shields.io/ansible/quality/halif/tarantool)](https://galaxy.ansible.com/ui/standalone/roles/YOUR_GALAXY_NAMESPACE/tarantool/)
[![GitHub Release](https://img.shields.io/github/v/release/halif/ansible-role-tarantool?display_name=tag)](https://github.com/YOUR_GITHUB_USERNAME/ansible-role-tarantool/releases)
[![GitHub Issues](https://img.shields.io/github/issues/halif/ansible-role-tarantool)](https://github.com/YOUR_GITHUB_USERNAME/ansible-role-tarantool/issues)
[![GitHub Stars](https://img.shields.io/github/stars/halif/ansible-role-tarantool)](https://github.com/YOUR_GITHUB_USERNAME/ansible-role-tarantool/stargazers)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Ansible](https://img.shields.io/badge/Ansible-2.15%2B-black?logo=ansible)](https://docs.ansible.com/)
[![Tarantool](https://img.shields.io/badge/Tarantool-3.x-blue)](https://www.tarantool.io/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-E95420?logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![AlmaLinux](https://img.shields.io/badge/AlmaLinux-9-0F4266?logo=almalinux&logoColor=white)](https://almalinux.org/)
[![YAML](https://img.shields.io/badge/Config-YAML-CB171E?logo=yaml)](https://yaml.org/)
[![Systemd](https://img.shields.io/badge/Service-systemd-555555?logo=linux)](https://systemd.io/)

Ansible role for installing and configuring **Tarantool 3.x** on Debian-based and RedHat-based Linux systems.

The role configures the official Tarantool 3 package repository, installs Tarantool, generates YAML configuration files for instances, and manages their lifecycle using systemd.

## Features

- Installs Tarantool 3.x using the official Tarantool installer.
- Configures package repositories for Debian and RedHat families.
- Installs the `tarantool` package.
- Installs the `tt` CLI on Debian-based systems.
- Creates a dedicated system user and group.
- Generates YAML configuration files for each Tarantool instance.
- Supports multiple Tarantool instances on a single host.
- Configures separate directories for snapshots, WAL files, and Vinyl data.
- Manages instance configuration symlinks.
- Creates a systemd service template for Tarantool instances.
- Starts and enables configured instances.
- Stops and disables instances marked as disabled.

## Requirements

### Control node

- Ansible Core 2.15 or later recommended.
- Python 3.

### Managed nodes

- Linux with systemd.
- Root privileges or Ansible privilege escalation (`become: true`).
- Internet access to download the Tarantool installer and packages.

### Supported platforms

| Platform | Status |
|---|---|
| Ubuntu 22.04 | Tested with Molecule |
| AlmaLinux 9 | Experimental; compatibility testing in progress |

**Note:** AlmaLinux 9 is not explicitly listed among the operating systems supported by the official Tarantool RHEL/CentOS installation documentation. Compatibility should be validated in your environment.

## Installation

Install the role from Ansible Galaxy:

```bash
ansible-galaxy role install YOUR_GALAXY_NAMESPACE.tarantool
```

Alternatively, install it directly from GitHub:

```bash
ansible-galaxy role install git+https://github.com/YOUR_GITHUB_USERNAME/ansible-role-tarantool.git
```

## Role variables

The following variables are defined in `defaults/main.yml`.

### Tarantool version

```yaml
tarantool_major_version: "3"
```

The role installs Tarantool 3.x.

### Packages

Debian-based systems:

```yaml
tarantool_packages:
  - tarantool
  - tt
```

RedHat-based systems:

```yaml
tarantool_packages_redhat:
  - tarantool
```

The `tt` CLI is installed through the Tarantool modules repository on Debian-based systems. It is not installed by default on RedHat-based systems.

### Tarantool instances

Default configuration:

```yaml
tarantool_instances:
  - name: router
    enabled: true
    port: 3301

  - name: storage_1
    enabled: false
    port: 3302
```

Each instance supports the following properties:

| Variable | Required | Default | Description |
|---|---|---|---|
| `name` | Yes | — | Instance name |
| `enabled` | Yes | — | Whether the instance should be running and enabled at boot |
| `port` | No | `3301` | IPROTO listening port |

Use a unique port for each instance.

## Example Playbook

```yaml
---
- name: Install and configure Tarantool
  hosts: tarantool_servers
  become: true

  vars:
    tarantool_instances:
      - name: router
        enabled: true
        port: 3301

      - name: storage_1
        enabled: true
        port: 3302

      - name: storage_2
        enabled: false
        port: 3303

  roles:
    - halif.tarantool
```

Run the playbook:

```bash
ansible-playbook -i inventory.ini site.yml
```

## Configuration

The role automatically generates a YAML configuration file for every configured instance.

For example, the `router` instance will have the following configuration:

```yaml
groups:
  default:
    replicasets:
      router:
        instances:
          router:
            iproto:
              listen:
                - uri: "127.0.0.1:3301"

            snapshot:
              dir: "/var/lib/tarantool/router/snapshots"

            wal:
              dir: "/var/lib/tarantool/router/wals"

            vinyl:
              dir: "/var/lib/tarantool/router/vinyl"
```

### Configuration directories

The role uses the following directory structure:

```text
/etc/tarantool/
├── instances.available/
│   ├── router.yaml
│   └── storage_1.yaml
│
└── instances.enabled/
    ├── router.yaml -> ../instances.available/router.yaml
    └── storage_1.yaml

/var/lib/tarantool/
├── router/
│   ├── snapshots/
│   ├── wals/
│   └── vinyl/
│
└── storage_1/
    ├── snapshots/
    ├── wals/
    └── vinyl/
```

Configuration files are generated in `instances.available`.

Enabled instances are linked into `instances.enabled`. Instances marked with `enabled: false` are not linked and are stopped and disabled through systemd.

## Managing instances

Each Tarantool instance is managed by a separate systemd service.

The service naming convention is:

```text
tarantool@INSTANCE_NAME
```

Examples:

Start an instance:

```bash
systemctl start tarantool@router
```

Stop an instance:

```bash
systemctl stop tarantool@router
```

Restart an instance:

```bash
systemctl restart tarantool@router
```

Check instance status:

```bash
systemctl status tarantool@router
```

View logs:

```bash
journalctl -u tarantool@router -f
```

Enable an instance at boot:

```bash
systemctl enable tarantool@router
```

## Tags

The role currently does not define dedicated task tags. It can be executed as a regular Ansible role.

## Testing

The role uses Molecule with Docker to test installation and configuration.

The current test matrix includes:

- Ubuntu 22.04
- AlmaLinux 9

Install the testing dependencies:

```bash
pip install ansible-core molecule
pip install 'molecule-plugins[docker]' docker
```

Run the test scenario:

```bash
molecule test
```

Run individual Molecule steps:

```bash
molecule syntax
molecule create
molecule converge
molecule verify
molecule destroy
```

Run lint checks:

```bash
yamllint -c .config/yamllint.yml .
ansible-lint
```

## CI

GitHub Actions runs the following checks:

1. YAML lint.
2. Ansible lint.
3. Molecule tests on the configured Linux distributions.

The CI workflow is located in:

```text
.github/workflows/ci.yml
```

## Limitations

- The role targets Tarantool 3.x and generates YAML configuration files.
- It does not provide automatic migration of existing Tarantool 2.x Lua configurations.
- The role generates basic instance configurations; application-specific settings should be added to the generated configuration template or managed separately.
- The `tt` CLI is installed by default only on Debian-based systems.
- AlmaLinux 9 compatibility is experimental and should be verified before production use.

## Official documentation

- [Tarantool documentation](https://www.tarantool.io/en/)
- [Tarantool 3 installation on Ubuntu](https://www.tarantool.io/en/download/os-installation/ubuntu/)
- [Tarantool installation on RHEL/CentOS](https://www.tarantool.io/en/download/os-installation/rhel-centos/)
- [Tarantool 3 configuration](https://www.tarantool.io/docs/tarantool/en/3_x/platform/configuration)
- [tt CLI installation](https://www.tarantool.io/docs/tarantool/en/3_x/tooling/tt_cli/installation/)

## License

MIT

## Contributions

Issues, bug reports, and pull requests are welcome.

If you encounter problems with a specific operating system or Tarantool version, please include the OS version, Ansible version, and relevant Molecule output in your report.