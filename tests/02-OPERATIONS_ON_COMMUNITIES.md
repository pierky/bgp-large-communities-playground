# Operations on communities

## Scenario

- ExaBGP, 192.0.2.2, 65536
- GoBGP, 192.0.2.3, 65537
- BIRD, 192.0.2.4, 65538
- GoBGP_Receiver, 192.0.2.5, 65551

GoBGP announces the following prefixes:

- 203.0.113.21, 1 large community (origin-ASN:1:1): receiver is supposed to add another large community (receiving-ASN:10:10).
- 203.0.113.22, 1 large community (origin-ASN:2:2): receiver is supposed to add two large communities (receiving-ASN:20:20 and receiving-ASN:200:200).
- 203.0.113.23, 1 large community (origin-ASN:3:3): receiver is supposed to delete the large community.
- 203.0.113.24, 2 large communities (origin-ASN:4:4 and origin-ASN:5:5): receiver is supposed to delete the large communities.
- 203.0.113.25, 1 large community (origin-ASN:6:6): receiver is supposed to match prefixes on the basis of this large community and set local-pref 66.

```
gobgp global rib add -a ipv4 203.0.113.21/32 large-community 65537:1:1
gobgp global rib add -a ipv4 203.0.113.22/32 large-community 65537:2:2
gobgp global rib add -a ipv4 203.0.113.23/32 large-community 65537:3:3
gobgp global rib add -a ipv4 203.0.113.24/32 large-community 65537:4:4,65537:5:5
gobgp global rib add -a ipv4 203.0.113.25/32 large-community 65537:6:6
```

## Results

GoBGP > BIRD > ExaBGP:

```
bird> show route all protocol GoBGP
203.0.113.24/32    unreachable [GoBGP 18:44:32 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community:
203.0.113.25/32    unreachable [GoBGP 18:44:32 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 66
        BGP.large_community: (65537, 6, 6)
203.0.113.22/32    unreachable [GoBGP 18:44:31 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65537, 2, 2) (65538, 20, 20) (65538, 200, 200)
203.0.113.23/32    unreachable [GoBGP 18:44:32 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community:
203.0.113.21/32    unreachable [GoBGP 18:44:30 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65537, 1, 1) (65538, 10, 10)
```

:white_check_mark: BIRD, add 1 large community (203.0.113.21/32)

:white_check_mark: BIRD, add 2 large communities (203.0.113.22/32)

:white_check_mark: BIRD, delete 1 large community (203.0.113.23/32)

:white_check_mark: BIRD, delete 2 large communities (203.0.113.24/32)

:white_check_mark: BIRD, match prefixes on the basis of large communities and perform action (203.0.113.25/32)

ExaBGP (receives prefixes from BIRD):

```
{ "exabgp": "3.5.0", "time": 1475601123.41, "host" : "exabgp", "pid" : 5, "ppid" : 1, "counter": 1, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65538, 65537 ], "confederation-path": [], "large-community": [ [ 65537, 1 , 1 ], [ 65538, 10 , 10 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.21/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1475601123.41, "host" : "exabgp", "pid" : 5, "ppid" : 1, "counter": 2, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65538, 65537 ], "confederation-path": [], "large-community": [ [ 65537, 2 , 2 ], [ 65538, 20 , 20 ], [ 65538, 200 , 200 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.22/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1475601123.42, "host" : "exabgp", "pid" : 5, "ppid" : 1, "counter": 3, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65538, 65537 ], "confederation-path": [] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.23/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1475601123.42, "host" : "exabgp", "pid" : 5, "ppid" : 1, "counter": 4, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65538, 65537 ], "confederation-path": [] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.24/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1475601123.42, "host" : "exabgp", "pid" : 5, "ppid" : 1, "counter": 5, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65538, 65537 ], "confederation-path": [], "large-community": [ [ 65537, 6 , 6 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.25/32" } ] } } } } } }
```

GoBGP > GoBGP_Receiver > ExaBGP:

```
root@gobgpReceiver:/go# gobgp global rib
    Network             Next Hop             AS_PATH              Age        Attrs
    *>  203.0.113.21/32     192.0.2.3            65537                00:00:07   [{Origin: ?} {LargeCommunity: [ 65537:1:1, 65551:10:10]}]
    *>  203.0.113.22/32     192.0.2.3            65537                00:00:07   [{Origin: ?} {LargeCommunity: [ 65537:2:2, 65551:20:20, 65551:200:200]}]
    *>  203.0.113.23/32     192.0.2.3            65537                00:00:07   [{Origin: ?} {LargeCommunity: [ ]}]
    *>  203.0.113.24/32     192.0.2.3            65537                00:00:07   [{Origin: ?} {LargeCommunity: [ ]}]
    *>  203.0.113.25/32     192.0.2.3            65537                00:00:07   [{Origin: ?} {LocalPref: 66} {LargeCommunity: [ 65537:6:6]}]
```

:white_check_mark: GoBGP, add 1 large community (203.0.113.21/32)

:white_check_mark: GoBGP, add 2 large communities (203.0.113.22/32)

:white_check_mark: GoBGP, delete 1 large community (203.0.113.23/32)

:white_check_mark: GoBGP, delete 2 large communities (203.0.113.24/32)

:white_check_mark: GoBGP, match prefixes on the basis of large communities and perform action (203.0.113.25/32)

ExaBGP (receives prefixes from GoBGP_Receiver):

```
{ "exabgp": "3.5.0", "time": 1476197237.2, "host" : "exabgp", "pid" : 316, "ppid" : 1, "counter": 1, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.5" }, "asn": { "local": "65536", "peer": "65551" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65551, 65537 ], "confederation-path": [], "large-community": [ [ 65537, 1 , 1 ], [ 65551, 10 , 10 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.5": [ { "nlri": "203.0.113.21/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1476197237.2, "host" : "exabgp", "pid" : 316, "ppid" : 1, "counter": 2, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.5" }, "asn": { "local": "65536", "peer": "65551" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65551, 65537 ], "confederation-path": [], "large-community": [ [ 65537, 2 , 2 ], [ 65551, 20 , 20 ], [ 65551, 200 , 200 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.5": [ { "nlri": "203.0.113.22/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1476197237.2, "host" : "exabgp", "pid" : 316, "ppid" : 1, "counter": 3, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.5" }, "asn": { "local": "65536", "peer": "65551" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65551, 65537 ], "confederation-path": [], "large-community": [  ] }, "announce": { "ipv4 unicast": { "192.0.2.5": [ { "nlri": "203.0.113.23/32" }, { "nlri": "203.0.113.24/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1476197237.21, "host" : "exabgp", "pid" : 316, "ppid" : 1, "counter": 4, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.5" }, "asn": { "local": "65536", "peer": "65551" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65551, 65537 ], "confederation-path": [], "large-community": [ [ 65537, 6 , 6 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.5": [ { "nlri": "203.0.113.25/32" } ] } } } } } }
```

Nota bene: third row contains both 203.0.113.23/32 and 203.0.113.24/32 NLRI.
