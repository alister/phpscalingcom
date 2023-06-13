---
title: 'DNS resolvers OR you can''t beat the speed of light'
date: Wed, 23 Nov 2011 12:53:55 +0000
draft: true
tags: ['dns', 'linux', 'tools']
---

http://www.manu-j.com/blog/opendns-alternative-google-dns-rocks/403/ has a DNS-speed testing script. I added my local DNS resolver to the list of DNS being tested, and got the following:

Google (8.8.8.8) Level 3 (4.2.2.2 ) OpenDNS (208.67.222.222 My own local network (192.168.1.94)

4.2.2.2 lifehacker.com 170 msec 8.8.8.8 lifehacker.com 24 msec 208.67.222.222 lifehacker.com 18 msec 192.168.1.94 lifehacker.com 0 msec

4.2.2.2 lifehacker.com 170 msec 8.8.8.8 lifehacker.com 24 msec 208.67.222.222 lifehacker.com 18 msec 192.168.1.94 lifehacker.com 0 msec

4.2.2.2 facebook.com 31 msec 8.8.8.8 facebook.com 24 msec 208.67.222.222 facebook.com 19 msec 192.168.1.94 facebook.com 0 msec

4.2.2.2 manu-j.com 31 msec 8.8.8.8 manu-j.com 24 msec 208.67.222.222 manu-j.com 18 msec 192.168.1.94 manu-j.com 0 msec

4.2.2.2 reddit.com 31 msec 8.8.8.8 reddit.com 26 msec 208.67.222.222 reddit.com 17 msec 192.168.1.94 reddit.com 306 msec

4.2.2.2 tb4.fr 31 msec 8.8.8.8 tb4.fr 23 msec 208.67.222.222 tb4.fr 18 msec 192.168.1.94 tb4.fr 52 msec

4.2.2.2 bbc.co.uk 31 msec 8.8.8.8 bbc.co.uk 24 msec 208.67.222.222 bbc.co.uk 17 msec 192.168.1.94 bbc.co.uk 41 msec

Second run: 4.2.2.2 lifehacker.com 31 msec 8.8.8.8 lifehacker.com 25 msec 208.67.222.222 lifehacker.com 18 msec 192.168.1.94 lifehacker.com 0 msec

4.2.2.2 facebook.com 32 msec 8.8.8.8 facebook.com 26 msec 208.67.222.222 facebook.com 18 msec 192.168.1.94 facebook.com 0 msec

4.2.2.2 manu-j.com 31 msec 8.8.8.8 manu-j.com 25 msec 208.67.222.222 manu-j.com 17 msec 192.168.1.94 manu-j.com 0 msec

4.2.2.2 reddit.com 32 msec 8.8.8.8 reddit.com 1857 msec 208.67.222.222 reddit.com 19 msec 192.168.1.94 reddit.com 293 msec

4.2.2.2 tb4.fr 30 msec 8.8.8.8 tb4.fr 25 msec 208.67.222.222 tb4.fr 17 msec 192.168.1.94 tb4.fr 0 msec

4.2.2.2 bbc.co.uk 31 msec 8.8.8.8 bbc.co.uk 27 msec 208.67.222.222 bbc.co.uk 18 msec 192.168.1.94 bbc.co.uk 0 msec

Looks like my local DNS resolver beats pretty much everyone.... Oh, and it also provides local top-level domain names just to my local network

$ ping testurl.virtual PING testurl.virtual (192.168.1.95) 56(84) bytes of data. 64 bytes from svenus.local (192.168.1.95): icmp\_seq=1 ttl=64 time=0.034 ms