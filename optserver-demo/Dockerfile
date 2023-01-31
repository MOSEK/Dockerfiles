# Simple demo installation of MOSEK OptServer.
FROM ubuntu:22.04

LABEL name="MOSEK OptServer Demo" maintainer="support@mosek.com"

# Set up the system
ARG DEBIAN_FRONTEND=noninteractive
# Change if you wish
ENV TZ=Europe/London
RUN apt-get update \
    && apt-get install -y postgresql curl zip bzip2 \
    && useradd -m optserver \
    && usermod -a -G postgres optserver

USER optserver
WORKDIR /home/optserver

# Set up environment variables
ENV PG=/usr/lib/postgresql/14/bin/ \
    PGDATA=/home/optserver/pgdata \
    MOSEKHOME=/home/optserver/mosek/latest \
    MOSEKBIN=/home/optserver/mosek/latest/tools/platform/linux64x86/bin \
    MOSEKOPT=/home/optserver/mosek/latest/opt-server \
    MOSEKOPTBIN=/home/optserver/mosek/latest/opt-server/bin \
    MOSEKKEYS=/home/optserver/mosek/latest/opt-server/etc/Mosek/ssl \
    MOSEKCONF=/home/optserver/mosek/latest/opt-server/etc/Mosek/server.conf \
    ADMINPASSWD=admin    

# Download and unpack latest MOSEK
RUN curl https://download.mosek.com/stable/latest/mosektoolslinux64x86.tar.bz2 --output mosektoolslinux64x86.tar.bz2; \
    mkdir -p mosek/latest; \
    tar xf mosektoolslinux64x86.tar.bz2 --strip-components=2 -C mosek/latest; \
    # We generate SSL keys but the demo version will not use them for anything
    mkdir -p $MOSEKKEYS; \
    openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -subj "/C=DK/ST=None/L=None/O=Mosek/CN=Mosek" -keyout $MOSEKKEYS/key.pem -out $MOSEKKEYS/cert.pem; \
    # Configure the OptServer with some default settings
    $MOSEKOPTBIN/install_MosekServer --inplace \
                                     --port 30080 \
                                     --mosekbindir $MOSEKBIN \
                                     --database-resource "host=/var/run/postgresql user=optserver dbname=optserver sslmode=disable" \
                                     --enable-anonymous \
                                     --ssl $MOSEKKEYS \
                                     --mode gui; \
    # Initialize OptServer database and admin password
    $PG/initdb; \
    $PG/pg_ctl start; \
    $PG/createdb optserver; \
    echo $ADMINPASSWD | $MOSEKOPTBIN/MosekServer --create-database --reset-admin --no-startup; \
    $PG/pg_ctl stop; \
    # Create some OptServer startup/stop scripts
    echo "#!/bin/bash\n$MOSEKOPTBIN/MosekServer --logfile - -unsafe -disable-tls" > start-server; \
    echo "#!/bin/bash\ncat $MOSEKOPT/var/Mosek/server.pid | xargs kill" > stop-server; \
    echo "#!/bin/bash\n./stop-server && ./start-server" > restart-server; \
    echo "#!/bin/bash\n$PG/pg_ctl start && ./start-server" > start-all; \
    chmod +x start-server stop-server restart-server start-all; 

# Copy license file
COPY mosek.lic mosek/

EXPOSE 30080
ENTRYPOINT [ "./start-all" ]
