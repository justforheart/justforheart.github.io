---
title: Linux测试页面

category: 兴趣
date: 2023-01-09
---

#### 扫ip

```shell

~ nmap -sP 192.168.31.1/24

Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-15 23:02 CST

Nmap scan report for _gateway (192.168.31.1)

Host is up (0.00054s latency).

Nmap scan report for 192.168.31.5

Host is up (0.018s latency).

Nmap scan report for 192.168.31.11

Host is up (0.00028s latency).

Nmap scan report for distance (192.168.31.44)

Host is up (0.00034s latency).

Nmap scan report for 192.168.31.56

Host is up (0.0081s latency).

Nmap scan report for 192.168.31.226

Host is up (0.0064s latency).

Nmap done: 256 IP addresses (6 hosts up) scanned in 3.07 seconds

```

### 扫端口
```shell

hong:~/ $ nmap -A 192.168.31.11 [22:57:30]

Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-15 22:57 CST

Stats: 0:00:02 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan

Connect Scan Timing: About 2.60% done; ETC: 22:58 (0:01:15 remaining)

Stats: 0:01:02 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan

Connect Scan Timing: About 95.30% done; ETC: 22:58 (0:00:03 remaining)

Nmap scan report for 192.168.31.11

Host is up (0.78s latency).

Not shown: 928 filtered tcp ports (no-response), 69 filtered tcp ports (host-unreach)

PORT STATE SERVICE VERSION

22/tcp open ssh OpenSSH 8.6 (protocol 2.0)

| ssh-hostkey:

| 256 5b2c3fdc8b76e9217bd05624dfbee9a8 (ECDSA)

|_ 256 b03c723b722126ce3a84e841ecc8f841 (ED25519)

80/tcp open http Apache httpd 2.4.51 ((Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9)

|_http-title: Bad Request (400)

|_http-server-header: Apache/2.4.51 (Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9

443/tcp open ssl/http Apache httpd 2.4.51 ((Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9)

| http-methods:

|_ Potentially risky methods: TRACE

|_http-title: Test Page for the HTTP Server on Fedora

|_ssl-date: TLS randomness does not represent time

| tls-alpn:

|_ http/1.1

|_http-server-header: Apache/2.4.51 (Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9

| ssl-cert: Subject: commonName=earth.local/stateOrProvinceName=Space

| Subject Alternative Name: DNS:earth.local, DNS:terratest.earth.local

| Not valid before: 2021-10-12T23:26:31

|_Not valid after: 2031-10-10T23:26:31

  

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .

Nmap done: 1 IP address (1 host up) scanned in 96.14 seconds

```

  

- 扫到的ip,直接访问，会爆400 bad request.如此错误可能为请求方问题。看攻略可知可通过改hosts骗过服务器

- 由nmap信息可知，dns可能是Subject Alternative Name: DNS:earth.local, DNS:terratest.earth.local。

### 扫目录

- 接着爆目录，如果用ip爆，同样是请求不到数据，所以还是要设置好hosts再逐个尝试，最终爆出主页，`https://earth.local/admin/`,`https://terratest.earth.local/robots.txt`
```shell
~ dirb https://earth.local
-----------------

DIRB v2.22

By The Dark Raver

-----------------
START_TIME: Sat Apr 15 23:08:40 2023

URL_BASE: https://earth.local/

WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

  

-----------------

  

GENERATED WORDS: 4612

  

---- Scanning URL: https://earth.local/ ----

+ https://earth.local/admin (CODE:301|SIZE:0)

+ https://earth.local/cgi-bin/ (CODE:403|SIZE:199)

```

```shell

~ dirb https://terratest.earth.local

  

-----------------

DIRB v2.22

By The Dark Raver

-----------------

  

START_TIME: Sat Apr 15 23:09:12 2023

URL_BASE: https://terratest.earth.local/

WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

  

-----------------

  

GENERATED WORDS: 4612

  

---- Scanning URL: https://terratest.earth.local/ ----

+ https://terratest.earth.local/cgi-bin/ (CODE:403|SIZE:199)

+ https://terratest.earth.local/index.html (CODE:200|SIZE:26)

+ https://terratest.earth.local/robots.txt (CODE:200|SIZE:521)

  

-----------------

END_TIME: Sat Apr 15 23:09:14 2023

DOWNLOADED: 4612 - FOUND: 3

```

### 对目录分析，寻找突破口
- `https://earth.local/admin/`暂时看不出有什么用，先看看`https://terratest.earth.local/robots.txt`。
```txt

User-Agent: *

Disallow: /*.asp

Disallow: /*.aspx

Disallow: /*.bat

Disallow: /*.c

Disallow: /*.cfm

Disallow: /*.cgi

Disallow: /*.com

Disallow: /*.dll

Disallow: /*.exe

Disallow: /*.htm

Disallow: /*.html

Disallow: /*.inc

Disallow: /*.jhtml

Disallow: /*.jsa

Disallow: /*.json

Disallow: /*.jsp

Disallow: /*.log

Disallow: /*.mdb

Disallow: /*.nsf

Disallow: /*.php

Disallow: /*.phtml

Disallow: /*.pl

Disallow: /*.reg

Disallow: /*.sh

Disallow: /*.shtml

Disallow: /*.sql

Disallow: /*.txt

Disallow: /*.xml

Disallow: /testingnotes.*

```

- `Disallow: /testingnotes.*`比较可疑（当然也是看攻略得知），那我们`猜想`他是txt文件
```txt

Testing secure messaging system notes:

*Using XOR encryption as the algorithm, should be safe as used in RSA.

*Earth has confirmed they have received our sent messages.

*testdata.txt was used to test encryption.

*terra used as username for admin portal.

Todo:

*How do we send our monthly keys to Earth securely? Or should we change keys weekly?

*Need to test different key lengths to protect against bruteforce. How long should the key be?

*Need to improve the interface of the messaging interface and the admin panel, it's currently very basic.

```

  

- 大意是说，这是个安全信息系统，使用的是异或运算对信息加密

- 有个testdata.txt 是用来做加密（密钥）

- terra是管理员账号

- 访问`https://terratest.earth.local/testdata.txt`拿到密钥文件

  

```txt

According to radiometric dating estimation and other evidence, Earth formed over 4.5 billion years ago. Within the first billion years of Earth's history, life appeared in the oceans and began to affect Earth's atmosphere and surface, leading to the proliferation of anaerobic and, later, aerobic organisms. Some geological evidence indicates that life may have arisen as early as 4.1 billion years ago.

```