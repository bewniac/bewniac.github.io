---
layout: post
title:  Open Zero Trust
date:   2019-06-19 15:14:24 +0200
categories: networking zerotrust
---

This is just an idea I'm having so bare with me. I'm a big believer in zero trust networks and really think that everybody should  implement these concepts in their organizations. But the concepts are fairly new and haven't been widely developed yet hence there's no standards or guidelines on how to implement zero trust using common methods. 

My idea is to create an open source project, called Open Zero Trust, and try to define the policies, components and implementation techniques as modular and cross-platform as possible using well-known technologies. If anyone reading this have any good resources, other ideas or just think that this might be ful to collaborate on don't hesitate to hit me up on [twitter](https://www.twitter.com/bewniac).

My brainstorm starts here.
# Open Zero Trust
I can see some requirements going through all the different components:
- Modular
- Fast
- Easy-deployable
- Scalable

## Policy 
- Scoring system
   - Untrusted 0 < Score < 1 Trusted
- Whitelisting (Defines resources in environment)
- Multiple policies throughout environment
   - Policy on business units
- JSON 
- Scoring components
   - Security instruments
      - Antivirus
      - Disk encryption
      - Multi-factor
      - Patch level
   - Operating system
   - Device
   - Location
   - Authentication
   - 

## Components
### Policy engine 
### Trust engine
### Enforcer
### Agent
# On-premise
# Cloud


# Resources
- [https://cloud.google.com/beyondcorp/](https://cloud.google.com/beyondcorp/)
- [https://www.amazon.com/Zero-Trust-Networks-Building-Untrusted/dp/1491962194](https://www.amazon.com/Zero-Trust-Networks-Building-Untrusted/dp/1491962194)
- [https://www.microsoft.com/security/blog/2018/06/14/building-zero-trust-networks-with-microsoft-365/](https://www.microsoft.com/security/blog/2018/06/14/building-zero-trust-networks-with-microsoft-365/)
- [https://duo.com/product](https://duo.com/product)
