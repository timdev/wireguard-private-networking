# Manage a network of wireguard servers and clients.

## This is a fork

This role is now a fork of `mawalu.wiregaurd_private_networking`, with expanded support for managing client configs.

My use-case is different than the upstream.  In my case, I'm mostly interested in networking developer VMs in various
locations around the world (wherever the developer happens to be sitting), and allowing developers (and other 
stakeholders) to participate in the dev-vm network from their primary devices, so they can browse web apps running on 
a remote dev's VM, or SSH into a remote VM to help troubleshoot.

To support this, I need to not only provision/configure one or more wireguard "servers" (with static, routable public 
IPs), but also several (or even "many") wireguard config files that can be distributed to individuals to use on their
local machines.

Most of the changes aren't documented yet.

At some point, it would be nice to merge this back upstream. However, it's not clear when I'll have time to think it
through, nor is it clear if I've gone too far out of the scope @mawalu wants to address. 



## How

The role installs [wireguard](https://wireguard.com) on Debian or Ubuntu, creates a mesh between all servers by adding them all as peers and configures the wg-quick systemd service.

## Installation

Installation can be done using [ansible galaxy](https://galaxy.ansible.com/mawalu/wireguard_private_networking):

```
$ ansible-galaxy install mawalu.wireguard_private_networking
```

## Setup

Install this role, assign a `vpn_ip` variable to every host that should be part of the network and run the role. Plese make sure to allow the VPN port (default is 5888) in your firewall. Here is a small example configuration:

```yaml
# inventory host file

wireguard:
  hosts:
    1.1.1.1:
      vpn_ip: 10.1.0.1/32
    2.2.2.2:
      vpn_ip: 10.1.0.2/32

```

```yaml
# playbook

- name: Configure wireguard mesh
  hosts: wireguard
  remote_user: root
  roles:
    - mawalu.wireguard_private_networking
```

```yaml
# playbook (with client config)
- name: Configure wireguard mesh
  hosts: wireguard
  remote_user: root
  vars:
    client_vpn_ip: 10.1.0.100
    client_wireguard_path: "~/my-client-config.conf"
  roles:
    - mawalu.wireguard_private_networking
```

## Additional configuration

There are a small number of role variables that can be overwritten.

```yaml
wireguard_port: "5888" # the port to use for server to server connections
wireguard_path: "/etc/wireguard" # location of all wireguard configurations

wireguard_network_name: "private" # the name to use for the config file and wg-quick

debian_enable_testing: true # if the debian testing repos should be added on debian machines
debian_pin_packages: true # if the pin configuration to limit the use of unstable repos should be created on debian machines

client_vpn_ip: "" # if set an additional wireguard config file will be generated at the specified path on localhost
client_wireguard_path: "~/wg.conf" # path on localhost to write client config, if client_vpn_ip is set 

# a list of additional peers that will be added to each server
wireguard_additional_peers:
  - comment: martin
    ip: 10.2.3.4
    key: your_wireguard_public_key
  - comment: other_network
    ip: 10.32.0.0/16
    key: their_wireguard_public_key
    keepalive: 20 
    endpoint: some.endpoint:2230 

wireguard_post_up: "iptables ..." # PostUp hook command
wireguard_post_down: "iptables"   # PostDown hook command
```

## Contributing

Feel free to open issues or MRs if you find problems or have ideas for improvements. I'm especially open for MRs that add support for more operating systems.

