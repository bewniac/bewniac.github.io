---
layout: post
title:  "gocryptfs"
date:   2020-08-27 02:00:00 +0800
categories: 
---
Looking for something commandline to help me mount encrypted folders for projects. Veracrypt and ecryptfs were two options. But then I found gocryptfs which in syntax looks more like encfs, simple as shit, and seems to be doing it fast. Based on Go's FUSE library. Good encryption methods, AES-256-GCM. From their Security design docs: "gocryptfs builts upon well-known cryptographic primitives: scrypt for key derivation, AES-GCM for file content encryption and, as a world's first for encrypted filesystems, EME wide-block encryption for file name encryption.".

[https://github.com/rfjakob/gocryptfs](https://github.com/rfjakob/gocryptfs)

```[bash]
bash$ apt install gocryptfs

gocryptfs 1.7.1; go-fuse 0.0~git20190214.58dcd77; 2019-12-26 go1.13.5 linux/amd64

Usage: gocryptfs -init|-passwd|-info [OPTIONS] CIPHERDIR
  or   gocryptfs [OPTIONS] CIPHERDIR MOUNTPOINT

Common Options (use -hh to show all):
  -aessiv            Use AES-SIV encryption (with -init)
  -allow_other       Allow other users to access the mount
  -i, -idle          Unmount automatically after specified idle duration
  -config            Custom path to config file
  -ctlsock           Create control socket at location
  -extpass           Call external program to prompt for the password
  -fg                Stay in the foreground
  -fusedebug         Debug FUSE calls
  -h, -help          This short help text
  -hh                Long help text with all options
  -init              Initialize encrypted directory
  -info              Display information about encrypted directory
  -masterkey         Mount with explicit master key instead of password
  -nonempty          Allow mounting over non-empty directory
  -nosyslog          Do not redirect log messages to syslog
  -passfile          Read password from file
  -passwd            Change password
  -plaintextnames    Do not encrypt file names (with -init)
  -q, -quiet         Silence informational messages
  -reverse           Enable reverse mode
  -ro                Mount read-only
  -speed             Run crypto speed test
  -version           Print version information
  --                 Stop option parsing


bash$ gocryptsfs -init EncryptedFolder
bash$ gocryptfs EncryptedFolder plainTextFolder
bash$ umount plainTextFolder
```
