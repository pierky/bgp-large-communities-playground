FROM debian:stable

MAINTAINER Pier Carlo Chiodi <pierky@pierky.com>

EXPOSE 179

RUN apt-get update && apt-get install -y \
	bison \
	build-essential \
	curl \
	flex \
	libreadline-dev \
	libncurses5-dev \
	m4 \
	unzip \
	libpcap-dev \
	libtool \
	automake1.1 \
	autoconf2.13 \
	pkg-config \
	libjansson-dev

WORKDIR /root
RUN curl -O -L https://github.com/pmacct/pmacct/archive/master.zip
RUN unzip master.zip

# This directory must be mounted as a local volume with '-v `pwd`/pmacct:/etc/pmacct:rw' docker's command line option.
# The host's files at `pwd`/pmacct/ are used as the configuration file for pmacct. Here pmacct writes its output too.
RUN mkdir /etc/pmacct

RUN cd pmacct-master && \
	./autogen.sh && \
	./configure --enable-jansson && \
	make && \
	make install

CMD pmacctd -f /etc/pmacct/pmacctd.conf
