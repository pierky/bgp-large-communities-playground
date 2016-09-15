FROM python:2

MAINTAINER Pier Carlo Chiodi <pierky@pierky.com>

EXPOSE 179

RUN apt-get update && apt-get install -y \
        unzip

WORKDIR /root
RUN curl -O -L https://github.com/Exa-Networks/exabgp/archive/master.zip
RUN unzip master.zip

# This directory must be mounted as a local volume with '-v `pwd`/exabgp:/etc/exabgp:rw' docker's command line option.
# The host's file at `pwd`/exabgp/exabgp.conf is used as the configuration file for ExaBGP.
RUN mkdir /etc/exabgp

RUN mkdir -p /exabgp-master/etc/exabgp
ENV PATH /root/exabgp-master/bin:/root/exabgp-master/sbin:$PATH

RUN exabgp --fi > /root/exabgp-master/etc/exabgp/exabgp.env

CMD env exabgp.log.destination=/etc/exabgp/log exabgp.log.routes=true exabgp.daemon.user=root exabgp /etc/exabgp/exabgp.conf
