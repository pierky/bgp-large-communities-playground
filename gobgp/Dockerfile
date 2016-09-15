FROM golang

MAINTAINER Pier Carlo Chiodi <pierky@pierky.com>

EXPOSE 179

RUN go get -v github.com/osrg/gobgp/gobgpd && go get -v github.com/osrg/gobgp/gobgp

# This directory must be mounted as a local volume with '-v `pwd`/gobgp:/etc/gobgp:rw' docker's command line option.
# The host's file at `pwd`/gobgp/gobgp.conf is used as the configuration file for GoBGP.
RUN mkdir /etc/gpbgp

WORKDIR /go

CMD gobgpd -f /etc/gobgp/gobgp.conf -p > /etc/gobgp/log
