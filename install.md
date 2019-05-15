# Quick Start Guide - Basic Configuration


Enter configuration mode:
```
vyos@vyos$ configure
vyos@vyos#
```

Configure network interfaces:
```
set interfaces ethernet eth0 address dhcp
set interfaces ethernet eth0 description 'OUTSIDE'
set interfaces ethernet eth1 address '192.168.0.1/24'
set interfaces ethernet eth1 description 'INSIDE'
```

Enable SSH for remote management:
```
set service ssh port '22'
```

Configure Source NAT for our "Inside" network.
```
set nat source rule 100 outbound-interface 'eth0'
set nat source rule 100 source address '192.168.0.0/24'
set nat source rule 100 translation address masquerade
```

Configure a DHCP Server:
```
set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 default-router '192.168.0.1'
set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 dns-server '192.168.0.1'
set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 domain-name 'internal-network'
set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 lease '86400'
```

>= 1.2.0
```
set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 range 0 start 192.168.0.9
set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 range 0 stop '192.168.0.254'
```

<= 1.1.8
```
set service dhcp-server disabled 'false'
set service dhcp-server shared-network-name LAN subnet 192.168.0.0/24 start 192.168.0.9 stop '192.168.0.254'
```

And a DNS forwarder:
```
set service dns forwarding cache-size '0'
set service dns forwarding listen-on 'eth1'
set service dns forwarding name-server '8.8.8.8'
set service dns forwarding name-server '8.8.4.4'
```

Add a set of firewall policies for our "Outside" interface:
```
set firewall name OUTSIDE-IN default-action 'drop'
set firewall name OUTSIDE-IN rule 10 action 'accept'
set firewall name OUTSIDE-IN rule 10 state established 'enable'
set firewall name OUTSIDE-IN rule 10 state related 'enable'
set firewall name OUTSIDE-LOCAL default-action 'drop'
set firewall name OUTSIDE-LOCAL rule 10 action 'accept'
set firewall name OUTSIDE-LOCAL rule 10 state established 'enable'
set firewall name OUTSIDE-LOCAL rule 10 state related 'enable'
set firewall name OUTSIDE-LOCAL rule 20 action 'accept'
set firewall name OUTSIDE-LOCAL rule 20 icmp type-name 'echo-request'
set firewall name OUTSIDE-LOCAL rule 20 protocol 'icmp'
set firewall name OUTSIDE-LOCAL rule 20 state new 'enable'
set firewall name OUTSIDE-LOCAL rule 30 action 'drop'
set firewall name OUTSIDE-LOCAL rule 30 destination port '22'
set firewall name OUTSIDE-LOCAL rule 30 protocol 'tcp'
set firewall name OUTSIDE-LOCAL rule 30 recent count '4'
set firewall name OUTSIDE-LOCAL rule 30 recent time '60'
set firewall name OUTSIDE-LOCAL rule 30 state new 'enable'
set firewall name OUTSIDE-LOCAL rule 31 action 'accept'
set firewall name OUTSIDE-LOCAL rule 31 destination port '22'
set firewall name OUTSIDE-LOCAL rule 31 protocol 'tcp'
set firewall name OUTSIDE-LOCAL rule 31 state new 'enable'
```

Apply the firewall policies:
```
set interfaces ethernet eth0 firewall in name 'OUTSIDE-IN'
set interfaces ethernet eth0 firewall local name 'OUTSIDE-LOCAL'
```

Commit changes, save the configuration, and exit configuration mode:
```
vyos@vyos# commit
vyos@vyos# save
Saving configuration to '/config/config.boot'...
Done
vyos@vyos# exit
vyos@vyos$
```
