# Large BGP Communities playground

## What Large BGP Communities are

A good site already explains it very well: http://largebgpcommunities.net/

## This playground

Luckly many vendors and networking software authors are approaching this solution and [started implementing](http://largebgpcommunities.net/implementations/) [the draft](https://tools.ietf.org/html/draft-heitz-idr-large-community-04). This repository (presumptuously) wants to offer some hints to quickly have a *large-bgp-communities*-aware lab up & running on the basis of the latest code available.

Currently it supports 2 products, [ExaBGP](https://github.com/Exa-Networks/exabgp) and [GoBGP](https://github.com/osrg/gobgp). [Docker images](https://hub.docker.com/u/pierky/) have been built in order to have them running on the latest code fetched from the `master` branch of both of them.

### Disclaimer

These images have been created with the only purpose of being used in a "playground", for labs and interoperability tests. They do not implement any security best practice. Use them at your own risk.

### How to run it

```
# git clone https://github.com/pierky/large-bgp-communities-playground.git
# cd large-bgp-communities-playground/
# docker network create --subnet=192.0.2.0/24 large-bgp-communities-playground
# docker run --net large-bgp-communities-playground --ip 192.0.2.2 --hostname=exabgp -d -v `pwd`/exabgp:/etc/exabgp:rw pierky/exabgp
# docker run --net large-bgp-communities-playground --ip 192.0.2.3 --hostname=gpbgp -d -v `pwd`/gobgp:/etc/gobgp:rw pierky/gobgp
```

This is enough to create a virtual network, have ExaBGP running on 192.0.2.2 and GoBGP on 192.0.2.3. The startup config files (`exabgp/exabgp.conf` and `gobgp/gobgp.conf`) allow the two instances to establish a BGP session:

```
# cat exabgp/log
...
Thu, 14 Sep 2016 17:54:57 5      network       Connected to peer neighbor 192.0.2.3 local-ip 192.0.2.2 local-as 65536 peer-as 65537 router-id 192.0.2.2 family
```

```
# cat gobgp/log
...
time="2016-09-14T17:54:57Z" level=info msg="Peer Up" Key=192.0.2.2 State="BGP_FSM_OPENCONFIRM" Topic=Peer
```

Commands can be run on the instances interactively, by attaching a new terminal to the Docker container, or directly from the host:

```
# # take note of the container ID of each instance
# docker ps
CONTAINER ID        IMAGE               ...
ff5c323d2118        pierky/gobgp        ...
2c46decfb88a        pierky/exabgp       ...
```

```
# docker exec -it ff5c323d2118 bash
root@gpbgp:/go# gobgp neighbor
Peer         AS  Up/Down State       |#Advertised Received Accepted
192.0.2.2 65536 00:11:03 Establ      |          0        1        1
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

Since ExaBGP's config file contains a static route which is tagged with a large BGP community we can verify how GoBGP sees it:

```
# docker exec ff5c323d2118 gobgp neighbor 192.0.2.2 adj-in
    Network             Next Hop             AS_PATH              Age        Attrs
    203.0.113.1/32      192.0.2.2            65536                00:14:49   [{Origin: i} {LargeCommunity: [ 65536:1:2]}]
```

**LargeCommunity: [ 65536:1:2]**, here it is!

Let's have [GoBGP announce a new tagged prefix](https://github.com/osrg/gobgp/blob/master/docs/sources/cli-command-syntax.md#more-examples) and see how ExaBGP receive it:

```
# docker exec ff5c323d2118 gobgp global rib add -a ipv4 203.0.113.2/32 large-community 65537:3:4
# cat exabgp/log
...
Thu, 14 Sep 2016 18:15:18 5      routes        peer 192.0.2.3 ASN 65537   << UPDATE (1) (   4)  attributes origin incomplete as-path [ 65537 ] large-community 65537:3:4
```

Since both images `EXPOSE` port 179, the `-p 179:179` Docker `run` option can be used to publish the [Exa|Go]BGP daemon outside the local host, in order to test interoperability with other software/hardware:

```
# docker run --net large-bgp-communities-playground --ip 192.0.2.3 -p 179:179 --hostname=gpbgp -d -v `pwd`/gobgp:/etc/gobgp:rw pierky/gobgp
# # now establish a BGP session with <your_host_ip>:179
```

Enjoy Large BGP Communities and have fun! ;)

# Author

Pier Carlo Chiodi - https://pierky.com

Blog: https://blog.pierky.com Twitter: [@pierky](https://twitter.com/pierky)
