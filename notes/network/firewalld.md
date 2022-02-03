# Firewalld

This page is extracted from a [TUTORIAL](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-8).

## Basic Concepts in firewalld

### Zones

The `firewalld` daemon manages groups of rules using entities called zones. Zones are sets of rules that dictate what traffic should be allowed depending on the level of trust you have in the network.

Network interfaces are assigned to a zone to dictate the behavior that the firewall should allow.

The predefined zones within `firewalld` are, in order from least trusted to most trusted:

- drop: The lowest level of trust. All incoming connections are dropped without reply and only outgoing connections are possible.
- ......

To use the firewall, we can create rules and alter the properties of our zones and then assign our network interfaces to whichever zones are most appropriate.

### Rule Permanence
In firewalld, rules can be applied to the current runtime ruleset, or be made permanent.

Most `firewall-cmd` operations can take a `--permanent` flag to indicate that the changes should be applied to the permenent configuration. Additionally, the currently running firewall can be saved to the permanent configuration with the `firewall-cmd --runtime-to-permanent` command.

## Getting Familiar with the Current Firewall Rules

### Exploring the Defaults
~~~
$ sudo firewall-cmd --get-default-zone
public
$ sudo firewall-cmd --get-active-zones
docker
  interfaces: br-81a402442f7d docker0
public
  interfaces: eno1 eno2
~~~

~~~
$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eno1 eno2
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
~~~

We can tell from the output that this zone is both the default and active, and that the eno1 and eno2 interfaces are associated with this zone.

However, we can also see that this zone allows traffic for a DHCP client ( for IP address assignment ), SSH ( for remote administration ), and Cockpit ( a web-based console ).

### Adding a Service to your Zones

~~~
$ sudo firewall-cmd --zone=public --add-service=http
~~~