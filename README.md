# Ansible role for roon-server-docker
This role deploys the Docker image [elgeeko/roon-server](https://hub.docker.com/repository/docker/elgeeko/roon-server) to a host and configures it to run at startup. An optional Samba server configures
samba shares for music and for Roon backup.

# Requirements
Provisioning host:
- ansible 2.9 or later
- ansible collection ansible.netcommon (`ansible-galaxy install ansible.netcommon`)

Host that will run docker:
- Ubuntu 18.04 or later
- docker
 - An ansible role for installing docker is available from https://github.com/elgeeko1/elgeeko1-docker-ansible.git

# How to use this role
### Method 1: Install using ansible-galaxy

The most robust way to install this role is to use ansible-galaxy,
which is installed with ansible by default. ansible-galaxy is effectively a package manager for ansible. It installs roles
from the community, or from private git repos, into your local machine for use in your playbooks.

Start by adding this role to an ansible-galaxy dependency file. Typically this file lives alongside your playbook file and by convention is named `requirements.yml`.

Add the following section to `requirements.yml`:

```
roles:
  - name: elgeeko1-roon-server-ansible
    src: https://github.com/elgeeko1/elgeeko1-roon-server-ansible
    version: main
  - name: ansible.netcommon
```

If there is already a `roles` section, simply append this role to
add it as a dependency.

Then, install the role using ansible-galaxy:

`ansible-galaxy install -r requirements.yml -v`

### Method 2: Clone this repository into a `roles` directory

Use this method if you plan to modify this role, or if for some
reason the ansible-galaxy method fails.

Starting from your playbook directory, change into the `roles`
directory and clone:

```
$ ls
 > playbook.yml
 > roles/
$ cd roles/
roles$ git clone https://github.com/elgeeko1/elgeeko1-roon-server-ansible
```

# Configuring
The variable `roon_server_path` defines the filesystem path on the host to store
roon data, and defaults to `/opt/RoonServer`. This should be a persistent
location in your filesystem for storing the Roon Server configuration and cache.

If you wish to add additional samba mounts, create a template file
and set the variable `roon_samba_conf_appendix_template` to its path.

Modify `smb.conf.j2` in the templates folder to customize Samba.
The default username and password is `music:music`.

You may map additional volumes using the docker volume mapping syntax into the
samba container with the variable `roon_samba_extra_volumes`. The variable is
a list and defaults to `[]`.

# Security

### Host network mode (default)
By default, Roon Server will run in a docker container with the `host` network
mode. This is less secure but is a more surefire configuration.

Host network mode is set with the role variable
`roon_network_mode: host`

### Macvlan network mode
Alternately the Roon Server container can run with the `macvlan` network mode,
which is the most secure configuration that still allows Roon to discover
all RAAT devices.

*Warning* macvlan network mode does not work over wireless connections.

macvlan network mode is set by the following role variables:
```
roon_network_mode: macvlan
roon_network_macvlan:
  name:         "roon"
  subnet:       "192.168.1.0/24"
  gateway:      "192.168.1.1"
  interface:    "{{ ansible_default_ipv4.interface }}"
  ipv4_address: "192.168.1.50" # IP address of Roon Server
```

### Bridged network mode
Lastly, this role can configure the Roon Server container to run with the
`bridge` network mode, which is arguably the most secure as it has the most
restrictive network access, however in bridge mode, RAAT devices that require
broadcast and multicast messaging are not discoverable. You'll have to
experiment to see what works.

Discoverable endpoints when Roon Server is running in bridge mode:
- Roon Server
- Roon Bridge
- ropieee

Non-discoverable endpoints when Roon Server is running in bridge mode:
- HiFiBerryOS

Bridged network mode is set with the role variable
`roon_network_mode: bridge`

### Privileged execution
This role starts Roon Server in an unprivileged docker container. USB and ALSA
devices are mapped from the host into the container to allow Roon Server to use
USB DACs attached to the host. Devices may be hot-swapped without restarting the container.

If you are unable to access your DAC, or if you need a bus other than USB (such
as I2C), you will either need to map the device into the container, or revert to
using the insecure docker privileged execution mode.

`roon_container_privileged: true`

# Mounting network shares
Mounting cifs shares from a docker container either requires RoonServer
to run as root, or for fstab to be configured in advance of starting
the container and configuring permissions of the container users.
The simplest workaround to support this comes at the unfortunate cost
of security, and is to set `roon_container_user` to `root`.
