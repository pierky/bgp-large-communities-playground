# Large BGP Communities playground

## What Large BGP Communities are

A good site already explains it very well: http://largebgpcommunities.net/

## This playground

Luckly many vendors and networking software authors are approaching this solution and [started implementing](http://largebgpcommunities.net/implementations/) [the draft](https://tools.ietf.org/html/draft-heitz-idr-large-community). This repository (presumptuously) wants to offer some hints to quickly have a *large-bgp-communities*-aware lab up & running on the basis of the latest code available.

Currently it supports 4 products:
- [ExaBGP](https://github.com/Exa-Networks/exabgp)
- [GoBGP](https://github.com/osrg/gobgp)
- [BIRD](http://bird.network.cz/)
- [pmacct](http://www.pmacct.net/)

[Docker images](https://hub.docker.com/u/pierky/) have been built in order to have them running on the latest Large-BGP-Communities-aware code fetched from the `master` branch of them.

### Disclaimer

These images have been created with the only purpose of being used in a "playground", for labs and interoperability tests. They do not implement any security best practice. Use them at your own risk.

### Tests

I used this Playground to run some interoperability tests and to verify implemented features among the covered tools: [here are my findings](tests/README.md).

### How to run it

```
# git clone https://github.com/pierky/large-bgp-communities-playground.git
# cd large-bgp-communities-playground/
# docker network create --subnet=192.0.2.0/24 large-bgp-communities-playground
# docker run --net large-bgp-communities-playground --ip 192.0.2.2 --hostname=exabgp -d -v `pwd`/exabgp:/etc/exabgp:rw pierky/exabgp
# docker run --net large-bgp-communities-playground --ip 192.0.2.3 --hostname=gobgp -d -v `pwd`/gobgp:/etc/gobgp:rw pierky/gobgp
# docker run --net large-bgp-communities-playground --ip 192.0.2.4 --hostname=bird -d -v `pwd`/bird:/etc/bird:rw pierky/bird
# docker run --net large-bgp-communities-playground --ip 192.0.2.5 --hostname=pmacct -d -v `pwd`/pmacct:/etc/pmacct:rw pierky/pmacct
```

This is enough to create a virtual network, have ExaBGP running on 192.0.2.2, GoBGP on 192.0.2.3 and BIRD on 192.0.2.4. The startup config files (`exabgp/exabgp.conf`, `gobgp/gobgp.conf` and `bird/bird.conf`) allow the three instances to establish BGP sessions:

```
# cat exabgp/log
...
Thu, 14 Sep 2016 17:54:57 5      network       Connected to peer neighbor 192.0.2.3 local-ip 192.0.2.2 local-as 65536 peer-as 65537 router-id 192.0.2.2 family
```

```
# cat gobgp/log
...
time="2016-09-14T17:54:57Z" level=info msg="Peer Up" Key=192.0.2.2 State="BGP_FSM_OPENCONFIRM" Topic=Peer
time="2016-09-14T17:54:57Z" level=info msg="Peer Up" Key=192.0.2.4 State="BGP_FSM_OPENCONFIRM" Topic=Peer
```

The BGP daemon built into pmacct is started too: ExaBGP is configured to setup a neighborship with it and to announce some prefixes:

```
# cat pmacct/log
...
INFO ( default/core/BGP ): [192.0.2.2] BGP peers usage: 1/2
INFO ( default/core/BGP ): [192.0.2.2] Capability: MultiProtocol [1] AFI [1] SAFI [1]

INFO ( default/core/BGP ): [192.0.2.2] Capability: 4-bytes AS [41] ASN [65536]
INFO ( default/core/BGP ): [192.0.2.2] BGP_OPEN: Asn: 65536 HoldTime: 180
```

```
# cat pmacct/bgp.log
{"event_type": "dump_init", "dump_period": 60}
{"event_type": "dump", "ip_prefix": "192.0.2.2/32", ..., "lcomms": "65536:1:1", ...}
{"event_type": "dump", "ip_prefix": "192.0.2.3/32", ..., "lcomms": "65537:1:1", ...}
{"event_type": "dump", "ip_prefix": "192.0.2.4/32", ..., "lcomms": "65538:1:1", ...}
{"event_type": "dump_close", "entries": 3, "tables": 1}
```

Commands can be run on the instances interactively, by attaching a new terminal to the Docker container, or directly from the host:

```
# # take note of the container ID of each instance
# docker ps
CONTAINER ID        IMAGE               ...
ff5c323d2118        pierky/gobgp        ...
2c46decfb88a        pierky/exabgp       ...
...
```

```
# docker exec -it ff5c323d2118 bash
root@gpbgp:/go# gobgp neighbor
Peer         AS  Up/Down State       |#Advertised Received Accepted
192.0.2.2 65536 00:02:19 Establ      |          0        1        1
192.0.2.4 65538 00:01:57 Establ      |          1        1        1
root@gpbgp:/go# exit
exit
```

```
# docker exec -it ff5c323d2118 gobgp global
AS:        65537
Router-ID: 192.0.2.3
Listening Port: 179, Addresses: 0.0.0.0, ::
MPLS Label Range: 16000..1048575
```

Since ExaBGP's config file contains a static route which is tagged with a large BGP community we can verify how GoBGP and BIRD see it:

```
# docker exec ff5c323d2118 gobgp neighbor 192.0.2.2 adj-in
    Network             Next Hop             AS_PATH              Age        Attrs
    203.0.113.1/32      192.0.2.2            65536                00:14:49   [{Origin: i} {LargeCommunity: [ 65536:1:2]}]
```

```
# docker exec 153b6165385f birdcl show route all
BIRD 1.6.1 ready.
203.0.113.1/32     unreachable [ExaBGP 14:29:25 from 192.0.2.2] * (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65536
        BGP.next_hop: 192.0.2.2
        BGP.local_pref: 100
        BGP.large_community: (65536,1,2)
                   unreachable [GoBGP 14:29:28 from 192.0.2.3] (100/-) [AS65536i]
        Type: BGP unicast univ
        BGP.origin: IGP
        BGP.as_path: 65537 65536
        BGP.next_hop: 192.0.2.3
        BGP.local_pref: 100
        BGP.large_community: (65536,1,2)
```

**LargeCommunity: [ 65536:1:2]**, **BGP.large_community: (65536,1,2)** - here it is!

Let's have [GoBGP announce a new tagged prefix](https://github.com/osrg/gobgp/blob/master/docs/sources/cli-command-syntax.md#more-examples) and see how ExaBGP receive it:

```
# docker exec ff5c323d2118 gobgp global rib add -a ipv4 203.0.113.2/32 large-community 65537:3:4
# cat exabgp/log
...
Thu, 14 Sep 2016 18:15:18 5      routes        peer 192.0.2.3 ASN 65537   << UPDATE (1) (   4)  attributes origin incomplete as-path [ 65537 ] large-community 65537:3:4
```

Similarly we can send a ping from one of the running containers to pmacct host to see how it handles large communities in its output:

```
# docker exec -it ff5c323d2118 bash
root@gobgp:/go# ping 192.0.2.5
PING 192.0.2.5 (192.0.2.5): 56 data bytes
64 bytes from 192.0.2.5: icmp_seq=0 ttl=64 time=0.185 ms
64 bytes from 192.0.2.5: icmp_seq=1 ttl=64 time=0.131 ms
64 bytes from 192.0.2.5: icmp_seq=2 ttl=64 time=0.125 ms
```

```
# cat pmacct/plugin1.out
COMMS,SRC_COMMS,SRC_IP,DST_IP,SRC_PORT,DST_PORT,PROTOCOL,PACKETS,BYTES
,2:2,192.0.2.3,192.0.2.5,0,0,icmp,3,252
2:2,,192.0.2.5,192.0.2.3,0,0,icmp,3,252
```

Since all the images `EXPOSE` port 179, the `-p 179:179` Docker `run` option can be used to publish the BGP daemon outside the local host, in order to test interoperability with other software/hardware:

```
# docker run --net large-bgp-communities-playground --ip 192.0.2.3 -p 179:179 --hostname=gpbgp -d -v `pwd`/gobgp:/etc/gobgp:rw pierky/gobgp
# # now establish a BGP session with <your_host_ip>:179
```

Enjoy Large BGP Communities and have fun! ;)

# Author

Pier Carlo Chiodi - https://pierky.com

Blog: https://blog.pierky.com Twitter: [@pierky](https://twitter.com/pierky)
