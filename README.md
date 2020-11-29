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
* keep existing [DNS blocking lists](https://filterlists.com) and exceptions

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

Install [dhclient.conf](src/etc/dhclient.conf)
```sh
sh /etc/netstart em0
```

Install [resolv.conf](src/etc/resolv.conf)

Install and configure [unwind.conf](src/etc/unwind.conf)
```sh
rcctl restart unwind
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
# du -h /var/db/unwind-block.txt
12.4M   /var/db/unwind-block.txt
```

```console
# unwindctl status memory 
msg-cache:   76198 / 1048576 (7.27%)
rrset-cache: 228898 / 1048576 (21.83%)
key-cache: 34504 / 1048576 (3.29%)
neg-cache: 14212 / 102400 (13.88%)
```

```console
# top -d1 -g unwind
  PID USERNAME PRI NICE  SIZE   RES STATE     WAIT      TIME    CPU COMMAND
71357 _unwind    2    0   10M   11M sleep/1   kqread    0:49  0.00% unwind
37791 _unwind    2    0   78M   85M sleep/2   kqread    0:08  0.00% unwind
38692 root       2    0 2916K 1180K idle      kqread    0:00  0.00% unwind
```

**Caveats**

Some public DNS resolvers (e.g. Google, Cloudflare) provide a response to DNS forward queries for home.arpa from IANA blackhole servers.

unwind provides a negative response to DNS reverse-mapping queries for IP addresses that are not globally unique, as a [feature](https://tools.ietf.org/html/rfc6305)

Split-horizon DNS is not supported. A [redirection and reflection](src/etc/pf.conf.unwinder) is used for connecting to the external address of the firewall from a host on the LAN.

