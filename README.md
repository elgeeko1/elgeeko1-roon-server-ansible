# Ansible role for roon-server-docker
This role deploys the Docker image https://hub.docker.com/repository/docker/elgeeko/roon-server to a host and configures it to run at startup.

## Requirements
- ansible
- docker

## Configuring

The variable `roon_server_path` defines the filesystem path on the host to store roon data, and defaults to `/etc/roon-server`. This should be a persistent location
in your filesystem for storing the Roon Server configuration and cache.

## Security

### Macvlan network mode (default)
By default, Roon Server will run in an unprivileged docker
container with the `macvlan` network mode. This is the most
secure configuration that still allows Roon to discover RAAT devices like Roon Bridge and Ropieee, which requires broadcast
and multicast messaging.

Macvlan network mode is set with the role variable
`roon_network_mode=macvlan`

Macvlan mode will query Ansible for the 'default'
IPv4 network and configure the Roon Server container
to pull a new IP address from this network. If you
need to customize this behavior, override the defaults
defined in `defaults/main.yml`.

### Host network mode
Alternately, this role can configure the Roon Server
container to run with the `host` network mode, which
is less secure but is a more surefire fallback if the
default configuration for `macvlan` fails.

Host network mode is set with the role variable
`roon_network_mode=host`

### Bridged network mode
Lastly, this role can configure the Roon Server container
to run with the `bridge` network mode, which is arguably
the most secure as it has the most restrictive network
access, however in bridge mode, RAAT devices are not
discoverable, as broadcast and multicast messages are
not available to the container.

Bridged network mode is set with the role variable
`roon_network_mode=bridge`

### Unprivileged execution
This role starts Roon Server in an unprivileged docker
container. USB and ALSA devices are mapped from the host
into the container to allow Roon Server to use USB DACs
attached to the host. Devices may be hot-swapped without
restarting the container.

If you are unable to access your DAC, or if you need a
bus other than USB (such as I2C), you will either need
to map the device into the container, or revert to using
the insecure docker privileged execution mode.
