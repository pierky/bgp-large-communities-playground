# Overview

This image is based on the master branch of [ExaBGP](https://github.com/Exa-Networks/exabgp). It `EXPOSE`s port 179 and mounts host's directory `./exabgp` in the container's `/etc/exabgp` directory; here the `exabgp.conf` is used to give `exabgp` the startup configuration and `log` is used by the daemon to write its log.

It has been created to run a playground to test BGP Large Communities: https://github.com/pierky/bgp-large-communities-playground

# Disclaimer

This image has been created with the only purpose of being used in a "playground", for labs and interoperability tests. It does not implement any security best practice. Use it at your own risk.

# How to use this image

Put the [ExaBGP startup config](https://github.com/Exa-Networks/exabgp/wiki/Controlling-ExaBGP-:-_-README-first#configuring-exabgp-to-use-your-program) into `exabgp/exabgp.conf`...

```
# mkdir exabgp
# vim exabgp/exabgp.conf
```

... then run the image in detached mode (`-d`) with the local `exabgp` directory mounted in `/etc/exabgp`:

```
# docker run -d -v `pwd`/exabgp:/etc/exabgp:rw pierky/exabgp
```

You can verify that `log` gets populated: `cat exabgp/log`.

Run `docker ps` to get the running container's ID (`cd61079342d2` in the example below), then use it to possibly attach a terminal to the running instance.

# Author

Pier Carlo Chiodi - https://pierky.com

Blog: https://blog.pierky.com Twitter: [@pierky](https://twitter.com/pierky)

