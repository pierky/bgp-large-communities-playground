FROM debian:stable

MAINTAINER Pier Carlo Chiodi <pierky@pierky.com>

EXPOSE 179

RUN apt-get update && apt-get install -y \
        autoconf \
	bison \
	build-essential \
	curl \
	flex \
	libreadline-dev \
	libncurses5-dev \
	m4 \
	unzip \
	gawk \
	libtool \
	telnet

WORKDIR /root
RUN curl -O -L http://download.savannah.gnu.org/releases/quagga/quagga-1.1.0.tar.gz
RUN curl -o large_bgp_communities.patch -L 'https://bugzilla.quagga.net/attachment.cgi?id=504'
RUN curl -O -L https://ftp.gnu.org/gnu/automake/automake-1.15.tar.gz
RUN tar -zxvf quagga-1.1.0.tar.gz
RUN tar -zxvf automake-1.15.tar.gz

RUN useradd -M quagga

# This directory must be mounted as a local volume with '-v `pwd`/quagga:/etc/quagga:rw' docker's command line option.
# The host's file at `pwd`/quagga/quagga.conf is used as the configuration file for Quagga BGPD.
RUN mkdir /etc/quagga && \
	mkdir /var/run/quagga && \
	chown quagga /etc/quagga && \
	chown quagga /var/run/quagga

RUN cd automake-1.15 && \
	./configure && \
	make && \
	make install

RUN cd quagga-1.1.0 && \
	patch -p1 < ../large_bgp_communities.patch && \
	./configure --disable-doc && \
	make && \
	make install

RUN ldconfig

CMD /usr/local/sbin/bgpd --config_file=/etc/quagga/quagga.conf --pid_file=/var/run/quagga/quagga.pid
