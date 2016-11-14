# Announcing, receiving and displaying large communities

## Scenario

- ExaBGP, 192.0.2.2, 65536
- GoBGP, 192.0.2.3, 65537
- BIRD, 192.0.2.4, 65538
- pmacct, 192.0.2.5
- Quagga, 192.0.2.6, 65539

They (pmacct excluded) announce the following prefixes:

- 203.0.113.x1, 1 large community (ASN:1:1)
- 203.0.113.x2, 2 large communities (ASN:1:1 ASN:1:2)
- 203.0.113.x3, 3 large communities (ASN:1:1 ASN:1:2 ASN:1:3)
- 203.0.113.x5, large communities with zeroes (ASN:0:1 ASN:1:0)
- 203.0.113.x6, large communities with reserved values (65535:1:1 4294967295:4294967295:4294967295 0:0:0)

x = 1 for ExaBGP, 2 for GoBGP, 3 for BIRD.

ExaBGP and BIRD read the prefixes to announce from their [configuration files](01/); GoBGP announces them via CLI:

```
gobgp global rib add -a ipv4 203.0.113.21/32 large-community 65537:1:1
gobgp global rib add -a ipv4 203.0.113.22/32 large-community 65537:1:1,65537:1:2
gobgp global rib add -a ipv4 203.0.113.23/32 large-community 65537:1:1,65537:1:2,65537:1:3
gobgp global rib add -a ipv4 203.0.113.25/32 large-community 65537:0:1,65537:1:0
gobgp global rib add -a ipv4 203.0.113.26/32 large-community 65535:1:1,4294967295:4294967295:4294967295,0:0:0
```

## Results

### ExaBGP announcing

GoBGP:

```
root@gobgp:/go# gobgp neighbor 192.0.2.2 adj-in
    Network             Next Hop             AS_PATH              Age        Attrs
    203.0.113.11/32     192.0.2.2            65536                00:03:23   [{Origin: i} {LargeCommunity: [ 65536:1:1]}]
    203.0.113.12/32     192.0.2.2            65536                00:03:23   [{Origin: i} {LargeCommunity: [ 65536:1:1, 65536:1:2]}]
    203.0.113.13/32     192.0.2.2            65536                00:03:23   [{Origin: i} {LargeCommunity: [ 65536:1:1, 65536:1:2, 65536:1:3]}]
    203.0.113.15/32     192.0.2.2            65536                00:03:23   [{Origin: i} {LargeCommunity: [ 65536:0:1, 65536:1:0]}]
    203.0.113.16/32     192.0.2.2            65536                00:03:23   [{Origin: i} {LargeCommunity: [ 65535:1:1, 4294967295:4294967295:4294967295, 0:0:0]}]
```

:white_check_mark: ExaBGP, send correctly formatted large communities

:white_check_mark: ExaBGP, send 1, 2 or 3 large communities (203.0.113.11/32, 203.0.113.12/32, 203.0.113.13/32)

:white_check_mark: ExaBGP, send reserved values (203.0.113.16/32)

:white_check_mark: GoBGP, receive and display large communities with routes

:white_check_mark: GoBGP, receive 1, 2 or 3 large communities (203.0.113.11/32, 203.0.113.12/32, 203.0.113.13/32)

:white_check_mark: GoBGP, textual representation, integers not omitted, even when zero (203.0.113.15/32)

:white_check_mark: GoBGP, display reserved values (203.0.113.16/32)

BIRD:

```
bird> show route all protocol ExaBGP
203.0.113.15/32    unreachable [ExaBGP 18:44:33 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (65536, 0, 1) (65536, 1, 0)
203.0.113.12/32    unreachable [ExaBGP 18:44:33 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (65536, 1, 1) (65536, 1, 2)
203.0.113.13/32    unreachable [ExaBGP 18:44:33 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (65536, 1, 1) (65536, 1, 2) (65536, 1, 3)
203.0.113.11/32    unreachable [ExaBGP 18:44:33 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (65536, 1, 1)
203.0.113.16/32    unreachable [ExaBGP 18:44:33 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (65535, 1, 1) (4294967295, 4294967295, 4294967295) (0, 0, 0)
```

:white_check_mark: BIRD, receive and display large communities with routes

:white_check_mark: BIRD, receive 1, 2 or 3 large communities (203.0.113.11/32, 203.0.113.12/32, 203.0.113.13/32)

:white_check_mark: BIRD, textual representation, integers not omitted, even when zero (203.0.113.15/32)

:white_check_mark: BIRD, display reserved values (203.0.113.16/32)

pmacct (bgp_table_dump_file):

```
# cat pmacct/bgp.log
{..., "peer_ip_src": "192.0.2.2", "event_type": "dump_init", "dump_period": 60}
{..., "peer_ip_src": "192.0.2.2", "event_type": "dump", "ip_prefix": "203.0.113.11/32", ..., "lcomms": "65536:1:1", ...}
{..., "peer_ip_src": "192.0.2.2", "event_type": "dump", "ip_prefix": "203.0.113.12/32", ..., "lcomms": "65536:1:1 65536:1:2", ...}
{..., "peer_ip_src": "192.0.2.2", "event_type": "dump", "ip_prefix": "203.0.113.13/32", ..., "lcomms": "65536:1:1 65536:1:2 65536:1:3", ...}
{..., "peer_ip_src": "192.0.2.2", "event_type": "dump", "ip_prefix": "203.0.113.15/32", ..., "lcomms": "65536:0:1 65536:1:0", ...}
{..., "peer_ip_src": "192.0.2.2", "event_type": "dump", "ip_prefix": "203.0.113.16/32", ..., "lcomms": "0:0:0 65535:1:1 4294967295:4294967295:4294967295", ...}
{..., "peer_ip_src": "192.0.2.2", "event_type": "dump_close", "entries": 5, "tables": 1}
```

:white_check_mark: pmacct, receive and display large communities with routes

:white_check_mark: pmacct, receive 1, 2 or 3 large communities (203.0.113.11/32, 203.0.113.12/32, 203.0.113.13/32)

:white_check_mark: pmacct, textual representation, integers not omitted, even when zero (203.0.113.15/32)

:white_check_mark: pmacct, display reserved values (203.0.113.16/32)

Quagga:

```
QuaggaBGPD# show bgp ipv4 unicast 203.0.113.11/32 bestpath
BGP routing table entry for 203.0.113.11/32
Paths: (3 available, best #2, table Default-IP-Routing-Table)
  Not advertised to any peer
  65536
    192.0.2.2 from 192.0.2.2 (192.0.2.2)
      Origin IGP, localpref 100, valid, external, best
      Large Community: 65536:1:1
      Last update: Mon Oct 31 15:18:50 2016

QuaggaBGPD# show bgp ipv4 unicast 203.0.113.12/32 bestpath
BGP routing table entry for 203.0.113.12/32
Paths: (3 available, best #2, table Default-IP-Routing-Table)
  Not advertised to any peer
  65536
    192.0.2.2 from 192.0.2.2 (192.0.2.2)
      Origin IGP, localpref 100, valid, external, best
      Large Community: 65536:1:1 65536:1:2
      Last update: Mon Oct 31 15:18:50 2016

QuaggaBGPD# show bgp ipv4 unicast 203.0.113.13/32 bestpath
BGP routing table entry for 203.0.113.13/32
Paths: (3 available, best #2, table Default-IP-Routing-Table)
  Not advertised to any peer
  65536
    192.0.2.2 from 192.0.2.2 (192.0.2.2)
      Origin IGP, localpref 100, valid, external, best
      Large Community: 65536:1:1 65536:1:2 65536:1:3
      Last update: Mon Oct 31 15:18:50 2016

QuaggaBGPD# show bgp ipv4 unicast 203.0.113.15/32 bestpath
BGP routing table entry for 203.0.113.15/32
Paths: (3 available, best #2, table Default-IP-Routing-Table)
  Not advertised to any peer
  65536
    192.0.2.2 from 192.0.2.2 (192.0.2.2)
      Origin IGP, localpref 100, valid, external, best
      Large Community: 65536:0:1 65536:1:0
      Last update: Mon Oct 31 15:18:50 2016

QuaggaBGPD# show bgp ipv4 unicast 203.0.113.16/32 bestpath
BGP routing table entry for 203.0.113.16/32
Paths: (3 available, best #1, table Default-IP-Routing-Table)
  Not advertised to any peer
  65536
    192.0.2.2 from 192.0.2.2 (192.0.2.2)
      Origin IGP, localpref 100, valid, external, best
      Large Community: 0:0:0 65535:1:1 4294967295:4294967295:4294967295
      Last update: Mon Nov 14 17:26:59 2016
```

:white_check_mark: Quagga, receive and display large communities with routes

:white_check_mark: Quagga, receive 1, 2 or 3 large communities (203.0.113.11/32, 203.0.113.12/32, 203.0.113.13/32)

:white_check_mark: Quagga, textual representation, integers not omitted, even when zero (203.0.113.15/32)

:white_check_mark: Quagga, display reserved values (203.0.113.16/32)

### GoBGP announcing

ExaBGP JSON dump:

```
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.3" }, "asn": { "local": "65536", "peer": "65537" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65537 ], "confederation-path": [], "large-community": [ [ 65537, 1 , 1 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.3": [ { "nlri": "203.0.113.21/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.3" }, "asn": { "local": "65536", "peer": "65537" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65537 ], "confederation-path": [], "large-community": [ [ 65537, 1 , 1 ], [ 65537, 1 , 2 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.3": [ { "nlri": "203.0.113.22/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.3" }, "asn": { "local": "65536", "peer": "65537" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65537 ], "confederation-path": [], "large-community": [ [ 65537, 1 , 1 ], [ 65537, 1 , 2 ], [ 65537, 1 , 3 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.3": [ { "nlri": "203.0.113.23/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.3" }, "asn": { "local": "65536", "peer": "65537" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65537 ], "confederation-path": [], "large-community": [ [ 65537, 0 , 1 ], [ 65537, 1 , 0 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.3": [ { "nlri": "203.0.113.25/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.3" }, "asn": { "local": "65536", "peer": "65537" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65537 ], "confederation-path": [], "large-community": [ [ 65535, 1 , 1 ], [ 4294967295, 4294967295 , 4294967295 ], [ 0, 0 , 0 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.3": [ { "nlri": "203.0.113.26/32" } ] } } } } } }
```

:white_check_mark: GoBGP, send correctly formatted large communities

:white_check_mark: GoBGP, send 1, 2 or 3 large communities (203.0.113.21/32, 203.0.113.22/32, 203.0.113.23/32)

:white_check_mark: GoBGP, send reserved values (203.0.113.26/32)

:white_check_mark: ExaBGP, receive and display large communities with routes

:white_check_mark: ExaBGP, receive 1, 2 or 3 large communities (203.0.113.21/32, 203.0.113.22/32, 203.0.113.23/32)

:white_check_mark: ExaBGP, textual representation, integers not omitted, even when zero (203.0.113.25/32)

:white_check_mark: ExaBGP, display reserved values (203.0.113.26/32)

BIRD:

```
bird> show route all protocol GoBGP
203.0.113.26/32    unreachable [GoBGP 17:36:23 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65535, 1, 1) (4294967295, 4294967295, 4294967295) (0, 0, 0)
203.0.113.25/32    unreachable [GoBGP 17:34:08 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65537, 0, 1) (65537, 1, 0)
203.0.113.22/32    unreachable [GoBGP 17:34:08 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65537, 1, 1) (65537, 1, 2)
203.0.113.23/32    unreachable [GoBGP 17:34:08 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65537, 1, 1) (65537, 1, 2) (65537, 1, 3)
203.0.113.21/32    unreachable [GoBGP 17:34:08 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65537, 1, 1)
```

:white_check_mark: BIRD, receive and display large communities with routes

:white_check_mark: BIRD, receive 1, 2 or 3 large communities (203.0.113.21/32, 203.0.113.22/32, 203.0.113.23/32)

:white_check_mark: BIRD, textual representation, integers not omitted, even when zero (203.0.113.25/32)

:white_check_mark: BIRD, display reserved values (203.0.113.26/32)

pmacct:

```
{..., "peer_ip_src": "192.0.2.3", "event_type": "dump_init", "dump_period": 60}
{..., "ip_prefix": "203.0.113.21/32", ..., "lcomms": "65537:1:1", ...}
{..., "ip_prefix": "203.0.113.22/32", ..., "lcomms": "65537:1:1 65537:1:2", ...}
{..., "ip_prefix": "203.0.113.23/32", ..., "lcomms": "65537:1:1 65537:1:2 65537:1:3", ...}
{..., "ip_prefix": "203.0.113.25/32", ..., "lcomms": "65537:0:1 65537:1:0", ...}
{..., "ip_prefix": "203.0.113.26/32", ..., "lcomms": "0:0:0 65535:1:1 4294967295:4294967295:4294967295", ...}
{..., "peer_ip_src": "192.0.2.3", "event_type": "dump_close", "entries": 5, "tables": 1}
```

:white_check_mark: pmacct, receive and display large communities with routes

:white_check_mark: pmacct, receive 1, 2 or 3 large communities (203.0.113.21/32, 203.0.113.22/32, 203.0.113.23/32)

:white_check_mark: pmacct, textual representation, integers not omitted, even when zero (203.0.113.25/32)

:white_check_mark: pmacct, display reserved values (203.0.113.26/32)

### BIRD announcing

GoBGP:

```
root@gobgp:/go# gobgp neighbor 192.0.2.4 adj-in
    Network             Next Hop             AS_PATH              Age        Attrs
    203.0.113.31/32     192.0.2.4            65538                00:00:01   [{Origin: i} {LargeCommunity: [ 65538:1:1]}]
    203.0.113.32/32     192.0.2.4            65538                00:00:01   [{Origin: i} {LargeCommunity: [ 65538:1:1, 65538:1:2]}]
    203.0.113.33/32     192.0.2.4            65538                00:00:01   [{Origin: i} {LargeCommunity: [ 65538:1:1, 65538:1:2, 65538:1:3]}]
    203.0.113.35/32     192.0.2.4            65538                00:00:01   [{Origin: i} {LargeCommunity: [ 65538:0:1, 65538:1:0]}]
    203.0.113.36/32     192.0.2.4            65538                00:00:01   [{Origin: i} {LargeCommunity: [ 0:0:0, 65535:1:1, 4294967295:4294967295:4294967295]}]
```

:white_check_mark: BIRD, send correctly formatted large communities

:white_check_mark: BIRD, send 1, 2 or 3 large communities (203.0.113.31/32, 203.0.113.32/32, 203.0.113.33/32)

:white_check_mark: BIRD, send reserved values (203.0.113.36/32)

:white_check_mark: GoBGP, receive and display large communities with routes

:white_check_mark: GoBGP, receive 1, 2 or 3 large communities (203.0.113.31/32, 203.0.113.32/32, 203.0.113.33/32)

:white_check_mark: GoBGP, textual representation, integers not omitted, even when zero (203.0.113.35/32)

:white_check_mark: GoBGP, display reserved values (203.0.113.36/32)

ExaBGP:

```
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65538 ], "confederation-path": [], "large-community": [ [ 0, 0 , 0 ], [ 65535, 1 , 1 ], [ 4294967295, 4294967295 , 4294967295 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.36/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65538 ], "confederation-path": [], "large-community": [ [ 65538, 0 , 1 ], [ 65538, 1 , 0 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.35/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65538 ], "confederation-path": [], "large-community": [ [ 65538, 1 , 1 ], [ 65538, 1 , 2 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.32/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65538 ], "confederation-path": [], "large-community": [ [ 65538, 1 , 1 ], [ 65538, 1 , 2 ], [ 65538, 1 , 3 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.33/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65538 ], "confederation-path": [], "large-community": [ [ 65538, 1 , 1 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.31/32" } ] } } } } } }
```

:white_check_mark: ExaBGP, receive and display large communities with routes

:white_check_mark: ExaBGP, receive 1, 2 or 3 large communities (203.0.113.31/32, 203.0.113.32/32, 203.0.113.33/32)

:white_check_mark: ExaBGP, textual representation, integers not omitted, even when zero (203.0.113.35/32)

:white_check_mark: ExaBGP, display reserved values (203.0.113.26/32)

pmacct:

```
{..., "peer_ip_src": "192.0.2.4", "event_type": "dump_init", "dump_period": 60}
{..., "ip_prefix": "203.0.113.31/32", ..., "lcomms": "65538:1:1", ...}
{..., "ip_prefix": "203.0.113.32/32", ..., "lcomms": "65538:1:1 65538:1:2", ...}
{..., "ip_prefix": "203.0.113.33/32", ..., "lcomms": "65538:1:1 65538:1:2 65538:1:3", ...}
{..., "ip_prefix": "203.0.113.35/32", ..., "lcomms": "65538:0:1 65538:1:0", ...}
{..., "ip_prefix": "203.0.113.36/32", ..., "lcomms": "0:0:0 65535:1:1 4294967295:4294967295:4294967295", ...}
{..., "peer_ip_src": "192.0.2.4", "event_type": "dump_close", "entries": 5, "tables": 1}
{..., "peer_ip_src": "192.0.2.4", "event_type": "dump_close", "entries": 5, "tables": 2}
```

:white_check_mark: pmacct, receive and display large communities with routes

:white_check_mark: pmacct, receive 1, 2 or 3 large communities (203.0.113.31/32, 203.0.113.32/32, 203.0.113.33/32)

:white_check_mark: pmacct, textual representation, integers not omitted, even when zero (203.0.113.35/32)

:white_check_mark: pmacct, display reserved values (203.0.113.36/32)

### Quagga announcing

GoBGP:

```
root@gobgp:/go# gobgp neighbor 192.0.2.6 adj-in
    Network             Next Hop             AS_PATH              Age        Attrs
    203.0.113.41/32     192.0.2.6            65539                00:00:21   [{Origin: i} {Med: 0} {LargeCommunity: [ 65539:1:1]}]
    203.0.113.42/32     192.0.2.6            65539                00:00:21   [{Origin: i} {Med: 0} {LargeCommunity: [ 65539:1:1, 65539:1:2]}]
    203.0.113.43/32     192.0.2.6            65539                00:00:21   [{Origin: i} {Med: 0} {LargeCommunity: [ 65539:1:1, 65539:1:2, 65539:1:3]}]
    203.0.113.45/32     192.0.2.6            65539                00:00:21   [{Origin: i} {Med: 0} {LargeCommunity: [ 65539:0:1, 65539:1:0]}]
    203.0.113.46/32     192.0.2.6            65539                00:00:21   [{Origin: i} {Med: 0} {LargeCommunity: [ 65535:1:1, 4294967295:4294967295:4294967295]}]
```

ExaBGP:

```
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.6" }, "asn": { "local": "65536", "peer": "65539" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65539 ], "confederation-path": [], "med": 0, "large-community": [ [ 65539, 1 , 1 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.6": [ { "nlri": "203.0.113.41/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.6" }, "asn": { "local": "65536", "peer": "65539" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65539 ], "confederation-path": [], "med": 0, "large-community": [ [ 65539, 1 , 1 ], [ 65539, 1 , 2 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.6": [ { "nlri": "203.0.113.42/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.6" }, "asn": { "local": "65536", "peer": "65539" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65539 ], "confederation-path": [], "med": 0, "large-community": [ [ 65539, 1 , 1 ], [ 65539, 1 , 2 ], [ 65539, 1 , 3 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.6": [ { "nlri": "203.0.113.43/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.6" }, "asn": { "local": "65536", "peer": "65539" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65539 ], "confederation-path": [], "med": 0, "large-community": [ [ 65539, 0 , 1 ], [ 65539, 1 , 0 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.6": [ { "nlri": "203.0.113.45/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.6" }, "asn": { "local": "65536", "peer": "65539" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "igp", "as-path": [ 65539 ], "confederation-path": [], "med": 0, "large-community": [ [ 0, 0 , 0 ], [ 65535, 1 , 1 ], [ 4294967295, 4294967295 , 4294967295 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.6": [ { "nlri": "203.0.113.46/32" } ] } } } } } }
```

Bird:

```
bird> show route all protocol Quagga
203.0.113.46/32    unreachable [Quagga 17:49:55 from 192.0.2.6] * (100/-) [AS65539i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65539
        BGP.next_hop: 192.0.2.6
        BGP.med: 0
        BGP.local_pref: 100
        BGP.large_community: (0, 0, 0) (65535, 1, 1) (4294967295, 4294967295, 4294967295)
203.0.113.45/32    unreachable [Quagga 17:49:55 from 192.0.2.6] * (100/-) [AS65539i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65539
        BGP.next_hop: 192.0.2.6
        BGP.med: 0
        BGP.local_pref: 100
        BGP.large_community: (65539, 0, 1) (65539, 1, 0)
203.0.113.42/32    unreachable [Quagga 17:49:55 from 192.0.2.6] * (100/-) [AS65539i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65539
        BGP.next_hop: 192.0.2.6
        BGP.med: 0
        BGP.local_pref: 100
        BGP.large_community: (65539, 1, 1) (65539, 1, 2)
203.0.113.43/32    unreachable [Quagga 17:49:55 from 192.0.2.6] * (100/-) [AS65539i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65539
        BGP.next_hop: 192.0.2.6
        BGP.med: 0
        BGP.local_pref: 100
        BGP.large_community: (65539, 1, 1) (65539, 1, 2) (65539, 1, 3)
203.0.113.41/32    unreachable [Quagga 17:49:55 from 192.0.2.6] * (100/-) [AS65539i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65539
        BGP.next_hop: 192.0.2.6
        BGP.med: 0
        BGP.local_pref: 100
        BGP.large_community: (65539, 1, 1)
```

:white_check_mark: Quagga, send correctly formatted large communities

:white_check_mark: Quagga, send 1, 2 or 3 large communities (203.0.113.41/32, 203.0.113.42/32, 203.0.113.43/32)

:white_check_mark: Quagga, send reserved values (203.0.113.46/32)
