+++
author = "Steve"
title = "Route Server"
date = "2024-11-28"
description = "Lab to undestand how route server work"
series = ["Networking"]
tags = [
    "Networking",
]
+++

How route server work?, lets see how this things work <!--more-->

***Don't forget your coffee before reading this posts :)***

This Posts for documented my lab about route server using [IXP Manager](https://www.ixpmanager.org/)

In production you can see [here](https://squad.iix.net.id/login)

So much tool like IXP Manager, to manage Internet Exchange Point, the other things is [Alice Looking Glass](https://github.com/alice-lg/alice-lg), 
but i use IXP Manager because this tool so many feature to manage Internet Exchange Point by UI. 

First we must undestand [BGP bird](https://bird.network.cz/), because in ixp manager using bgp bird version 2 and the other reason is because bird is can be customized. 

Here is my lab topologi, 

![Topologi](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/TOPO.PNG)

## Table of Contents
1. [What is Route Server?](#what-is-route-server)
2. [How Crucial Route Server on Data Center](#how-crucial-route-server-on-data-center)
3. [IXP Manager UI](#ixp-manager-ui)
4. [How IXP Manager And Route Server Work](#how-ixp-manager-and-route-server-work)
    - [Flow Route Server and IXP Manager](#flow-route-server-and-ixp-manager)
    - [Reconfigure Script](#reconfigure-script)
    - [Flow How Route Server Export and Import Prefix](#flow-route-server-import-and-export-prefix)
       - [Flow route server import prefix](#flow-route-server-import-prefix)
       - [Flow route server export prefix](#flow-route-server-export-prefix)
5. [Route Server Configuration](#route-server-configure)
    - [Route Server Filtering](#route-server-filtering)
    - [Route Server client configuration](#route-server-client-configuration)
6. [Route Server Filtering From UI](#route-server-filtering-from-ui)
7. [Forward Traffc](#forward-traffic)

---

## What is Route Server
 
Route servers facilitate the sharing of routing information between multiple autonomous systems (ASes) without each network having to directly establish multiple peer-to-peer BGP (Border Gateway Protocol) sessions. This is especially useful in an IXP environment, where multiple networks (ISPs, data centers, etc.) connect to a common exchange.

## How Crucial Route Server on Data Center

Let undestand how crucial route server is. 


- #### Simplifies Peering Relationships
- A route server allows multiple Autonomous Systems (ASes) to exchange routing information without requiring direct, bilateral peering agreements between each pair. Instead of every network operator needing to manually configure routes for every other network, the route server automates and centralizes the process.
- #### Reduces Complexity
- large-scale data centers or IXPs, where many networks peer with each other, a route server simplifies peering management and reduces the number of direct connections required, saving time and effort.
- #### Security
- Route servers often implement security measures, such as prefix filtering and route validation, to prevent malicious or incorrect routing information from being propagated. This helps in safeguarding the network from attacks like prefix hijacking or route leaks.
- #### Scalability
- As the number of connected networks in a data center or IXP increases, a route server provides scalability by automatically managing the routing information exchanges for all networks involved, without overwhelming the administrators with manual configurations.
- #### Efficient Bandwidth Utilization
-  By enabling peering between different networks in a data center, route servers can help reduce the reliance on third-party transit providers, leading to better control over bandwidth costs and traffic distribution.

## IXP Manager UI 

UI and feature for administrator to manage IXP manager 

![UI For Admin](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/UI-admin.png)

UI and feature for member 

![UI For Admin](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/UI-member.png)

In IXP Manager also have Looking glass for member to monitor all prefix, below LG for IPV4

![LG](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/LG-IPV4.PNG)

Below LG for IPV6

![LG](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/LG-IPV6.PNG)


## How IXP Manager And Route Server Work

#### Flow Route Server and IXP Manager
IXP manager will push configure to Route Server via API, 

Here is flow how IXP manager push configure

![Flow IXP Manager](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/RS-FLOW.png)

#### Reconfigure Script

Route Server need reconfigure script to get update from IXP manager [here](https://raw.githubusercontent.com/inex/IXP-Manager/refs/heads/master/tools/runtime/route-servers/api-reconfigure-example-birdv2.sh) is reconfigure config file for route server 

To reconfigure both IPV4 and IPV6 you can use [this](https://raw.githubusercontent.com/inex/IXP-Manager/refs/heads/master/tools/runtime/route-servers/api-reconfigure-all.sh) config file

In config file you need api key and handle to get update config from IXP Manager

Here is reconfigure both ipv4 and ipv6 in my route server

```bash

echo "Reconfiguring all bird instances:"

# These handles should match the definitions in config/routers.php, and
# should be changed as appropriate:

for handle in rs2-vl2000-ipv4 rs2-vl2000-ipv6; do
    echo -ne "HANDLE: ${handle}: "
    /usr/local/sbin/api-reconfigure-rs2-birdv2.sh -f -h $handle -q
    if [[ $? -eq 0 ]]; then
        echo -ne "OK    "
    else
        echo -ne "ERROR "
    fi
    echo
done


```

Handle You can get in routers menu

![Routers Menu](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Handle-Route-Server.png)


In route server using cron job to get update from IXP manager

Here example in my lab, route server will get update from IXP Manager every 1 hour

```bash
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#

# Get update from IXP manager every 1 hour
0 *     * * *   root   /usr/local/sbin/api-reconfigure-all.sh > /dev/null

```

#### Flow Route Server Import And Export Prefix

##### Flow route server import prefix

Here is flow how route server import prefix from member

![Route Server Import Prefix](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Import-Prefix-From-member-to-master-table.png)

Bird will create routing table name `t_000X_asXXXXX` each member, in this table all prefix (filtered and accept prefix) will be placed

And only accepted prefix will through to master table

Filtered prefix will get large community tag. 

Here example prefix filtered get community tag

![Example prefix filtered](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Prefix-Filtered-example.png)

Here is prefix get RPKI and IRRDB community tag

![Example prefix filtered](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/RPKI-and-IRRDB-tag.png)


##### Flow route server export prefix

Here is flow how route server export prefix to member

![Route Server Import Prefix](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Export-prefix-to-member-from-master-table.png)

All accepted prefix pass through to member routing table and then strip route server tag, that means route server path not include, below example

![Path Example from member](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Prefix-example-view-from-member.png)

So member got prefix from route server and look like that mamber direct peer to other member, interesting isn't it :)

lets view detail some prefix

example below is detail prefix 34.1.88.0/24 we got from route server

```bash
vyos# sh ip bgp 34.1.88.0/24
BGP routing table entry for 34.1.88.0/24, version 123
Paths: (2 available, best #2, table default)
  Not advertised to any peer
  139190
    123.108.8.10 from 123.108.8.200 (123.108.8.200)
      Origin IGP, metric 0, valid, external
      Community: 32934:7597
      Last update: Mon Nov 25 01:00:02 2024
  139190
    123.108.8.10 from 123.108.8.100 (123.108.8.100)
      Origin IGP, metric 0, valid, external, best (Router ID)
      Community: 32934:7597
      Last update: Mon Nov 25 01:01:11 2024
vyos#

```

So route server not forward traffic, traffic will through via switch from one member to other member

## Route Server Configure

In this section I will explain BGP bird configure for route server

### Route Server Filtering  
First I will explain how route server filtering prefix

In the route server all prefix is filtered by BGP Community (standar and large community)

Here is the example large community filtering on route server


```bash

## These will all be filtered and not piped to the master table:

define IXP_LC_FILTERED_PREFIX_LEN_TOO_LONG      = ( routeserverasn, 1101, 1  );
define IXP_LC_FILTERED_PREFIX_LEN_TOO_SHORT     = ( routeserverasn, 1101, 2  );
define IXP_LC_FILTERED_BOGON                    = ( routeserverasn, 1101, 3  );
define IXP_LC_FILTERED_BOGON_ASN                = ( routeserverasn, 1101, 4  );
define IXP_LC_FILTERED_AS_PATH_TOO_LONG         = ( routeserverasn, 1101, 5  );
define IXP_LC_FILTERED_AS_PATH_TOO_SHORT        = ( routeserverasn, 1101, 6  );
define IXP_LC_FILTERED_FIRST_AS_NOT_PEER_AS     = ( routeserverasn, 1101, 7  );
define IXP_LC_FILTERED_NEXT_HOP_NOT_PEER_IP     = ( routeserverasn, 1101, 8  );
define IXP_LC_FILTERED_IRRDB_PREFIX_FILTERED    = ( routeserverasn, 1101, 9  );
define IXP_LC_FILTERED_IRRDB_ORIGIN_AS_FILTERED = ( routeserverasn, 1101, 10 );
define IXP_LC_FILTERED_PREFIX_NOT_IN_ORIGIN_AS  = ( routeserverasn, 1101, 11 );

define IXP_LC_FILTERED_RPKI_UNKNOWN             = ( routeserverasn, 1101, 12 );
define IXP_LC_FILTERED_RPKI_INVALID             = ( routeserverasn, 1101, 13 );
define IXP_LC_FILTERED_TRANSIT_FREE_ASN         = ( routeserverasn, 1101, 14 );
define IXP_LC_FILTERED_TOO_MANY_COMMUNITIES     = ( routeserverasn, 1101, 15 );

```

```bash

# Informational prefixes

define IXP_LC_INFO_RPKI_VALID       = ( routeserverasn, 1000, 1  );
define IXP_LC_INFO_RPKI_UNKNOWN     = ( routeserverasn, 1000, 2  );
define IXP_LC_INFO_RPKI_NOT_CHECKED = ( routeserverasn, 1000, 3  );

define IXP_LC_INFO_IRRDB_VALID         = ( routeserverasn, 1001, 1  );
define IXP_LC_INFO_IRRDB_NOT_CHECKED   = ( routeserverasn, 1001, 2  );
define IXP_LC_INFO_IRRDB_MORE_SPECIFIC = ( routeserverasn, 1001, 3  );

define IXP_LC_INFO_IRRDB_FILTERED_LOOSE  = ( routeserverasn, 1001, 1000 );
define IXP_LC_INFO_IRRDB_FILTERED_STRICT = ( routeserverasn, 1001, 1001 );
define IXP_LC_INFO_IRRDB_PREFIX_EMPTY    = ( routeserverasn, 1001, 1002 );

define IXP_LC_INFO_SAME_AS_NEXT_HOP = ( routeserverasn, 1001, 1200 );

# ( routeserverasn, 1010, peerasn ) -> route learnt from peerasn via routeserverasn
# ( routeserverasn, 1011, originasn ) -> route origin asn via routeserverasn


# And the filter for examining routes in the peers import table being exported
# to the master table

filter f_export_to_master
{

    if bgp_large_community ~ [( routeserverasn, 1101, * )] then reject;

    accept;
}

```

Here is Standard IXP community filter

```bash

function ixp_community_filter(int peerasn)
{
    if !(source = RTS_BGP) then
            return false;

    # AS path prepending
    if (routeserverasn, 103, peerasn) ~ bgp_large_community then {
        bgp_path.prepend( bgp_path.first );
        bgp_path.prepend( bgp_path.first );
        bgp_path.prepend( bgp_path.first );
    } else if (routeserverasn, 102, peerasn) ~ bgp_large_community then {
        bgp_path.prepend( bgp_path.first );
        bgp_path.prepend( bgp_path.first );
    } else if (routeserverasn, 101, peerasn) ~ bgp_large_community then {
        bgp_path.prepend( bgp_path.first );
    } else if (routeserverasn, 103, 0) ~ bgp_large_community then {
        bgp_path.prepend( bgp_path.first );
        bgp_path.prepend( bgp_path.first );
        bgp_path.prepend( bgp_path.first );
    } else if (routeserverasn, 102, 0) ~ bgp_large_community then {
        bgp_path.prepend( bgp_path.first );
        bgp_path.prepend( bgp_path.first );
    } else if (routeserverasn, 101, 0) ~ bgp_large_community then {
        bgp_path.prepend( bgp_path.first );
    }


    # support for BGP Large Communities
    if (routeserverasn, 0, peerasn) ~ bgp_large_community then
            return false;
    if (routeserverasn, 1, peerasn) ~ bgp_large_community then
            return true;
    if (routeserverasn, 0, 0) ~ bgp_large_community then
            return false;
    if (routeserverasn, 1, 0) ~ bgp_large_community then
            return true;

    # it's unwise to conduct a 32-bit check on a 16-bit value
    if routeserverasn > 65535 || peerasn > 65535 then
            return true;

    # Implement widely used community filtering schema.
    if (0, peerasn) ~ bgp_community then
            return false;
    if (routeserverasn, peerasn) ~ bgp_community then
            return true;
    if (0, routeserverasn) ~ bgp_community then
            return false;

    return true;
}

```

Function below is for filter all bogon ipv4 prefix

```bash
# This function excludes weird networks
#  rfc1918, class D, class E, too long and too short prefixes
function avoid_martians()
prefix set martians;
{

        martians = [
                0.0.0.0/32-,            # rfc5735 Special Use IPv4 Addresses
                0.0.0.0/0{0,7},         # rfc1122 Requirements for Internet Hosts -- Communication Layers 3.2.1.3
                10.0.0.0/8+,            # rfc1918 Address Allocation for Private Internets
                100.64.0.0/10+,         # rfc6598 IANA-Reserved IPv4 Prefix for Shared Address Space
                127.0.0.0/8+,           # rfc1122 Requirements for Internet Hosts -- Communication Layers 3.2.1.3
                169.254.0.0/16+,        # rfc3927 Dynamic Configuration of IPv4 Link-Local Addresses
                172.16.0.0/12+,         # rfc1918 Address Allocation for Private Internets
                192.0.0.0/24+,          # rfc6890 Special-Purpose Address Registries
                192.0.2.0/24+,          # rfc5737 IPv4 Address Blocks Reserved for Documentation
                192.168.0.0/16+,        # rfc1918 Address Allocation for Private Internets
                198.18.0.0/15+,         # rfc2544 Benchmarking Methodology for Network Interconnect Devices
                198.51.100.0/24+,       # rfc5737 IPv4 Address Blocks Reserved for Documentation
                203.0.113.0/24+,        # rfc5737 IPv4 Address Blocks Reserved for Documentation
                224.0.0.0/4+,           # rfc1112 Host Extensions for IP Multicasting
                240.0.0.0/4+            # rfc6890 Special-Purpose Address Registries
        ];


        # Avoid RFC1918 and similar networks
        if net ~ martians then
                return false;

        return true;
}

```
Function below is for filter all bogon ipv6 prefix


```bash
# This function excludes weird networks
#  rfc1918, class D, class E, too long and too short prefixes
function avoid_martians()
prefix set martians;
{

        martians = [
                ::/0,                   # Default (can be advertised as a route in BGP to peers if desired)
                ::/96,                  # IPv4-compatible IPv6 address - deprecated by RFC4291
                ::/128,                 # Unspecified address
                ::1/128,                # Local host loopback address
                ::ffff:0.0.0.0/96+,     # IPv4-mapped addresses
                ::224.0.0.0/100+,       # Compatible address (IPv4 format)
                ::127.0.0.0/104+,       # Compatible address (IPv4 format)
                ::0.0.0.0/104+,         # Compatible address (IPv4 format)
                ::255.0.0.0/104+,       # Compatible address (IPv4 format)
                0000::/8+,              # Pool used for unspecified, loopback and embedded IPv4 addresses
                0200::/7+,              # OSI NSAP-mapped prefix set (RFC4548) - deprecated by RFC4048
                3ffe::/16+,             # Former 6bone, now decommissioned
                2001:db8::/32+,         # Reserved by IANA for special purposes and documentation
                2002:e000::/20+,        # Invalid 6to4 packets (IPv4 multicast)
                2002:7f00::/24+,        # Invalid 6to4 packets (IPv4 loopback)
                2002:0000::/24+,        # Invalid 6to4 packets (IPv4 default)
                2002:ff00::/24+,        # Invalid 6to4 packets
                2002:0a00::/24+,        # Invalid 6to4 packets (IPv4 private 10.0.0.0/8 network)
                2002:ac10::/28+,        # Invalid 6to4 packets (IPv4 private 172.16.0.0/12 network)
                2002:c0a8::/32+,        # Invalid 6to4 packets (IPv4 private 192.168.0.0/16 network)
                fc00::/7+,              # Unicast Unique Local Addresses (ULA) - RFC 4193
                fe80::/10+,             # Link-local Unicast
                fec0::/10+,             # Site-local Unicast - deprecated by RFC 3879 (replaced by ULA)
                ff00::/8+               # Multicast
        ];


        # Avoid RFC1918 and similar networks
        if net ~ martians then
                return false;

        return true;
}

```


If in the route server enable RPKI filtering, below function to filtering with RPKI, 

In this lab i enable RPKI filtering using [Routinator3000](https://routinator.docs.nlnetlabs.nl/en/stable/) for validation

```bash

roa4 table t_roa;

protocol rpki rpki1 {

    roa4 { table t_roa; };

    remote "REMOTE-RPKI-SERVER" port 3323;

    retry keep 90;
    refresh keep 900;
    expire keep 172800;
}


/*
 * RPKI check for the path
 *
 * return: true means the filter should stop processing, false means keep processing
 */
function filter_rpki()
{
    # RPKI check
    if( roa_check( t_roa, net, bgp_path.last_nonaggregated ) = ROA_INVALID ) then {
        print "Tagging invalid ROA ", net, " for ASN ", bgp_path.last;
        bgp_large_community.add( IXP_LC_FILTERED_RPKI_INVALID );
        return true;
    }

    if( roa_check( t_roa, net, bgp_path.last_nonaggregated ) = ROA_VALID ) then {
        bgp_large_community.add( IXP_LC_INFO_RPKI_VALID );
        return true;
    }

    # RPKI unknown, keep checking and mark as unknown for info
    bgp_large_community.add( IXP_LC_INFO_RPKI_UNKNOWN );

    return false;
}

```
Here is to filtering Transit ASN


```bash
# Filtering the following ASNs:
#
# 174 - Cogent
# 701 - UUNET
# 1299 - Telia
# 2914 - NTT Communications
# 3257 - GTT Backbone
# 3320 - Deutsche Telekom AG (DTAG)
# 3356 - Level3
# 3491 - PCCW
# 4134 - Chinanet
# 5511 - Orange opentransit
# 6453 - Tata Communications
# 6461 - Zayo Bandwidth
# 6762 - Seabone / Telecom Italia
# 6830 - Liberty Global
# 7018 - AT&T

define TRANSIT_ASNS = [ 174, 701, 1299, 2914, 3257, 3320, 3356, 3491, 4134, 5511, 6453, 6461, 6762, 6830, 7018 ];

function filter_has_transit_path()
int set transit_asns;
{
    transit_asns = TRANSIT_ASNS;
    if (bgp_path ~ transit_asns) then {
        bgp_large_community.add( IXP_LC_FILTERED_TRANSIT_FREE_ASN );
        return true;
    }

    return false;
}

```


#### Route Server client configuration

Config below is for export all prefix IPV4 that have been filtered to all member 

```bash

template bgp tb_rsclient {
    local as routeserverasn;
    source address routeserveraddress;
    strict bind yes;

    # give RPKI-RTR a chance to start and populate
    # (RPKI is /really/ quick)
    connect delay time 30;

    interpret communities off;  # enable rfc1997 well-known community pass through

    ipv4 {
        export all;
    };

    rs client;
}

```

Config below is for export all prefix IPV6 that have been filtered to all member 


```bash

template bgp tb_rsclient {
    local as routeserverasn;
    source address routeserveraddress;
    strict bind yes;

    # give RPKI-RTR a chance to start and populate
    # (RPKI is /really/ quick)
    connect delay time 30;

    interpret communities off;  # enable rfc1997 well-known community pass through

    ipv6 {
        export all;
    };

    rs client;
}

```

#### Route Server Peer Configure

Here is one example route server peer configure to member

Example below only for IPV4 Peer

```bash
########################################################################################
########################################################################################
###
### AS139190 - Google Cloud Indonesia - VLAN Interface #3

ipv4 table t_0003_as139190;



filter f_import_as139190
prefix set allnet;
ip set allips;
int set allas;
{


    # Filter small prefixes
    if ( net ~ [ 0.0.0.0/0{25,32} ] ) then {
        bgp_large_community.add( IXP_LC_FILTERED_PREFIX_LEN_TOO_LONG );
        accept;
    }


    if !(avoid_martians()) then {
        bgp_large_community.add( IXP_LC_FILTERED_BOGON );
        accept;
    }

    # Belt and braces: must have at least one ASN in the path
    if( bgp_path.len < 1 ) then {
        bgp_large_community.add( IXP_LC_FILTERED_AS_PATH_TOO_SHORT );
        accept;
    }

    # Peer ASN == route's first ASN?
    if (bgp_path.first != 139190 ) then {
        bgp_large_community.add( IXP_LC_FILTERED_FIRST_AS_NOT_PEER_AS );
        accept;
    }

    # set of all IPs this ASN uses to peer with on this VLAN
    allips = [ 123.108.8.10 ];

    # Prevent BGP NEXT_HOP Hijacking
    if !( from = bgp_next_hop ) then {

        # need to differentiate between same ASN next hop or actual next hop hijacking
        if( bgp_next_hop ~ allips ) then {
            bgp_large_community.add( IXP_LC_INFO_SAME_AS_NEXT_HOP );
        } else {
            # looks like hijacking (intentional or not)
            bgp_large_community.add( IXP_LC_FILTERED_NEXT_HOP_NOT_PEER_IP );
            accept;
        }
    }


    # Filter Known Transit Networks
    if filter_has_transit_path() then accept;

    # Belt and braces: no one needs an ASN path with > 64 hops, that's just broken
    if( bgp_path.len > 64 ) then {
        bgp_large_community.add( IXP_LC_FILTERED_AS_PATH_TOO_LONG );
        accept;
    }


  allas = [ 139190
    ];


    # Ensure origin ASN is in the neighbors AS-SET
    if !(bgp_path.last_nonaggregated ~ allas) then {
        bgp_large_community.add( IXP_LC_FILTERED_IRRDB_ORIGIN_AS_FILTERED );
        accept;
    }



    # RPKI test - if it's INVALID or VALID, we are done
    if filter_rpki() then accept;




    allnet = [ 34.1.88.0/24,             34.101.0.0/16,            35.219.0.0/17
    ];

    
    if ! (net ~ allnet) then {
        bgp_large_community.add( IXP_LC_FILTERED_IRRDB_PREFIX_FILTERED );
        bgp_large_community.add( IXP_LC_INFO_IRRDB_FILTERED_STRICT );
        accept;
    } else {
        bgp_large_community.add( IXP_LC_INFO_IRRDB_VALID );
    }



    accept;
}


# The route server export filter exists as the export gateway on the BGP protocol.
#
# Remember that standard IXP community filtering has already happened on the
# master -> bgp protocol pipe.

filter f_export_as139190{

    # we should strip our own communities which we used for the looking glass and filtering
    bgp_large_community.delete( [( routeserverasn, *, * )] );
    bgp_community.delete( [( routeserverasn, * )] );



    # default position is to accept:
    accept;

}


    
protocol pipe pp_0003_as139190 {
        description "Pipe for AS139190 - Google Cloud Indonesia - VLAN Interface 3";
        table master4;
        peer table t_0003_as139190;
        import filter f_export_to_master;
        export where ixp_community_filter(139190);
}

protocol bgp pb_0003_as139190 from tb_rsclient {
        description "AS139190 - Google Cloud Indonesia";
        neighbor 123.108.8.10 as 139190;
        ipv4 {
            import limit 6000 action restart;
            import filter f_import_as139190;
            table t_0003_as139190;
            export filter f_export_as139190;
        };
        
}


```

Below for IPV6 peer

```bash
########################################################################################
########################################################################################
###
### AS139190 - Google Cloud Indonesia - VLAN Interface #3

ipv6 table t_0003_as139190;



filter f_import_as139190
prefix set allnet;
ip set allips;
int set allas;
{


    # Filter small prefixes
    if ( net ~ [ ::/0{49,128} ] ) then {
        bgp_large_community.add( IXP_LC_FILTERED_PREFIX_LEN_TOO_LONG );
        accept;
    }


    if !(avoid_martians()) then {
        bgp_large_community.add( IXP_LC_FILTERED_BOGON );
        accept;
    }

    # Belt and braces: must have at least one ASN in the path
    if( bgp_path.len < 1 ) then {
        bgp_large_community.add( IXP_LC_FILTERED_AS_PATH_TOO_SHORT );
        accept;
    }

    # Peer ASN == route's first ASN?
    if (bgp_path.first != 139190 ) then {
        bgp_large_community.add( IXP_LC_FILTERED_FIRST_AS_NOT_PEER_AS );
        accept;
    }

    # set of all IPs this ASN uses to peer with on this VLAN
    allips = [ 2001:7fa:2:5::10 ];

    # Prevent BGP NEXT_HOP Hijacking
    if !( from = bgp_next_hop ) then {

        # need to differentiate between same ASN next hop or actual next hop hijacking
        if( bgp_next_hop ~ allips ) then {
            bgp_large_community.add( IXP_LC_INFO_SAME_AS_NEXT_HOP );
        } else {
            # looks like hijacking (intentional or not)
            bgp_large_community.add( IXP_LC_FILTERED_NEXT_HOP_NOT_PEER_IP );
            accept;
        }
    }


    # Filter Known Transit Networks
    if filter_has_transit_path() then accept;

    # Belt and braces: no one needs an ASN path with > 64 hops, that's just broken
    if( bgp_path.len > 64 ) then {
        bgp_large_community.add( IXP_LC_FILTERED_AS_PATH_TOO_LONG );
        accept;
    }


        
    allas = [ 139190
    ];


    # Ensure origin ASN is in the neighbors AS-SET
    if !(bgp_path.last_nonaggregated ~ allas) then {
        bgp_large_community.add( IXP_LC_FILTERED_IRRDB_ORIGIN_AS_FILTERED );
        accept;
    }



    # RPKI test - if it's INVALID or VALID, we are done
    if filter_rpki() then accept;




    allnet = [ 2404:f340:4000::/34
    ];

    
    if ! (net ~ allnet) then {
        bgp_large_community.add( IXP_LC_FILTERED_IRRDB_PREFIX_FILTERED );
        bgp_large_community.add( IXP_LC_INFO_IRRDB_FILTERED_STRICT );
        accept;
    } else {
        bgp_large_community.add( IXP_LC_INFO_IRRDB_VALID );
    }



    accept;
}


# The route server export filter exists as the export gateway on the BGP protocol.
#
# Remember that standard IXP community filtering has already happened on the
# master -> bgp protocol pipe.

filter f_export_as139190{

    # we should strip our own communities which we used for the looking glass and filtering
    bgp_large_community.delete( [( routeserverasn, *, * )] );
    bgp_community.delete( [( routeserverasn, * )] );



    # default position is to accept:
    accept;

}


    
protocol pipe pp_0003_as139190 {
        description "Pipe for AS139190 - Google Cloud Indonesia - VLAN Interface 3";
        table master6;
        peer table t_0003_as139190;
        import filter f_export_to_master;
        export where ixp_community_filter(139190);
}

protocol bgp pb_0003_as139190 from tb_rsclient {
        description "AS139190 - Google Cloud Indonesia";
        neighbor 2001:7fa:2:5::10 as 139190;
        ipv6 {
            import limit 6000 action restart;
            import filter f_import_as139190;
            table t_0003_as139190;
            export filter f_export_as139190;
        };
        
}

```


Example above that member i enable IRRDB Filtering so only AS Number define on `allas` and all net define on `allnet` only will accepted

In this lab I using [bgpq3](https://github.com/snar/bgpq3) for filtering IRRDB, so this lab using RPKI and IRRDB for filtering prefix from member,

a little stricter, but it's good Right :)


## Route Server Filtering From UI 

In the IXP Manager version 6.3.x member can't filtering or prepand prefix from UI, so member manual tag below community
to filtering or prepend prefix to other member

![Filtering Base Community](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Filtering-Base-Community.png)

but in version 6.4.x member can filtering or prepand prefix from UI

In this Lab I using IXP Manager version 6.4.1

Below example member(AS23679) filtering prefix from UI 

![Filtering From UI](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Filtering-From-UI.png)

Rule above is for prepend 3 time to member name `META` and all prefix from member name `META` to member using that rule will be prepend 3 time

Below is member name META got prefix from AS23679 that have been prepend 3 time

![Filtering From UI](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Filtering-From-UI-2.png)

Below is Member with `AS23679` got prefix from `META` that have been prepend 3 time

![Filtering From UI](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Filtering-From-UI-3.png)


## NOT Forward Traffic

Route Server also not forward traffic but only export and import prefix from member to other member and I already explain in [Flow Route Server Import And Export Prefix](#flow-route-server-import-and-export-prefix)

Lets prove it

Here I prove it as a member `AS23679`

Below in router Bromo, I only have PEER to Route Server

![Prove](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Prove-route-server.png)

In router bromo I get prefix member name `Cloudflare` via route server

![Prove-1](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Prove-route-server-1.png)

Below is routing table on Router Bromo

![Prove-4](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Prove-route-server-4.png)

Let see detail prefix we got from route server, below detail prefix 1.1.1.0/24 we got

![Prove-3](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Prove-route-server-3.png)

In the router distribusi below I try traceroute to 1.1.1.1 

![Prove-2](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Prove-route-server-2.png)

Lets try on cloudflare router

![Prove-5](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/Prove-route-server-5.png)

See it's like direct PEER but it's via route server :)


For more detail configure route server or route collector you can see youtube [IXP Manager](https://www.youtube.com/@ixpmanager4386)

Or you can see the documentation [here](https://docs.ixpmanager.org/latest/)

Hope You enjoy read this Posts :)