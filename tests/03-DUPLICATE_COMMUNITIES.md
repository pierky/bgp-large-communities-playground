# Duplicate large communities handling

## Scenario

- ExaBGP, 192.0.2.2, 65536
- GoBGP, 192.0.2.3, 65537
- BIRD, 192.0.2.4, 65538
- pmacct, 192.0.2.5, any
- GoBGP_receiver, 192.0.2.103, 65551

They all announce the following prefix (except pmacct and GoBGP_receiver):

- 203.0.113.x4, 1 duplicate large community configured (ASN:1:1 ASN:1:1)

x = 1 for ExaBGP, 2 for GoBGP, 3 for BIRD.

ExaBGP and BIRD read the prefix to announce from their [configuration files](03/); GoBGP announces it via CLI:

```
gobgp global rib add -a ipv4 203.0.113.24/32 large-community 65537:1:1,65537:1:1
```

## Results

### ExaBGP announcing

tcpdump on ExaBGP host (UPDATEs toward BIRD and GoBGP):

```
    192.0.2.2.35718 > 192.0.2.4.179: Flags [P.], cksum 0x846c (incorrect -> 0xa221), seq 177:240, ack 67, win 457, options [nop,nop,TS val 778507 ecr 778507], length 63: BGP, length: 63
        Update Message (2), length: 63
          Origin (1), length: 1, Flags [T]: IGP
          AS Path (2), length: 6, Flags [T]: 65536
          Next Hop (3), length: 4, Flags [T]: 192.0.2.2
          Unknown Attribute (30), length: 12, Flags [OT]:
            no Attribute 30 decoder
            0x0000:  0001 0000 0000 0001 0000 0001
          Updated routes:
            203.0.113.14/32
    192.0.2.2.33069 > 192.0.2.3.179: Flags [P.], cksum 0x846b (incorrect -> 0x4e73), seq 177:240, ack 65, win 457, options [nop,nop,TS val 778507 ecr 778507], length 63: BGP, length: 63
        Update Message (2), length: 63
          Origin (1), length: 1, Flags [T]: IGP
          AS Path (2), length: 6, Flags [T]: 65536
          Next Hop (3), length: 4, Flags [T]: 192.0.2.2
          Unknown Attribute (30), length: 12, Flags [OT]:
            no Attribute 30 decoder
            0x0000:  0001 0000 0000 0001 0000 0001
          Updated routes:
            203.0.113.14/32
```

:white_check_mark: ExaBGP, duplicate large communities not transmitted (203.0.113.14/32)

### GoBGP announcing

tcpdump on GoBGP host (UPDATEs toward GoBGP_Receiver, BIRD and ExaBGP):

```
    192.0.2.3.36189 > 192.0.2.103.179: Flags [P.], cksum 0x84dc (incorrect -> 0xcb9d), seq 132:207, ack 132, win 457, options [nop,nop,TS val 697398 ecr 691150], length 75: BGP, length: 75
        Update Message (2), length: 75
          Origin (1), length: 1, Flags [T]: Incomplete
          AS Path (2), length: 6, Flags [T]: 65537
          Next Hop (3), length: 4, Flags [T]: 192.0.2.3
          Unknown Attribute (32), length: 24, Flags [OT]:
            no Attribute 32 decoder
            0x0000:  0001 0001 0000 0001 0000 0001 0001 0001
            0x0010:  0000 0001 0000 0001
          Updated routes:
            203.0.113.24/32
    192.0.2.3.39844 > 192.0.2.4.179: Flags [P.], cksum 0x8479 (incorrect -> 0xb417), seq 65:140, ack 247, win 457, options [nop,nop,TS val 697398 ecr 697398], length 75: BGP, length: 75
        Update Message (2), length: 75
          Origin (1), length: 1, Flags [T]: Incomplete
          AS Path (2), length: 6, Flags [T]: 65537
          Next Hop (3), length: 4, Flags [T]: 192.0.2.3
          Unknown Attribute (32), length: 24, Flags [OT]:
            no Attribute 32 decoder
            0x0000:  0001 0001 0000 0001 0000 0001 0001 0001
            0x0010:  0000 0001 0000 0001
          Updated routes:
            203.0.113.24/32
    192.0.2.3.179 > 192.0.2.2.43883: Flags [P.], cksum 0x8477 (incorrect -> 0x65f7), seq 151:226, ack 219, win 470, options [nop,nop,TS val 697398 ecr 695732], length 75: BGP, length: 75
        Update Message (2), length: 75
          Origin (1), length: 1, Flags [T]: Incomplete
          AS Path (2), length: 6, Flags [T]: 65537
          Next Hop (3), length: 4, Flags [T]: 192.0.2.3
          Unknown Attribute (32), length: 24, Flags [OT]:
            no Attribute 32 decoder
            0x0000:  0001 0001 0000 0001 0000 0001 0001 0001
            0x0010:  0000 0001 0000 0001
          Updated routes:
            203.0.113.24/32

```

:x: GoBGP, duplicate large communities not transmitted (203.0.113.24/32)

ExaBGP JSON dump:

```
{ ..., "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.3" }, "asn": { "local": "65536", "peer": "65537" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65537 ], "confederation-path": [], "large-community": [ [ 65537, 1 , 1 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.3": [ { "nlri": "203.0.113.24/32" } ] } } } } } }
```

:white_check_mark: ExaBGP, remove duplicate large communities on receipt (203.0.113.24/32)

BIRD:

tcpdump on BIRD host (UPDATE toward ExaBGP):

```
    192.0.2.4.49861 > 192.0.2.103.179: Flags [P.], cksum 0x84e1 (incorrect -> 0x4083), seq 176:255, ack 132, win 457, options [nop,nop,TS val 697398 ecr 697398], length 79: BGP, length: 79
        Update Message (2), length: 79
          Origin (1), length: 1, Flags [T]: Incomplete
          AS Path (2), length: 10, Flags [T]: 65538 65537
          Next Hop (3), length: 4, Flags [T]: 192.0.2.4
          Unknown Attribute (32), length: 24, Flags [OT]:
            no Attribute 32 decoder
            0x0000:  0001 0001 0000 0001 0000 0001 0001 0001
            0x0010:  0000 0001 0000 0001
          Updated routes:
            203.0.113.24/32
```

```
bird> show route all protocol GoBGP
203.0.113.24/32    unreachable [GoBGP 09:20:54 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65537, 1, 1) (65537, 1, 1)
bird> show route all protocol GoBGP_Receiver
203.0.113.24/32    unreachable [GoBGP_Receiver 09:20:53 from 192.0.2.103] (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65551 65537
        BGP.next_hop: 192.0.2.103
        BGP.local_pref: 100
        BGP.large_community: (65537, 1, 1)
```

:x: BIRD, remove duplicate large communities on receipt (203.0.113.24/32 from 192.0.2.3)

:white_check_mark: GoBGP, remove duplicate large communities on receipt (203.0.113.24/32 received by BIRD from GoBGP_Receiver 192.0.2.103)

GoBGP_Receiver:

```
root@gobgpreceiver:/go# gobgp neighbor 192.0.2.3 adj-in
    Network             Next Hop             AS_PATH              Age        Attrs
    203.0.113.24/32     192.0.2.3            65537                00:10:18   [{Origin: ?} {LargeCommunity: [ 65537:1:1]}]
    203.0.113.34/32     192.0.2.3            65537 65538          00:10:43   [{Origin: i} {LargeCommunity: [ 65538:1:1]}]
```

:white_check_mark: GoBGP, remove duplicate large communities on receipt (203.0.113.24/32)

pmacct:

```
# cat pmacct/bgp.log | grep "203.0.113.24/32"
{"timestamp": "2016-10-14 16:35:00", "peer_ip_src": "192.0.2.3", "event_type": "dump", "ip_prefix": "203.0.113.24/32", "bgp_nexthop": "192.0.2.3", "as_path": "", "lcomms": "65537:1:1", "origin": 0, "local_pref": 100}
```

:white_check_mark: pmacct, remove duplicate large communities on receipt (203.0.113.24/32)

### BIRD announcing

tcpdump on BIRD (UPDATEs toware GoBGP and ExaBGP):

```
    192.0.2.4.179 > 192.0.2.3.33443: Flags [P.], cksum 0x8484 (incorrect -> 0xb8de), seq 71:157, ack 65, win 453, options [nop,nop,TS val 1134300 ecr 1134300], length 86: BGP, length: 86
        Update Message (2), length: 63
          Origin (1), length: 1, Flags [T]: IGP
          AS Path (2), length: 6, Flags [T]: 65538
          Next Hop (3), length: 4, Flags [T]: 192.0.2.4
          Unknown Attribute (30), length: 12, Flags [OT]:
            no Attribute 30 decoder
            0x0000:  0001 0002 0000 0001 0000 0001
          Updated routes:
            203.0.113.34/32
        Update Message (2), length: 23
          End-of-Rib Marker (empty NLRI)
    192.0.2.4.179 > 192.0.2.2.43119: Flags [P.], cksum 0x846c (incorrect -> 0xac81), seq 67:130, ack 177, win 470, options [nop,nop,TS val 1135988 ecr 1135988], length 63: BGP, length: 63
        Update Message (2), length: 63
          Origin (1), length: 1, Flags [T]: IGP
          AS Path (2), length: 6, Flags [T]: 65538
          Next Hop (3), length: 4, Flags [T]: 192.0.2.4
          Unknown Attribute (30), length: 12, Flags [OT]:
            no Attribute 30 decoder
            0x0000:  0001 0002 0000 0001 0000 0001
          Updated routes:
            203.0.113.34/32

```

:white_check_mark: BIRD, duplicate large communities not transmitted (203.0.113.34/32)
