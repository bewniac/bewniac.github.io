---
layout: post
title:  Zero trusting my home - part 2
date:   2019-07-15 15:14:24 +0200
categories: networking zerotrust
---
## Certificate Authority (OpenSSL)
The most important part in a zero trust network is the identification of devices and users. We can identify a device using x509 certificates. Depending on the content of the certificate it can provide metrics to better make descisions on whether or not the device is allowed access to resources. This is not a very common thing to run in a home environment (in my experience) but there's a lot of security benifits of doing so. Like running 802.1X on your LAN and WLAN or using mutual TLS towards webb services.

First of all, we need [OpenSSL](https://www.openssl.org/). Which comes pre-installed on most linux distributions. I'm running it on Ubuntu 18.04 LTS. Then we need to create our private key and our CA certificate. I created a new directory under `/etc/ssl` called `home_certs` to put my signed certificates. And a directory called `private` which will store my private key. The CA need some files to work, a file called `serial` which contains the latest serial number to keep track of the certificates, and an `index.txt` file. Both files are put in a folder I call `CA` under `/etc/ssl`. The `serial` file should contain a number which could be incremented, I just use `01` as my first serial number. 
```
root@server$ mkdir /etc/ssl/CA/
root@server$ mkdir /etc/ssl/home_certs/
root@server$ echo "01" > /etc/ssl/CA/serial
root@server$ touch /etc/ssl/CA/index.txt
```
Now we create our CA certificate and private key using the OpenSSL `req` command.
```
root@server$ openssl req -new -x509 -extensions v3_ca -keyout cakey.pem -out cacert.pem -days 3650
root@server$ mv cacert.pem /etc/ssl/certs/
root@server$ mv cakey.pem /etc/ssl/private/
```
Next we need to exit the configuration file for OpenSSL so it knows which files to use. We edit the file `/etc/ssl/openssl.cnf` and point the configuration to use the files we created, along with the private key and the CA certificate.
```
--- SNIP ---
[ CA_default ]
dir		= /etc/ssl		# Where everything is kept
certs		= $dir/certs		# Where the issued certs are kept
crl_dir		= $dir/crl		# Where the issued crl are kept
database	= $dir/CA/index.txt	# database index file.
#unique_subject	= no			# Set to 'no' to allow creation of
					# several certs with same subject.
new_certs_dir	= $dir/home_certs	# default place for new certs.

certificate	= $dir/certs/cacert.pem	# The CA certificate
serial		= $dir/CA/serial	# The current serial number
crlnumber	= $dir/crlnumber	# the current crl number
					# must be commented out to leave a V1 CRL
crl		= $dir/crl.pem		# The current CRL
private_key	= $dir/private/cakey.pem# The private key
RANDFILE	= $dir/private/.rand	# private random number file
--- SNIP ---
```
To sign a certificate request, just run the following and the certificate will be stored in `/etc/ssl/home_certs/[SERIAL].pem` where \[SERIAL\] is the current serial number of signed certificates.
```
root@server$ openssl ca -in server.csr -config /etc/ssl/openssl.cnf
```
Now import the cacert.pem to your devices and you're good to go! Certificates for everyone!
### IMPORTANT NOTE
I'm no expert in this area. And it would be very helpful if anyone reading this with more knowledge of how to do this properly and generate good enough certicates and private keys. Preferably using elliptic curves? And also, I will in the future aqcuire a YubiHSM2 to store my CA private key, but that is out of scope for now. And since my home environment is not on any target-list of anyone other than opportunistic malware authors I think this is good enough.  

## Nginx Reverse Proxy
I don't have a lot of web-services running in my home. But I do have a unifi software controller for my WLAN and a synology NAS. I want to proxy all web-traffic through Nginx. This will help my goal to authenticate the traffic flows if the traffic transports through the same place. Below is an example config I used. 

```
server {
		listen 443 ssl;
		server_name unifi.hackad.lan;
		ssl_certificate /etc/nginx/ssl/certs/unifi.hackad.lan.crt;
		ssl_certificate_key /etc/nginx/ssl/private/unifi.key;
		ssl_client_certificate /etc/nginx/ssl/certs/cacert.pem;
		ssl_protocols TLSv1.2;
		ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
		ssl_prefer_server_ciphers on;
		ssl_verify_client on;
		proxy_ssl_verify off;
		location / {
			proxy_pass https://192.168.1.1:8443/;
			proxy_redirect      off;
			proxy_set_header    Host                    $host;
			proxy_http_version	1.1;
			proxy_set_header Upgrade $http_upgrade;
			proxy_set_header Connection "upgrade";
		}
	}
```
I have a ECDSA certificate signed with my own CA, thats why the `ssl_ciphers` are defined to support ECDSA. The `proxy_ssl_verify off;` line don't require the SSL to be verified on the target. Since the targets are using self-signed certificates. This will change in the future, I want all devices to provice their own certificate for identification signed by my CA. But for this example, they got self-signed certificates. 

The proxy headers (`proxy_set_header`) can be very important depending on the web-service. For my synology I can ommit almost all headers, for my Unifi Controller I have to use the `Upgrade`, `Connection` and `Host` headers for the proxy to work. We want the proxy to pass the traffic back and forth, but we want it to act in the same way as when the client are making the requests directly to the site. There be several issues regarding Cross-Origin resource sharing (CORS) and maybe other HTTP och browser security measures. The `Upgrade` and `Connection` headers are used to support web-sockets (Unifi Controller using websockets). 

### Mutual TLS
The line `ssl_verify_client on;` enables the mutual TLS. With this line my device (browser) have to present a valid certificate signed by my CA in order to access the web-services. The line `ssl_client_certificate PATH;` holds the path to the CA certificate and will be used to validate my client certificate upon request.  
