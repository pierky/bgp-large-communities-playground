# Malformed BGP Large Community attribute handling

## Scenario

- ExaBGP, 192.0.2.2, 65536
- GoBGP, 192.0.2.3, 65537
- BIRD, 192.0.2.4, 65538
- pmacct, 192.0.2.5, any
- ExaBGP_Receiver, 192.0.2.102, 65551
- Quagga, 192.0.2.6, 65539

ExaBGP:

1. announces 203.0.113.11/32 with 2 valid large communities (1:2:3, 4:5:6)
2. waits 60 seconds
3. announces 203.0.113.12/32 with above valid large communities (1:2:3, 4:5:6)
4. waits 60 seconds
5. announces the same prefix above with a malformed BGP Large Community attribute

## Results

### GoBGP

tcpdump on GoBGP host:

```
    192.0.2.2.34056 > 192.0.2.3.179: Flags [P.], cksum 0x8474 (incorrect -> 0x9a5b), seq 402:474, ack 141, win 229, options [nop,nop,TS val 754925709 ecr 754924708], length 72: BGP
        Update Message (2), length: 72
          Origin (1), length: 1, Flags [T]: IGP
            0x0000:  00
          AS Path (2), length: 6, Flags [T]: 65536
            0x0000:  0201 0001 0000
          Next Hop (3), length: 4, Flags [T]: 192.0.2.2
            0x0000:  c000 0202
          Large Community (32), length: 21, Flags [OTP]: invalid len
            0x0000:  0000 0001 0000 0002 0000 0003 0000 0004
            0x0010:  0000 0005 ff
          Updated routes:
            203.0.113.12/32
```

GoBGP log:

```
time="2017-10-03T16:21:28Z" level=warning msg="the received Update message was treated as withdraw" Key=192.0.2.2 State=BGP_FSM_ESTABLISHED Topic=Peer error="large communities length isn't correct"
```

GoBGP output from 'gobgp global rib':

```
root@gobgp:/go# gobgp global rib
   Network              Next Hop             AS_PATH              Age        Attrs
*> 203.0.113.11/32      192.0.2.2            65536                00:01:12   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
*> 203.0.113.12/32      192.0.2.2            65536                00:00:12   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
root@gobgp:/go# gobgp global rib
   Network              Next Hop             AS_PATH              Age        Attrs
*> 203.0.113.11/32      192.0.2.2            65536                00:01:32   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
*> 203.0.113.12/32      192.0.2.2            65536                00:00:32   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
root@gobgp:/go# gobgp global rib
   Network              Next Hop             AS_PATH              Age        Attrs
*> 203.0.113.11/32      192.0.2.2            65536                00:02:04   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
```

:white_check_mark: GoBGP, malformed Large Communities attributes, treat-as-withdraw approach

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

### Quagga

```
BGP: 192.0.2.2: Attribute LARGE_COMMUNITY, parse error - treating as withdrawal
BGP: 192.0.2.2 rcvd UPDATE with errors in attr(s)!! Withdrawing route.
BGP: 192.0.2.2 rcvd UPDATE w/ attr: nexthop 192.0.2.2, origin i, path 65536
```

```
--------------------------------------
    Network             Next Hop             AS_PATH              Age        Attrs
    203.0.113.11/32     192.0.2.6            65539 65536          00:01:45   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
    203.0.113.12/32     192.0.2.6            65539 65536          00:00:45   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
--------------------------------------
    Network             Next Hop             AS_PATH              Age        Attrs
    203.0.113.11/32     192.0.2.6            65539 65536          00:01:46   [{Origin: i} {LargeCommunity: [ 1:2:3, 4:5:6]}]
--------------------------------------
```

:white_check_mark: Quagga, malformed Large Communities attributes, treat-as-withdraw approach
