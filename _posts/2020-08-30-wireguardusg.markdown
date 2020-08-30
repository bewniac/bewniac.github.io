---
layout: post
title:  "WireGuard for Ubiquiti Devices"
date:   2020-08-30 10:00:00 +0200
categories: 
---
I've added Wireguard to my USG to route all traffic through Wireguard interface with Mullvad VPN.

Install Wireguard on your USG device with [Wireguards offical USG release.](https://github.com/WireGuard/wireguard-vyatta-ubnt)

```[bash]
set interfaces wireguard wg0 address [IPv4]
set interfaces wireguard wg0 address [IPv6]

set interfaces wireguard wg0 peer [PEER PUB KEY] endpoint [PEER IPv4:PORT]
set interfaces wireguard wg0 peer [PEER PUB KEY] allowed-ips 0.0.0.0/0
set interfaces wireguard wg0 peer [PEER PUB KEY] allowed-ips ::0/0

set interfaces wireguard wg0 private-key [PRIVATE KEY]
set interfaces wireguard wg0 route-allowed-ips false

set interfaces ethernet eth1 firewall in modify [MODIFY_NAME]
set interfaces ethernet eth1 firewall in name LAN_IN
set firewall modify [MODIFY_NAME] rule 20 action modify
set firewall modify [MODIFY_NAME] rule 20 description Wireguard
set firewall modify [MODIFY_NAME] rule 20 modify table 10
set firewall modify [MODIFY_NAME] rule 20 source address [LOCAL_IP_TO_BE_ROUTED_IN_WIREGUARD]
set firewall source-validation disable

set interfaces ethernet eth1 vif 10 firewall in modify [MODIFY_NAME_2]
set interfaces ethernet eth1 vif 10 firewall in name LAN_IN
set firewall modify [MODIFY_NAME_2] rule 10 action modify
set firewall modify [MODIFY_NAME_2] rule 10 description Wireguard
set firewall modify [MODIFY_NAME_2] rule 10 modify table 10
set firewall modify [MODIFY_NAME_2] rule 10 source address [OTHER_LOCAL_IP_TO_BE_ROUTED_IN_WIREGUARD]

set service nat rule 5001 description "Wireguard NAT"
set service nat rule 5001 destination address 0.0.0.0/0
set service nat rule 5001 outbound-interface wg0
set service nat rule 5001 type masquerade

set protocols static table 10 interface-route 0.0.0.0/0 next-hop-interface wg0
```
