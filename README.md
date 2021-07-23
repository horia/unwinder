# unwinder

Simple unwind drop-in replacement for unbound
> ideal for tiny routers

![unwinder logo](unwinder.jpg)

## About

[unbound(8)](https://man.openbsd.org/unbound.8) is a validating DNS resolver with complex configurations well suited for datacenter.

[unwind(8)](https://man.openbsd.org/unwind.8) is a simple validating DNS resolver wizard.

It's time to unwind.

## Why

* lower DNS latency
* free RAM
* keep existing DNS blocking lists and exceptions <sup>[[`1`](https://filterlists.com)] [[`2`](https://www.bravedns.com/configure)]</sup>

## How

[relayd(8)](https://man.openbsd.org/relayd.8) forwards DNS traffic between a client and unwind(8) using [unpredictable](https://man.openbsd.org/relayd.conf#dns) requested IDs in the DNS header.

unwind(8) queries its recursor, a DoT forwarder, or an authoritative nameserver.

## Features

* [lightweight](#preview)
* [fast](https://man.openbsd.org/unwind.conf#preference)
* [simple configuration](src/etc/unwind.conf)
* [sane defaults](https://www.openbsd.org/papers/bsdcan2019_unwind.pdf)
* [best practice recursor](https://undeadly.org/cgi?action=article;sid=20200922090542)
* [automatic cache](https://man.openbsd.org/unwind#DESCRIPTION)
* [efficient blocking list](https://man.openbsd.org/unwind.conf#block)
* residential [`home.arpa.`](https://tools.ietf.org/html/rfc8375) network support

## Getting started

Include and configure [pf.conf.unwinder](src/etc/pf.conf.unwinder)
```sh
pfctl -f /etc/pf.conf
```

Install and configure [myname](src/etc/myname)

Install and configure [nsd.conf](src/var/nsd/etc/nsd.conf)

Install and configure the [master zones](src/var/nsd/zones/master)
```sh
nsd-control-setup
rcctl restart nsd
```

Install and configure the [`unwind-block`](src/usr/local/bin/unwind-block) list fetcher and [daily.local](src/etc/daily.local) updates.
```sh
echo badexample.com > /var/db/unwind-block.txt.local
/usr/local/bin/unwind-block > /var/db/unwind-block.txt
```

Install and configure the [`unwind-unblock`](src/usr/local/bin/unwind-unblock) exceptions and [daily.local](src/etc/daily.local) updates.
```sh
echo example.com > /var/db/unwind-unblock.txt
/usr/local/bin/unwind-unblock > /var/db/unwind-block.txt.clean
mv /var/db/unwind-block.txt.clean /var/db/unwind-block.txt
```

Install and configure the [egress](src/etc/hostname.if) interface
```sh
cp src/etc/hostname.if /etc/hostname.em0
sh /etc/netstart em0
```

Install [resolv.conf](src/etc/resolv.conf)

Enable dhcpleased and resolvd on -release or -stable
```sh
rcctl enable dhcpleased resolvd
rcctl start dhcpleased resolvd
```

Install and configure [unwind.conf](src/etc/unwind.conf)
```sh
rcctl enable unwind
rcctl restart unwind
```

Install and configure [TLS certificates](https://github.com/horia/defaulter/) for DoT

*n.b.* To use DoT e.g. on a laptop, configure its unwind
```console
# $OpenBSD: unwind.conf

# Macros

v4unwinder="10.0.0.1 authentication name unwinder.example.com DoT"
v6unwinder="fd80:a:b:c::1 authentication name unwinder.example.com DoT"

# Global Configuration

forwarder {
 $v4unwinder
 $v6unwinder
}

preference DoT
```

Install and configure [relayd.conf](src/etc/relayd.conf)
```sh
rcctl restart relayd
```

Install and configure [dhcpd.conf](src/etc/dhcpd.conf)
```sh
rcctl restart dhcpd
```

Install and configure [rad.conf](src/etc/rad.conf)
```sh
rcctl restart rad
```

#### Preview

```console
$ du -h /var/db/unwind-block.txt
12.4M   /var/db/unwind-block.txt
```

```console
$ unwindctl status memory 
msg-cache:   76198 / 1048576 (7.27%)
rrset-cache: 228898 / 1048576 (21.83%)
key-cache: 34504 / 1048576 (3.29%)
neg-cache: 14212 / 102400 (13.88%)
```

```console
$ ps aux -U _unwind                                                                            
USER       PID %CPU %MEM   VSZ   RSS TT  STAT   STARTED       TIME COMMAND
_unwind  10938  0.0  0.4 16824 15980 ??  IpU    26Jun21   34:30.76 unwind: resolver (unwind)
_unwind  19822  0.0  1.6 65644 66052 ??  Ip     26Jun21   13:35.89 unwind: frontend (unwind)
```

```console
$ fstat -u _unwind -n
USER     CMD          PID   FD  DEV      INUM        MODE   R/W    SZ|DV
_unwind  unwind     19822   wd  4,57   207368        40755    r      512
_unwind  unwind     19822 root  4,57   207368        40755    r      512
_unwind  unwind     19822    0  4,48    17811        20666   rw    2,2  
_unwind  unwind     19822    1  4,48    17811        20666   rw    2,2  
_unwind  unwind     19822    2  4,48    17811        20666   rw    2,2  
_unwind  unwind     19822    3* unix stream 0x0
_unwind  unwind     19822    4 kqueue 0x0 0 state: W
_unwind  unwind     19822    5* unix stream 0x0
_unwind  unwind     19822    6* internet dgram udp 127.0.0.1:53
_unwind  unwind     19822    7* internet6 dgram udp [::1]:53
_unwind  unwind     19822    8* internet stream tcp 0x0 127.0.0.1:53
_unwind  unwind     19822    9* internet6 stream tcp 0x0 [::1]:53
_unwind  unwind     19822   10  4,57   129757       100644   rw      376
_unwind  unwind     19822   11* unix stream 0x0 /dev/unwind.sock
_unwind  unwind     19822   12* route raw 0 0x0
_unwind  unwind     19822   13* unix stream 0x0 /dev/unwind.sock
_unwind  unwind     10938   wd  4,48    49761        40700    r      512
_unwind  unwind     10938    0  4,48    17811        20666   rw    2,2  
_unwind  unwind     10938    1  4,48    17811        20666   rw    2,2  
_unwind  unwind     10938    2  4,48    17811        20666   rw    2,2  
_unwind  unwind     10938    3* unix stream 0x0
_unwind  unwind     10938    4 kqueue 0x0 0 state: W
_unwind  unwind     10938    5* unix stream 0x0
```

**Caveats**

Some public DNS resolvers (e.g. Google, Cloudflare) provide a response to DNS forward queries for home.arpa from IANA blackhole servers.

As a [feature](https://tools.ietf.org/html/rfc6305), unwind provides a negative response to DNS reverse-mapping queries for IP addresses that are not globally unique i.e. [AS112 zones](https://github.com/openbsd/src/blob/c552023e18e58a2d6580786dd404af8ac5f2cd5f/sbin/unwind/resolver.c#L221)

Split-horizon DNS is not supported. A [redirection and reflection](src/etc/pf.conf.unwinder) is used for connecting to the external address of the firewall from a host on the LAN.

