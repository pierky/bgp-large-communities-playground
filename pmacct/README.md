# Overview

This image is based on the master branch of [pmacct](https://github.com/pmacct/pmacct). It mounts host's directory `./pmacct` in the container's `/etc/pmacct` directory; here the `pmacctd.conf` is used to give `pmacctd` the startup configuration and `log` and `bgp.log` are used by the daemon to write its logs.

It has been created to run a playground to test BGP Large Communities: https://github.com/pierky/bgp-large-communities-playground

# Disclaimer

This image has been created with the only purpose of being used in a "playground", for labs and interoperability tests. It does not implement any security best practice. Use it at your own risk.

# How to use this image

Put the [pmacct startup config](http://wiki.pmacct.net/OfficialConfigKeys) into `pmacct/pmacctd.conf`...

```
# mkdir pmacct
# vim pmacct/pmacctd.conf
```

... then run the image in detached mode (`-d`) with the local `pmacct` directory mounted in `/etc/pmacct`:

```
# docker network create --subnet=192.0.2.0/24 bgp-large-communities-playground
# docker run --net bgp-large-communities-playground --ip 192.0.2.5 -d -v `pwd`/pmacct:/etc/pmacct:rw pierky/pmacct
```

You can verify that `log` gets populated: `cat pmacct/log`.

Run `docker ps` to get the running container's ID, then use it to possibly attach a terminal to the running instance:

```
# docker ps
CONTAINER ID        IMAGE               COMMAND                ...
bd30b1671eff        pierky/pmacct       "/bin/sh -c 'pmacctd " ...
# docker exec -it bd30b1671eff bash
root@bd30b1671eff:~#
```

# Author

Pier Carlo Chiodi - https://pierky.com

Blog: https://blog.pierky.com Twitter: [@pierky](https://twitter.com/pierky)

