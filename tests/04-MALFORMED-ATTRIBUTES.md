# Malformed large BGP community attribute handling

## Scenario

- ExaBGP, 192.0.2.2, 65536
- GoBGP, 192.0.2.3, 65537
- BIRD, 192.0.2.4, 65538
- pmacct, 192.0.2.5, any
- ExaBGP_Receiver, 192.0.2.102, 65551

ExaBGP:

1. announces 203.0.113.11/32 with 2 valid large communities (1:2:3, 4:5:6)
2. waits 2 seconds
3. announces 203.0.113.12/32 with above valid large communities (1:2:3, 4:5:6)
4. waits 2 seconds
5. announces the same prefix above with a malformed large BGP community attribute

## Results

### GoBGP

tcpdump on GoBGP host:

```
    192.0.2.2.40208 > 192.0.2.3.179: Flags [P.], cksum 0x8477 (incorrect -> 0x4292), seq 200:275, ack 65, win 457, options [nop,nop,TS val 5123423 ecr 5123179], length 75: BGP, length: 75
        Update Message (2), length: 75
          Origin (1), length: 1, Flags [T]: IGP
          AS Path (2), length: 6, Flags [T]: 65536
          Next Hop (3), length: 4, Flags [T]: 192.0.2.2
          Unknown Attribute (30), length: 24, Flags [OTP]:
            no Attribute 30 decoder
            0x0000:  0000 0001 0000 0002 0000 0003 0000 0004
            0x0010:  0000 0005 0000 0006
          Updated routes:
            203.0.113.11/32
    192.0.2.2.40208 > 192.0.2.3.179: Flags [P.], cksum 0x8477 (incorrect -> 0x3e5e), seq 275:350, ack 65, win 457, options [nop,nop,TS val 5123924 ecr 5123423], length 75: BGP, length: 75
        Update Message (2), length: 75
          Origin (1), length: 1, Flags [T]: IGP
          AS Path (2), length: 6, Flags [T]: 65536
          Next Hop (3), length: 4, Flags [T]: 192.0.2.2
          Unknown Attribute (30), length: 24, Flags [OTP]:
            no Attribute 30 decoder
            0x0000:  0000 0001 0000 0002 0000 0003 0000 0004
            0x0010:  0000 0005 0000 0006
          Updated routes:
            203.0.113.12/32
    192.0.2.2.40208 > 192.0.2.3.179: Flags [P.], cksum 0x8474 (incorrect -> 0x2f47), seq 350:422, ack 65, win 457, options [nop,nop,TS val 5124424 ecr 5123924], length 72: BGP, length: 72
        Update Message (2), length: 72
          Origin (1), length: 1, Flags [T]: IGP
          AS Path (2), length: 6, Flags [T]: 65536
          Next Hop (3), length: 4, Flags [T]: 192.0.2.2
          Unknown Attribute (30), length: 21, Flags [OTP]:
            no Attribute 30 decoder
            0x0000:  0000 0001 0000 0002 0000 0003 0000 0004
            0x0010:  0000 0005 ff
          Updated routes:
            203.0.113.12/32
    192.0.2.3.179 > 192.0.2.2.40208: Flags [P.], cksum 0x8441 (incorrect -> 0xf9e0), seq 65:86, ack 422, win 470, options [nop,nop,TS val 5124425 ecr 5124424], length 21: BGP, length: 21
        Notification Message (3), length: 21, UPDATE Message Error (3), subcode Attribute Length Error (5)
    192.0.2.3.179 > 192.0.2.2.40208: Flags [F.], cksum 0x842c (incorrect -> 0x0200), seq 86, ack 422, win 470, options [nop,nop,TS val 5124425 ecr 5124424], length 0

```

GoBGP log:

```
WARN[3024] malformed BGP message  Key=192.0.2.2 State=BGP_FSM_ESTABLISHED Topic=Peer error=large communities length isn't correct
WARN[3024] sent notification      Data=&{Header:{Marker:[] Len:21 Type:3} Body:0xc4203f5040} Key=192.0.2.2 State=BGP_FSM_ESTABLISHED Topic=Peer
INFO[3024] Peer Down              Key=192.0.2.2 Reason=notification-sent code 3(update) subcode 5(attribute length error) State=BGP_FSM_ESTABLISHED Topic=Peer
```

ExaBGP log:

```
INFO     | 272    | network       | Peer       192.0.2.3 ASN 65537   out loop, peer reset, message [notification received (3,5)] error[UPDATE message error / Attribute Length Error]
```

:x: GoBGP, malformed Large Communities attributes, treat-as-withdraw approach

### BIRD

```
root@bird:~# while true; do birdcl show route all; echo "--------------------------------------"; sleep 1; done
BIRD 1.6.2 ready.
203.0.113.11/32    unreachable [ExaBGP 13:19:11 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (1, 2, 3) (4, 5, 6)
--------------------------------------
BIRD 1.6.2 ready.
203.0.113.11/32    unreachable [ExaBGP 13:19:11 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (1, 2, 3) (4, 5, 6)
--------------------------------------
BIRD 1.6.2 ready.
203.0.113.12/32    unreachable [ExaBGP 13:19:13 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (1, 2, 3) (4, 5, 6)
203.0.113.11/32    unreachable [ExaBGP 13:19:11 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (1, 2, 3) (4, 5, 6)
--------------------------------------
BIRD 1.6.2 ready.
203.0.113.12/32    unreachable [ExaBGP 13:19:13 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (1, 2, 3) (4, 5, 6)
203.0.113.11/32    unreachable [ExaBGP 13:19:11 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (1, 2, 3) (4, 5, 6)
--------------------------------------
BIRD 1.6.2 ready.
203.0.113.11/32    unreachable [ExaBGP 13:19:11 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (1, 2, 3) (4, 5, 6)
--------------------------------------
```

BIRD log:

```
2016-10-13 16:22:00 <TRACE> ExaBGP: Got UPDATE
2016-10-13 16:22:00 <TRACE> ExaBGP: Got END-OF-RIB
2016-10-13 16:22:00 <TRACE> ExaBGP: Got UPDATE
2016-10-13 16:22:00 <TRACE> ExaBGP > added [best] 203.0.113.11/32 unreachable
2016-10-13 16:22:00 <TRACE> ExaBGP < rejected by protocol 203.0.113.11/32 unreachable
2016-10-13 16:22:01 <TRACE> ExaBGP: Got UPDATE
2016-10-13 16:22:01 <TRACE> ExaBGP > added [best] 203.0.113.12/32 unreachable
2016-10-13 16:22:01 <TRACE> ExaBGP < rejected by protocol 203.0.113.12/32 unreachable
2016-10-13 16:22:04 <TRACE> ExaBGP: Got UPDATE
2016-10-13 16:22:04 <WARN> ExaBGP: Attribute large_community is malformed, withdrawing update
2016-10-13 16:22:04 <TRACE> ExaBGP > removed [sole] 203.0.113.12/32 unreachable
```

GoBGP (receives prefixes from BIRD):

```
root@gobgp:/go# while true; do gobgp global rib; echo "--------------------------------------"; sleep 1; done
    Network             Next Hop             AS_PATH              Age        Attrs
*>  203.0.113.11/32     192.0.2.4            65538 65536          00:00:01   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
--------------------------------------
    Network             Next Hop             AS_PATH              Age        Attrs
*>  203.0.113.11/32     192.0.2.4            65538 65536          00:00:02   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
*>  203.0.113.12/32     192.0.2.4            65538 65536          00:00:00   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
--------------------------------------
    Network             Next Hop             AS_PATH              Age        Attrs
*>  203.0.113.11/32     192.0.2.4            65538 65536          00:00:03   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
*>  203.0.113.12/32     192.0.2.4            65538 65536          00:00:01   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
--------------------------------------
    Network             Next Hop             AS_PATH              Age        Attrs
*>  203.0.113.11/32     192.0.2.4            65538 65536          00:00:04   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
```

:white_check_mark: BIRD, malformed Large Communities attributes, treat-as-withdraw approach

### ExaBGP

ExaBGP_Receiver JSON output (exabgp/received):

```
{ ..., "type": "update", "neighbor": { ..., "message": { "update": { ..., "large-community": [ [ 1, 2 , 3 ], [ 4, 5 , 6 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.2": [ { "nlri": "203.0.113.11/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { ..., "message": { "update": { ..., "large-community": [ [ 1, 2 , 3 ], [ 4, 5 , 6 ] ] }, "announce": { "ipv4 unicast": { "192.0.2.2": [ { "nlri": "203.0.113.12/32" } ] } } } } } }
{ ..., "type": "update", "neighbor": { ..., "message": { "update": { ..., "error": "treat-as-withdraw" }, "withdraw": { "ipv4 unicast": [ { "nlri": "203.0.113.12/32" } ] } } } } }
```

:white_check_mark: ExaBGP, malformed Large Communities attributes, treat-as-withdraw approach

### pmacct

Content of bgp_table_dump_file after ExaBGP completed its job:

```
# cat pmacct/bgp.log
{..., "event_type": "dump_init", "dump_period": 60}
{..., "event_type": "dump", "ip_prefix": "203.0.113.11/32", ..., "lcomms": "1:2:3 4:5:6", "origin": 0, "local_pref": 100}
{..., "event_type": "dump", "ip_prefix": "203.0.113.12/32", ..., "origin": 0, "local_pref": 100}
{..., "event_type": "dump_close", "entries": 2, "tables": 1}
```

:x: pmacct, malformed Large Communities attributes, treat-as-withdraw approach
