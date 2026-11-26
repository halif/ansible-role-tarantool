# Ansible Role: Tarantool

[![CI](https://github.com/halif/ansible-role-tarantool/actions/workflows/ci.yml/badge.svg)](https://github.com/halif/ansible-role-tarantool/actions/workflows/ci.yml)
[![Ansible Galaxy](https://img.shields.io/badge/Ansible%20Galaxy-Tarantool-blue?logo=ansible)](https://galaxy.ansible.com/ui/standalone/roles/halif/tarantool/)
[![Galaxy Downloads](https://img.shields.io/ansible/role/d/halif/tarantool?label=Galaxy%20downloads)](https://galaxy.ansible.com/ui/standalone/roles/halif/tarantool/)
[![Galaxy Quality](https://img.shields.io/ansible/quality/halif/tarantool)](https://galaxy.ansible.com/ui/standalone/roles/halif/tarantool/)
[![GitHub Release](https://img.shields.io/github/v/release/halif/ansible-role-tarantool?display_name=tag)](https://github.com/halif/ansible-role-tarantool/releases)
[![GitHub Issues](https://img.shields.io/github/issues/halif/ansible-role-tarantool)](https://github.com/halif/ansible-role-tarantool/issues)
[![GitHub Stars](https://img.shields.io/github/stars/halif/ansible-role-tarantool)](https://github.com/halif/ansible-role-tarantool/stargazers)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Ansible](https://img.shields.io/badge/Ansible-2.15%2B-black?logo=ansible)](https://docs.ansible.com/)
[![Tarantool](https://img.shields.io/badge/Tarantool-3.x-blue)](https://www.tarantool.io/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-E95420?logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![AlmaLinux](https://img.shields.io/badge/AlmaLinux-9-0F4266?logo=almalinux&logoColor=white)](https://almalinux.org/)
[![openSUSE](https://img.shields.io/badge/openSUSE-Leap%2015.6-73BA25?logo=opensuse&logoColor=white)](https://www.opensuse.org/)
[![YAML](https://img.shields.io/badge/Config-YAML-CB171E?logo=yaml)](https://yaml.org/)
[![systemd](https://img.shields.io/badge/Service-systemd-555555?logo=linux)](https://systemd.io/)

Ansible role for installing and configuring **Tarantool 3.x** on Debian-based, RedHat-based, and SUSE Linux systems.

The role installs Tarantool using OS-specific installation methods, generates YAML configuration files for instances, and manages their lifecycle using systemd.

## Features

- Installs Tarantool 3.x.
- Supports Debian, RedHat, and SUSE operating system families.
- Configures the official Tarantool package repositories where applicable.
- Installs the `tarantool` package or binary.
- Installs the `tt` CLI on Debian-based and SUSE systems.
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

- Ansible Core 2.15 or later.
- Python 3.
- The `community.general` Ansible collection (required for SUSE package management).

### Managed nodes

- Linux with systemd.
- Root privileges or Ansible privilege escalation (`become: true`).
- Internet access to download the Tarantool installer, packages, and binaries.

### Additional openSUSE requirements

For openSUSE Leap 15.6:

- Python 3.11 must be installed before Ansible executes the role.
- Set `ansible_python_interpreter` to `/usr/bin/python3.11`.

Install Python 3.11:

```bash
sudo zypper --non-interactive install python311
```

Example inventory entry:

```ini
[tarantool_servers]
opensuse01 ansible_host=192.0.2.10 ansible_python_interpreter=/usr/bin/python3.11
```

### Supported and tested platforms

| Platform | OS family | CI status | Notes |
|---|---|---|---|
| Ubuntu 22.04 | Debian | Tested | Molecule |
| AlmaLinux 9 | RedHat | Tested | Compatibility is not explicitly documented by Tarantool upstream |
| openSUSE Leap 15.6 | Suse | Tested | Experimental integration; operating system reached EOL on April 30, 2026 |

**Important:** openSUSE Leap 15.6 no longer receives maintenance or security updates. It is retained in the test matrix for compatibility verification, but is not recommended for new production deployments.

Tarantool's official package installation documentation primarily describes APT- and YUM-based distributions. The SUSE installation method implemented by this role is an independently tested integration.

## Installation

### Install from Ansible Galaxy

```bash
ansible-galaxy role install halif.tarantool
```

Install the required Ansible collection:

```bash
ansible-galaxy collection install community.general:10.7.8
```

### Install directly from GitHub

```bash
ansible-galaxy role install git+https://github.com/halif/ansible-role-tarantool.git
```

## Role variables

The following variables are defined in `defaults/main.yml`.

### Tarantool version

```yaml
tarantool_major_version: "3"
```

The role installs Tarantool 3.x.

The exact patch version depends on the available upstream package or binary version.

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

On Debian-based systems, the role installs Tarantool and the `tt` CLI from the configured repositories.

On RedHat-based systems, the role installs the `tarantool` package. The `tt` CLI is not installed by default.

On SUSE systems, the role installs the `tt` CLI separately and uses it to install Tarantool Community Edition.

### SUSE installation variables

```yaml
tarantool_suse_binary_root: /opt/tarantool

tarantool_tt_cli_version: "3.0.0-alpha.2"
```

| Variable | Description |
|---|---|
| `tarantool_suse_binary_root` | Installation root for Tarantool binaries on SUSE |
| `tarantool_tt_cli_version` | Version of the `tt` CLI installed on SUSE |

The `tt` CLI version is pinned independently of the Tarantool major version.

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

## Installation methods by operating system

The role uses different installation methods depending on the target operating system.

| OS family | Installation method | `tt` CLI |
|---|---|---|
| Debian | Official Tarantool installer and APT repositories | Installed from packages |
| RedHat | Official Tarantool installer and RPM packages | Not installed by default |
| SUSE | `tt` CLI installer and Tarantool binary installation | Installed separately |

On SUSE, Tarantool binaries are installed under the directory configured by `tarantool_suse_binary_root`. The role also creates a standard `/usr/bin/tarantool` symlink.

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
    └── router.yaml -> ../instances.available/router.yaml

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

Enabled instances are linked into `instances.enabled`.

Instances marked with `enabled: false` are not linked and are stopped and disabled through systemd.

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

The CI test matrix includes:

- Ubuntu 22.04
- AlmaLinux 9
- openSUSE Leap 15.6

### Install testing dependencies

```bash
pip install ansible-core molecule
pip install 'molecule-plugins[docker]' docker
```

Install Ansible collection dependencies:

```bash
ansible-galaxy collection install community.general:10.7.8
```

### Run the Molecule scenario

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

### Run lint checks

```bash
yamllint -c .config/yamllint.yml .
ansible-lint
```

## Continuous Integration

GitHub Actions runs the following checks:

1. YAML lint.
2. Ansible lint.
3. Molecule tests on Ubuntu 22.04.
4. Molecule tests on AlmaLinux 9.
5. Molecule tests on openSUSE Leap 15.6.

The CI workflow is located in:

```text
.github/workflows/ci.yml
```

Molecule runs inside Docker containers on GitHub-hosted Linux runners. A local Docker installation is not required to use the GitHub Actions CI workflow.

## Limitations

- The role targets Tarantool 3.x and generates YAML configuration files.
- It does not provide automatic migration of existing Tarantool 2.x Lua configurations.
- The role generates basic instance configurations; application-specific settings should be added to the configuration template or managed separately.
- The `tt` CLI is not installed by default on RedHat-based systems.
- AlmaLinux 9 compatibility is tested by this role but is not explicitly documented by Tarantool upstream.
- openSUSE Leap 15.6 compatibility is tested by this role, but the operating system has reached end of life and is not recommended for production use.
- The SUSE installation method is an independently tested integration and is not an officially documented Tarantool package installation method.

## Official documentation

- [Tarantool documentation](https://www.tarantool.io/en/)
- [Tarantool 3 installation on Ubuntu](https://www.tarantool.io/en/download/os-installation/ubuntu/)
- [Tarantool installation on RHEL/CentOS](https://www.tarantool.io/en/download/os-installation/rhel-centos/)
- [Tarantool 3 configuration](https://www.tarantool.io/docs/tarantool/en/3_x/platform/configuration/)
- [tt CLI installation](https://www.tarantool.io/docs/tarantool/en/3_x/tooling/tt_cli/installation/)
- [openSUSE Leap lifecycle announcement](https://news.opensuse.org/2026/04/01/leap-15-eol/)

## License

MIT

## Contributions

Issues, bug reports, and pull requests are welcome.

If you encounter problems with a specific operating system or Tarantool version, please include:

- Operating system and version.
- Ansible version.
- Tarantool version.
- Relevant Molecule or Ansible output.
