# Simple demo installation of MOSEK OptServer.
FROM ubuntu

LABEL name="MOSEK OptServer Demo" maintainer="support@mosek.com"

# Set up the system
RUN apt-get update \
    && apt-get install -y postgresql curl \
    && useradd -m optserver \
    && usermod -a -G postgres optserver

USER optserver
WORKDIR /home/optserver

# Set up environment variables
ENV PG=/usr/lib/postgresql/10/bin/ \
    PGDATA=/home/optserver/pgdata \
    MOSEKHOME=/home/optserver/mosek/latest \
    MOSEKBIN=/home/optserver/mosek/latest/tools/platform/linux64x86/bin \
    MOSEKOPT=/home/optserver/mosek/latest/opt-server \
    MOSEKOPTBIN=/home/optserver/mosek/latest/opt-server/bin

# Download and unpack latest MOSEK
RUN curl https://download.mosek.com/stable/latest/mosektoolslinux64x86.tar.bz2 --output mosektoolslinux64x86.tar.bz2; \
    mkdir -p mosek/latest; \
    tar xf mosektoolslinux64x86.tar.bz2 --strip-components=2 -C mosek/latest; \
    # Configure the OptServer with some default settings
    $MOSEKOPTBIN/install_MosekServer --inplace \
                                     --port 30080 \
                                     --mosekbindir $MOSEKBIN \
                                     --database-resource "host=/var/run/postgresql user=optserver dbname=optserver sslmode=disable" \
                                     --enable-anonymous \
                                     --mode gui; \
    # Initialize OptServer database and admin password
    $PG/initdb; \
    $PG/pg_ctl start; \
    $PG/createdb optserver; \
    echo admin | $MOSEKOPTBIN/MosekServer --create-database --reset-admin --no-startup; \
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
