---
layout: post
title:  Open Zero Trust - Introduction
date:   2019-07-05 15:14:24 +0200
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

### Table of Contents: 
#### Concepts
1. Policy
2. Authentication
3. Authorization
#### Components
1. Inventory Database
2. Agent
3. Policy engine
4. Trust engine
5. Enforcer 
6. Logging & monitoring

If anyone reading this have any good resources, comments (all feedback are welcome!), other ideas or just think that this might be fun to collaborate on don't hesitate to hit me up on [twitter](https://www.twitter.com/bewniac) or send me an email. 


# Resources
- [https://www.nodeprotect.com/](https://www.nodeprotect.com/)
- [https://cloud.google.com/beyondcorp/](https://cloud.google.com/beyondcorp/)
- [https://www.amazon.com/Zero-Trust-Networks-Building-Untrusted/dp/1491962194](https://www.amazon.com/Zero-Trust-Networks-Building-Untrusted/dp/1491962194)
- [https://www.microsoft.com/security/blog/2018/06/14/building-zero-trust-networks-with-microsoft-365/](https://www.microsoft.com/security/blog/2018/06/14/building-zero-trust-networks-with-microsoft-365/)
- [https://duo.com/product](https://duo.com/product)
- [https://osquery.readthedocs.io](https://osquery.readthedocs.io)