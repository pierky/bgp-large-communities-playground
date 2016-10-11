# Duplicate large communities handling

## Scenario

- ExaBGP, 192.0.2.2, 65536
- GoBGP, 192.0.2.3, 65537
- BIRD, 192.0.2.4, 65538
- GoBGP_receiver, 192.0.2.5, 65539

They all announce the following prefix (except GoBGP_receiver):

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

tcpdump on GoBGP host (UPDATEs toward BIRD and ExaBGP):

```
    192.0.2.3.179 > 192.0.2.4.56832: Flags [P.], cksum 0x8479 (incorrect -> 0xc4b6), seq 65:140, ack 113, win 453, options [nop,nop,TS val 908952 ecr 908090], length 75: BGP, length: 75
        Update Message (2), length: 75
          Origin (1), length: 1, Flags [T]: Incomplete
          AS Path (2), length: 6, Flags [T]: 65537
          Next Hop (3), length: 4, Flags [T]: 192.0.2.3
          Unknown Attribute (30), length: 24, Flags [OT]:
            no Attribute 30 decoder
            0x0000:  0001 0001 0000 0001 0000 0001 0001 0001
            0x0010:  0000 0001 0000 0001
          Updated routes:
            203.0.113.24/32
    192.0.2.3.179 > 192.0.2.2.34662: Flags [P.], cksum 0x8477 (incorrect -> 0x9189), seq 65:140, ack 200, win 470, options [nop,nop,TS val 908952 ecr 903084], length 75: BGP, length: 75
        Update Message (2), length: 75
          Origin (1), length: 1, Flags [T]: Incomplete
          AS Path (2), length: 6, Flags [T]: 65537
          Next Hop (3), length: 4, Flags [T]: 192.0.2.3
          Unknown Attribute (30), length: 24, Flags [OT]:
            no Attribute 30 decoder
            0x0000:  0001 0001 0000 0001 0000 0001 0001 0001
            0x0010:  0000 0001 0000 0001
          Updated routes:
            203.0.113.24/32
```

:x: GoBGP, duplicate large communities not transmitted (203.0.113.24/32)

ExaBGP JSON dump:

```
{ "exabgp": "3.5.0", "time": 1476179306.15, "host" : "exabgp", "pid" : 204, "ppid" : 1, "counter": 1, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.3" }, "asn": { "local": "65536", "peer": "65537" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65537 ], "confederation-path": [], "large-community": [ [ 65537, 1 , 1 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.3": [ { "nlri": "203.0.113.24/32" } ] } } } } } }
{ "exabgp": "3.5.0", "time": 1476179306.16, "host" : "exabgp", "pid" : 204, "ppid" : 1, "counter": 1, "type": "update", "neighbor": { "address": { "local": "192.0.2.2", "peer": "192.0.2.4" }, "asn": { "local": "65536", "peer": "65538" }, "direction": "receive", "message": { "update": { "attribute": { "origin": "incomplete", "as-path": [ 65538, 65537 ], "confederation-path": [], "large-community": [ [ 65537, 1 , 1 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.4": [ { "nlri": "203.0.113.24/32" } ] } } } } } }
```

:white_check_mark: ExaBGP, remove duplicate large communities on receipt (203.0.113.24/32)

BIRD:

tcpdump on BIRD host (UPDATE toward ExaBGP):

```
    192.0.2.4.179 > 192.0.2.2.35891: Flags [P.], cksum 0x847c (incorrect -> 0xbbfd), seq 48:127, ack 38, win 470, options [nop,nop,TS val 1047320 ecr 1043030], length 79: BGP, length: 79
        Update Message (2), length: 79
          Origin (1), length: 1, Flags [T]: Incomplete
          AS Path (2), length: 10, Flags [T]: 65538 65537
          Next Hop (3), length: 4, Flags [T]: 192.0.2.4
          Unknown Attribute (30), length: 24, Flags [OT]:
            no Attribute 30 decoder
            0x0000:  0001 0001 0000 0001 0000 0001 0001 0001
            0x0010:  0000 0001 0000 0001
          Updated routes:
            203.0.113.24/32
```

```
bird> show route all protocol GoBGP
203.0.113.24/32    unreachable [GoBGP 09:48:25 from 192.0.2.3] * (100/-) [AS65537?]
        Type: BGP unicast univ
        BGP.origin: Incomplete
        BGP.as_path: 65537
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65537, 1, 1) (65537, 1, 1)
```

:x: BIRD, remove duplicate large communities on receipt (203.0.113.24/32)

GoBGP_Receiver:

```
root@gobgpReceiver:/go# gobgp neighbor 192.0.2.3 adj-in
    Network             Next Hop             AS_PATH              Age        Attrs
    203.0.113.24/32     192.0.2.3            65537                00:00:51   [{Origin: ?} {LargeCommunity: [ 65537:1:1, 65537:1:1]}]
```

:x: GoBGP, remove duplicate large communities on receipt (203.0.113.24/32)

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
