---
layout: post
title:  "Simple one-liners and scripts"
date:   2019-06-19 09:34:54 +0200
categories: bash scripting oneliners
---
Get CVE info from CVE details in text form. 

```
#!/bin/bash
CVE=$1; 
curl -s https://cve.mitre.org/data/downloads/allitems.txt.gz | pigz -dc | awk \'/$CVE/{y=1} /====================/{y=0}y\'
```