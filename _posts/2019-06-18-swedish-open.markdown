---
layout: post
title:  "Swedish Open!"
date:   2019-06-18 10:34:54 +0200
categories: hacking scanning port dns rpd masscan
---
# Swedish Open
NOTE! The scans are done solely in the purpose of research. Any issues I find have been reported in a responsible manner and no sensitive data is stored. I suggest to all organisations to keep track of their Internet connected services. Scanning their own IP blocks (IPv4 and IPv6). You can reach out to me at twitter [@bewniac](https://twitter.com/bewniac).

## DNS udp 53
I'm trying to get more into DNS and wanted to do some research on open resolvers in Sweden. First I wanted to get a list of all Swedish registred IP blocks, which is listed [here](http://ipverse.net/ipblocks/data/countries/se.zone). I haven't checked it against other lists and it might now be fully complete but for my purpose it's good enough. 

So first I want to get all servers running DNS (UDP port 53). I used [masscan](https://github.com/robertdavidgraham/masscan), which is a really fast Internet port scanner. 

By setting the rate to 100 000 I can scan the Swedish IP blocks in under five minutes. 
```
masscan --rate=100000 -pU:53 -iL se.zone -oA sweden.dns.servers
grep "open" sweden.dns.servers | awk '{print $4}' > sweden.dns.ip
```

Next I wanted to find which servers are open resolvers. You can do this by sending a TXT query for "test.openresolver.com" to the nameserver. I wrote a simple python script to do this. 

```
#!/usr/bin/python3
import dns.resolver
import socket
import sys

if len(sys.argv) < 3:
	print("Usage : " + str(sys.argv[0]) + " [IP list of DNS servers to check] [output file]")
	exit()

f = open(sys.argv[1], "r");
out = open(sys.argv[2], "w")
dns.resolver.default_resolver = dns.resolver.Resolver(configure=False)
dns.resolver.default_resolver.timeout = 1
dns.resolver.default_resolver.lifetime = 1
for ip in f:
	print("Trying server: " + ip);
	try:
		dns.resolver.default_resolver.nameservers = [socket.gethostbyname(ip)]
		answer = dns.resolver.query("test.openresolver.com", "TXT")
	except:
		continue

	if answer[0]:
		out.write(ip)

f.close()
out.close()
```
Now I got a list with all open DNS resolvers in Sweden. So what can I do with this information?

### AXFR Transfer
AXFR is an old method to sync zone files between nameservers. This shouldn't be possible to do from Internet. In some misconfigurations this unfortunately true and anyone can ask for the zonefile for a specific domain. I ran some tests to check if there was possible to transfer the zonefile. 

First I did a reverse lookup to see which domain the nameservers are in. They can have multiple domains, but this is just a simple check-up. 
```
for ns in `cat sweden.open.resolvers`; do rev=`dig +time=1 +short +answer -x $ns @$ns`; echo $ns" "$rev done | tee sweden.open.resolvers.reverse
```

Filter out the top domain and the domain name with the IP address. 
```
awk -F'[ ]' 'NF == 3 {print $1" "$3}' sweden.open.resolvers.reverse | awk -F'[ .]' '{print $1"."$2"."$3"."$4" "$(NF-2)"."$(NF-1)}' > sweden.openresolvers.ns.domain
```

Let's see if zone-transfer is enabled. 

```
awk '{print "dig +time=1 +short -t axfr @"$1" "$2}' sweden.openresolvers.ns.domain > axfr.commands
while read cmd; do echo $cmd; $cmd; done <axfr.commands > axfr.result
```
### Other findings
When I did the reverse lookup I found a lot of A records named `asus.router.com`, `dsldevice.lan`, `router.lan` and other `.lan` devices which tells me they're not supposed to be exposed on the internet. 

## RDP tcp 3389
On thing about Remote Desktop Protocol (RDP) is that it should never ever be accessible from the Internet. So let see how many RDP servers are running in Sweden accessible to Internet. 

```
masscan --rate=100000 -p3389 -iL se.zone -oG RDP/sweden.3389.scan
```

The result shows there's almost 2 000 servers running RDP in Sweden. 

I'm not comfortable to do anything more with this. You could take screen shots of the login screens to gather intel about users. You could start bruteforcing passwords. One extremely bad vulnerability was disclosed this year (2019) called `BlueKeep` or [CVE-2019-0708](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-0708). It already exists metasploit modules for checking if the system is vulnerable and there will probably be more PoC's actually exploiting this vulnerability in the wild. Watch out!
