# Ansible role for roon-server-docker
This role deploys the Docker image https://hub.docker.com/repository/docker/elgeeko/roon-server to a host and configures it to run at startup.

## Requirements
- ansible
- docker

## Configuring
The variable `roon_server_path` defines the filesystem path on the host to store
roon data, and defaults to `/opt/RoonServer`. This should be a persistent
location in your filesystem for storing the Roon Server configuration and cache.

## Security

### Host network mode (default)
By default, Roon Server will run in a docker container with the `host` network
mode. This is less secure but is a more surefire configuration.

Host network mode is set with the role variable
`roon_network_mode: host`

### Macvlan network mode
Alternately the Roon Server container can run with the `macvlan` network mode,
which is the most secure configuration that still allows Roon to discover RAAT devices like Roon Bridge and Ropieee, which require broadcast and multicast
messaging.

Macvlan network mode is set by the following role variables:
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
`bridge` network mode, which is arguably the most secure as it has the most restrictive network access, however in bridge mode, RAAT devices are not discoverable, as broadcast and multicast messages are not available to the
container.

Bridged network mode is set with the role variable
`roon_network_mode: bridge`

### Privileged execution
This role starts Roon Server in an unprivileged docker container. USB and ALSA devices are mapped from the host into the container to allow Roon Server to use
USB DACs attached to the host. Devices may be hot-swapped without restarting the container.

If you are unable to access your DAC, or if you need a bus other than USB (such
as I2C), you will either need to map the device into the container, or revert to using the insecure docker privileged execution mode.

`roon_container_privileged: true`
