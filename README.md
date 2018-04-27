# Run named on Docker

[![Docker Pulls](https://img.shields.io/docker/pulls/talmai/docker-bind9.svg)](https://hub.docker.com/r/talmai/docker-bind9/)
[![Docker Stars](https://img.shields.io/docker/stars/talmai/docker-bind9.svg)](https://hub.docker.com/r/talmai/docker-bind9/)

Lightweight docker container for bind9. This is mostly for my own usage.

### Command to use with this image:

```
docker run -d --name bind9 -p 53:53 -p 53:53/udp -v /absolute/path/named.conf:/etc/bind/named.conf -v /absolute/path/named.conf.local:/etc/bind/named.conf.local -v /absolute/path/talm.ai.zone:/etc/bind/pri/talm.ai.zone talmai/docker-bind9
```

### Authoritative nameserver

This is a small basic file named.conf if you want to run bind as an authoritative nameserver:

```
// This is the primary configuration file for the BIND DNS server named.
// Please read /usr/share/doc/bind9/README.Debian.gz for information on the
// structure of BIND configuration files in Debian,

/*
 * You might put in here some ips which are allowed to use the cache or
 * recursive queries
 */
acl "trusted" {
        192.249.63.29;
        192.249.61.160;
        127.0.0.0/8;
        ::1/128;
};

acl "xfer" {
        /* Deny transfers by default except for the listed hosts.
         * If we have other name servers, place them here.
         */
        202.157.182.142; 212.227.123.29; 87.98.164.164; 195.234.42.1; 88.191.64.64; 92.243.14.172;
};

// Load options
include "/etc/bind/named.conf.options";

include "/etc/bind/rndc.key";
controls {
        inet 127.0.0.1 port 953 allow { 127.0.0.1/32; ::1/128; };
};

// setup logging
logging {
        channel default_log {
                file "/var/log/named/named.log" versions 5 size 50M;
                print-time yes;
                print-severity yes;
                print-category yes;
                severity debug; //warning;
        };

        category config  { default_log; };
        category default { default_log; };
        category general { default_log; };
        category notify  { default_log; };
        category dnssec  { default_log; };
        category security { default_log; };
        category xfer-out { default_log; };
        category lame-servers { default_log; };
        category queries { default_log; };
        category update { default_log; };
};

// load zones...
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```

And here is the named.conf.local file:

```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

/*
 * limited to containing NS RRs for subdomains, but no actual data beyond its
 * own apex (for example, its SOA RR and apex NS RRset). This can be used to
 * filter out "wildcard" or "synthesized" data from NAT boxes or from
 * authoritative name servers whose undelegated (in-zone) data is of no
 * interest.
 * See http://www.isc.org/software/bind/delegation-only for more info
 */

//zone "COM" { type delegation-only; };
//zone "NET" { type delegation-only; };

//zone "YOUR-DOMAIN.TLD" {
//	type master;
//	file "/var/bind/pri/YOUR-DOMAIN.TLD.zone";
//	allow-query { any; };
//	allow-transfer { xfer; };
//};

//zone "YOUR-SLAVE.TLD" {
//	type slave;
//	file "/var/bind/sec/YOUR-SLAVE.TLD.zone";
//	masters { <MASTER>; };

	/* Anybody is allowed to query but transfer should be controlled by the master. */
//	allow-query { any; };
//	allow-transfer { none; };

	/* The master should be the only one who notifies the slaves, shouldn't it? */
//	allow-notify { <MASTER>; };
//	notify no;
//};

zone "talm.ai" {
        type master;
        file "/etc/bind/pri/talm.ai.zone";
        notify no;
        allow-query {any; };
        allow-transfer {xfer; };
};

```
