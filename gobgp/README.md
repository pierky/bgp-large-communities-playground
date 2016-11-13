# Overview

This image is based on the master branch of [GoBGP](https://github.com/osrg/gobgp). It `EXPOSE`s port 179 and mounts host's directory `./gobgp` in the container's `/etc/gpbgp` directory; here the `gobgp.conf` is used to give `gobgpd` the startup configuration and `log` is used by the daemon to write its log.

It has been created to run a playground to test BGP Large Communities: https://github.com/pierky/bgp-large-communities-playground

# Disclaimer

This image has been created with the only purpose of being used in a "playground", for labs and interoperability tests. It does not implement any security best practice. Use it at your own risk.

# How to use this image

Put the [GoBGP startup config](https://github.com/osrg/gobgp/blob/master/docs/sources/getting-started.md#configuration) into `gobgp/gobgp.conf`...

```
# mkdir gobgp
# vim gobgp/gobgp.conf
```

... then run the image in detached mode (`-d`) with the local `gobgp` directory mounted in `/etc/gpbgp`:

```
# docker run -d -v `pwd`/gobgp:/etc/gobgp:rw pierky/gobgp
```

You can verify that `log` gets populated: `cat gobgp/log`.

Run `docker ps` to get the running container's ID (`68dd61ea9f92` in the example below), then use it to attach a terminal to the running instance or to directly execute GoBGP commands on it:

```
# docker exec -it 68dd61ea9f92 bash
root@68dd61ea9f92:/go# gobgp neighbor
Peer         AS Up/Down State       |#Advertised Received Accepted
192.0.2.2 65536   never Active      |          0        0        0
root@68dd61ea9f92:/go# exit
exit
```

```
# docker exec 68dd61ea9f92 gobgp neighbor 192.0.2.2 adj-in
Neighbor 192.0.2.2's BGP session is not established
```

# Author

Pier Carlo Chiodi - https://pierky.com

Blog: https://blog.pierky.com Twitter: [@pierky](https://twitter.com/pierky)

