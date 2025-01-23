# Simple demo installation of MOSEK OptServer.
FROM ubuntu:24.04

LABEL name="MOSEK OptServer Demo" maintainer="support@mosek.com"

# Set up the system
ARG DEBIAN_FRONTEND=noninteractive
# Change if you wish
RUN apt-get update
RUN apt-get -q -y install curl bzip2
RUN useradd optserver --create-home

USER optserver
WORKDIR /home/optserver

# Set up environment variables
ENV ADMINPASSWD=admin    

# Download and unpack latest MOSEK
RUN curl https://download.mosek.com/stable/latest/mosektoolslinux64x86.tar.bz2 --output mosektoolslinux64x86.tar.bz2
RUN mkdir -p mosek/latest
RUN tar xf mosektoolslinux64x86.tar.bz2 --strip-components=2 -C mosek/latest

# Download and unpack latest OptServer
RUN curl https://download.mosek.com/optserver/latest --output latest
RUN curl https://download.mosek.com/optserver/optserver-linux-amd64-$(cat latest).tar.bz2 --output opt-server-3.tar.bz2
RUN mkdir -p opt-server
RUN tar xf opt-server-3.tar.bz2 --strip-components=1 -C opt-server

# Copy server config file into default location
COPY optserver.conf opt-server/etc/mosek/optserver.conf

# Initialize
RUN ./opt-server/bin/optserver --log-file - --create-db initialize --unsafe-admin-password $ADMINPASSWD

# Copy license file (you have to provide it yourself)
COPY mosek.lic mosek/

EXPOSE 30080
COPY start.sh .
ENTRYPOINT [ "./start.sh" ]
