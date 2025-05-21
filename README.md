[![Build
Status](https://travis-ci.com/nkakouros-original/ansible-role-taskserver.svg?branch=master)](https://travis-ci.com/nkakouros-original/ansible-role-taskserver)
[![Galaxy](https://img.shields.io/badge/galaxy-nkakouros.taskserver-blue.svg)](https://galaxy.ansible.com/nkakouros/taskserver/)

# nkakouros.taskserver

Ansible role to setup a Taskserver(taskd) for Taskwarrior.

This role follows closely the tutorial at
https://gitpitch.com/GothenburgBitFactory/taskserver-setup

It will install taskd as per the tutorial, download client certificates to be
used in other devices and generate config snippets for configuring the android
app among other things.

Original work on the role was done by Wilmar den Ouden at
https://github.com/wilmardo/ansible-role-taskserver. I relicensed it as GPLv3
and added more functionality, fixed bugs, etc.

## Requirements

None

## Role Variables

Take a look at [defaults/main.yml](defaults/main.yml) for documentation on the
available variables.

### Example usage

**Main playbook**

```yaml
- hosts: reverse-proxy-01
  vars:
    role_path: "{{ playbook_dir }}/roles/taskwarrior-server"
    taskd_organizations:
      - First Last
    taskd_users:
      - name: "First Last"
        organization: First Last
  tasks:
    - include_vars: "{{ envs_dir }}/{{ platform }}/host_vars/taskwarrior-server.yml"
    - import_tasks: "{{ role_path }}/tasks/provision/main.yml"
```

**provision/main.yml**

```yaml
- name: Execute taskwarrior-server role
  include_role:
    name: 'Taskwarrior server (taskd)'
```

**host_vars/taskwarrior-server.yml**

```yaml
taskd_version: 1.1.0
taskd_tarball_url: "https://github.com/GothenburgBitFactory/taskserver/releases/download/v1.1.0/taskd-1.1.0.tar.gz"
taskd_user: taskd
taskd_group: taskd
taskd_port: 53589
taskd_download_location: "{{ ansible_env.HOME }}/taskd"
taskd_install_location: ~/.local/bin/taskd
taskd_data_location: ~/.cache/taskd
taskd_user_conf_location: ~/.config/taskd/users
taskd_user_cert_location: ~/.config/taskd/users/certs

# Using self signed certificates
taskd_selfsigned: true
taskd_selfsigned_bits: 4096
# Common Name for your certificate (MUST match hostname!)
taskd_selfsigned_cn: "{{ ansible_hostname }}"
taskd_selfsigned_expiration_days: 365
taskd_selfsigned_organization: First Name
taskd_selfsigned_country: FR
taskd_selfsigned_state: Bretagne
taskd_selfsigned_locality: Brest

taskd_selfsigned_clients_download: true
taskd_selfsigned_clients_download_dir: ~/.taskd/
taskd_taskwarrior_config: true
taskd_taskwarrior_config_path: ~/.taskd/taskd.rc
taskd_android_config: true
```

### Default usage

For default usage of this role you only need to define the following, for more advanced usage look at the [Advanced usage](#advanced-usage) section:

```yaml
# Organizations to be created
taskd_organizations:
  - Public
# The users to be created, organization must be in the taskd_organizations variable!
taskd_users:
  - name: First Last
    organization: Public
```

### Advanced usage

For more advanced usage the following variables are available:

```yaml
# Taskd version to install
taskd_version: 1.1.0

# Download url for taskd
taskd_tarball_url: "https://taskwarrior.org/download/taskd-{{ taskd_version }}.tar.gz"

# User taskd daemon runs as
taskd_user: taskd
# Group taskd daemon runs as
taskd_group: taskd
# The port to use for the taskserver (for any port under 1024 the taskserver needs to be run as root, NOT recommended)
taskd_port: 53589
# Download location for taskd tarball
taskd_download_location: "{{ ansible_env.HOME }}/taskd"
# Location to install taskd to
taskd_install_location: /opt/taskd
# Location for taskd data, sets the $TASKDDATA variable (recommended is to NOT put it in your Taskd exec dir)
taskd_data_location: /var/taskd
# Location to save user.conf file to
taskd_user_conf_location: /var/taskd/users
# Location to save user certificates to
taskd_user_cert_location: /var/taskd/users/certs
```

#### SSL Setup

For the SSL setup the following variables are available, the role defaults to
selfsigned certificates. Selfsigned certificates are not recommended for
production use but might be acceptable for personal use.

```yaml
# Set to false to disable the generation of selfsigned certificates
taskd_selfsigned: true

# Bits for SSL certificate (2048 or 4096)
taskd_selfsigned_bits: 4096
# Common Name for your certificate (MUST match hostname!)
taskd_selfsigned_cn: "{{ ansible_hostname }}"
# Days the certificate is valid for
taskd_selfsigned_expiration_days: 365
# Certificate organization
taskd_selfsigned_organization: Göteborg Bit Factory
# Certificate country
taskd_selfsigned_country: SE
# Certificate state
taskd_selfsigned_state: Västra Götaland
# Certificate locality
taskd_selfsigned_locality: Göteborg
```

If you want to use existing certificates you can adapt the following variables
to the correct locations for your setup:

```yaml
# Variables for ssl certs (defaults for selfsigned). Can be used to use signed certificates already stored on the server
taskd_client_cert: "{{ taskd_data_location }}/client.cert.pem"
taskd_client_key: "{{ taskd_data_location }}/client.key.pem"
taskd_server_cert: "{{ taskd_data_location }}/server.cert.pem"
taskd_server_key: "{{ taskd_data_location }}/server.key.pem"
taskd_server_crl: "{{ taskd_data_location }}/server.crl.pem"
taskd_ca_cert: "{{ taskd_data_location }}/ca.cert.pem"
```

To download the certificate files together with configuration to use in your
taskwarrior:

```yaml
taskd_selfsigned_clients_download: true
taskd_selfsigned_clients_download_dir: ~/.taskd/
taskd_taskwarrior_config: true
taskd_taskwarrior_config_path: ~/.taskd/taskd.rc
taskd_android_config: true
```

Then, you can set `include ~/.taskd/taskd.rc` in your `taskrc` file and
taskwarrior should be ready to sync with taskserver.

The android config file will be also placed under
`taskd_taskwarrior_config_path`. You can copy-paste its contents to the setting
of the [android app](https://f-droid.org/en/packages/kvj.taskw/).

License
-------

GPLv3
