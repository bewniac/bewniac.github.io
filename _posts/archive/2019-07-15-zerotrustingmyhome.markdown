---
layout: post
title:  Zero trusting my home
date:   2019-07-15 15:14:24 +0200
categories: networking zerotrust
---
Zero trust network is a concept with the goal to remove the old and outdated perimeter model all together. The perimeter model in networking design traditionally defines the Internet as an untrusted network which is separated from the trusted corporate networks by a perimiter. The perimiter is usually some kind of firewall or proxy. 

```
 ##########       #           #      #
# Internet # <--> # Perimeter # <--> # Corporate trusted networks
 ##########       #           #	     # 
```
Zero trust defines all networks as untrusted and is placing the trust on the device and user. A device could be anything connected to a network, a client, server, IoT device, printer, smartphone etc. Simply put the user and device requires to be authenticated and the network flow accessing any resource on the network need to be authorized. A zero trust network collects information about devices and users to build a trust score. The trust score is used to evaluate if a user and device pair are authorized to access a network resource. The connection to any resource is only available to the device if the trust score is equeal to or over the limit required to access the resource, the zero trust network opens access to resources dynamically. This means that no resources are available for an unathenticated device or user. 

This is a very brief explaination of zero trust networks. I will write a separate post for each and every component in a zero trust network. My goal is to define zero trust networks and how they could be implemented using open source tools and well-known technologies, as well as learning some new things along the way.

*Concepts*
1. Trust policy
2. Trust score
3. Authentication
4. Authorization
5. Single Sign-On
6. Secure storage

*Components*
1. Inventory Database
2. Agent
3. Policy engine
4. Trust engine
5. Enforcer 
6. Logging & monitoring
7. Private Public Key Infrastructure
8. Credential vault

If anyone reading this have any good resources, comments (all feedback are welcome!), other ideas or just think that this might be fun to collaborate on don't hesitate to hit me up on [twitter](https://www.twitter.com/bewniac) or send me an email. 

# Zero trusting my home, start with basics

So I've started my migration in my home environment from a perimeter based model to the zero trust. By first of all implementing some services like DNS, reverse proxy and a certificate authority. My goal is to make this transition as easy and transparent as possible because my fiance do not enjoy me making it harder to access network resources. First of all, let's try to deply a DNS server.

## DNS Server 
I've chosen to make my own life harder for myself and installed bind as my DNS server. [Bind](https://www.isc.org/bind/) got a lot of features and could be a pain to configure what I've heard, but for a home environment the setup and configuration wasn't that difficult. I've run through what I did step by step. 

First of all, install bind. I'm running on an Ubuntu Server 18.04 LTS.
```
root@server$ apt install bind9 bind9utils
```
When the installation has finished you should've all configuration files in `/etc/bind/`. First of all we're interested in two files, `named.conf.options` and `named.conf.local`. We start editing `named.conf.option`.

```
acl "trusted" {
	localhost;
	192.168.10.0/24;
	192.168.20.0/24;
	192.168.30.0/24;
	192.168.40.0/24;
};

options {
	directory "/var/cache/bind";

	recursion yes;
	allow-recursion {trusted;};
	listen-on port 53 { localhost;}; 
	allow-query { trusted; };
	allow-transfer { none; };

	// If there is a firewall between you and nameservers you want
	// to talk to, you may need to fix the firewall to allow multiple
	// ports to talk.  See http://www.kb.cert.org/vuls/id/800113

	// If your ISP provided one or more IP addresses for stable 
	// nameservers, you probably want to use them as forwarders.  
	// Uncomment the following block, and insert the addresses replacing 
	// the all-0's placeholder.

	forwarders {
		1.1.1.1;
		1.0.0.1;
	};

	//========================================================================
	// If BIND logs error messages about the root key being expired,
	// you will need to update your keys.  See https://www.isc.org/bind-keys
	//========================================================================
	dnssec-validation auto;

	auth-nxdomain no;    # conform to RFC1035
	listen-on-v6 { none; };
};
```
The `acl` section defines the networks allowed to query the bind server. You only need to put the subnets in your home environment that should be able to query the DNS server (for now, this will probably be changed when I'm migrating to the zero trust model since no networks are trusted). Also, only the trusted networks are permitted to make recursive queries. And since I don't have any redundancy at home, I don't have the need for transfer. 

The `forwarders` section defines your DNS forwarders, the ones in the example are CloudFlares servers. I'm using [mullvad](https://www.mullvad.net)s DNS servers since I'm also running theirs [Wireguard](https://www.wireguard.com/) setup to route all my traffic through a mullvad VPN tunnel. 

Last I disable IPv6, I'm not running it in my home environment.

Next we will edit the `named.conf.local` file and add our zones. I've created a directory in `/etc/bind/` called `zones` where I put the zone files which are defined in `named.conf.local` below.
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "hackad.lan" {
	type master;
	file "/etc/bind/zones/db.hackad.lan";
};
zone "168.192.in-addr.arpa" {
	type master;
	file "/etc/bind/zones/db.168.192";
};
```
Both the forward-lookup zone and reverse-lookup zone are defined here with the absolute path to both. For this example they're named `db.hackad.lan` and `db.168.192`. Now let's create our zone files. For our forward lookup zone, we can copy the file `/etc/bind/db.local` to `/etc/bind/zones/db.hackad.lan`. The file will look like this:
```
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	localhost. root.localhost. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	localhost.
@	IN	A	127.0.0.1
@	IN	AAAA	::1
```
Now we can edit the zone file and add the records we want. For the purpose of this example, something like this:
```
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	hackad.lan. admin.hackad.lan. (
			      2 	; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
; NS Records
@	IN	NS	ns.hackad.lan.
; A Records
ns.hackad.lan.	IN	A	192.168.1.1
webb.hackad.lan.	IN	A	192.168.20.10
pi.hackad.lan.		IN	A	192.168.20.20
```
We define the SOA record first. The @ translates to hackad.lan, which is the zone name. The `hackad.lan` in the SOA record is the primary master name server for the zone and the `admin.hackad.lan` is a e-mail address using a dot instead of @ since the @ character is short for the zone name. The serial is used by a slave name server to know if the zone has been updated. I don't have to increase this value since there's no zone-transfers going on. But if your setup involves slaves, then you should increase this number every time you make a change to the zone file. Next is the NS record, which defines the name server for the zone. And then the A records used to translate FQDNs to IP addresses.

Now we do the same thing for the reverse lookup zone. We copy the file `/etc/bind/db.local` to `/etc/bind/zones/db.168.192`. I won't be elaborating the records to much. But instead to define A records you define PTR records instead. Which could translate IP addresses to FQDN.
```
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	hackad.lan. admin.hackad.lan. (
			      2		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
; NS Records
@	IN	NS	ns.hackad.lan.
; PTR Records
1.1	IN	PTR	ns.hackad.lan.
20.10	IN	PTR	webb.hackad.lan.
20.20	IN	PTR	pi.hackad.lan.
```

Now we can check our configuration and our zone files using the `named-checkzone` and `named-checkconf` commands, like this.
```
root@server$ named-checkzone hackad.lan zones/db.hackad.lan 
zone hackad.lan/IN: loaded serial 2
OK
root@server$ named-checkconf named.conf
root@server$
```
Next I need to define my iptables rules to permit DNS traffic to the DNS server. Also, if you have any outbound restrictions you need to open from the DNS servers IP to the forwarders IP addresses.  
```
iptables -N INPUT:BIND9
iptables -A INPUT:BIND9 -d 192.168.1.1/32 -p udp -m udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT:BIND9 -d 192.168.1.1/32 -p tcp -m tcp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -j INPUT:BIND9
```
Edit the file `/etc/default/bind9` and add edit the line `OPTIONS="-u bind"` to `OPTIONS="-u bind -4"`. This is used by the bind server to only listen on IPv4 addresses.

Check if the service is active:
```
root@server$ systemctl status bind9
● bind9.service - BIND Domain Name Server
   Loaded: loaded (/lib/systemd/system/bind9.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2019-07-15 10:06:24 UTC; 3h 27min ago
     Docs: man:named(8)
  Process: 9504 ExecStop=/usr/sbin/rndc stop (code=exited, status=0/SUCCESS)
 Main PID: 9507 (named)
    Tasks: 11 (limit: 4915)
   CGroup: /system.slice/bind9.service
           └─9507 /usr/sbin/named -f -u bind -4
```
Enable it to run on boot:
```
root@server$ systemctl enable bind9
```
And try it out:
```
user@client$ dig +short webb.hackad.lan
192.168.20.10
```
Don't forget to change your DHCP server configuration to give your clients the new DNS server with their DHCP response.

There we go, a bind server running in your home environment. Next I will write posts about setting up your own Certicate Authority, a Nginx reverse proxy and mutual TLS! 