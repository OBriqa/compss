# =============================================================================
# Image for building COMPSs
# =============================================================================
FROM ubuntu:22.04 AS builder
LABEL maintainer="COMPSs Support <support-compss@bsc.es>" \
      vendor="Barcelona Supercomputing Center (BSC)" \
      url="http://compss.bsc.es"

ENV GRADLE_HOME=/opt/gradle 
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/ 
ENV EXTRAE_MPI_HEADERS=/usr/include/x86_64-linux-gnu/mpi

ENV PATH=$PATH:/opt/COMPSs/Runtime/scripts/user:/opt/COMPSs/Bindings/c/bin:/opt/COMPSs/Runtime/scripts/utils:/opt/gradle/bin
ENV CLASSPATH=$CLASSPATH:/opt/COMPSs/Runtime/compss-engine.jar
ENV LD_LIBRARY_PATH=/opt/COMPSs/Bindings/bindings-common/lib:$JAVA_HOME/jre/lib/amd64/server 
ENV COMPSS_HOME=/opt/COMPSs/

COPY . /framework

RUN apt-get update && \
    apt-get install -y --no-install-recommends wget maven git python3 pip

RUN cd /framework && \
    ./submodules_get.sh && \
    /framework/builders/buildlocal -N --skip-tests /opt/COMPSs 

# =============================================================================
# Image for running COMPSs agent
# =============================================================================
FROM ubuntu:22.04 AS base
LABEL maintainer="COMPSs Support <support-compss@bsc.es>" \
      vendor="Barcelona Supercomputing Center (BSC)" \
      url="http://compss.bsc.es"

RUN apt-get update && \
# Install Packages
    apt-get install -y --no-install-recommends openjdk-11-jdk \
# Dependencies
    curl build-essential openmpi-bin openmpi-doc libopenmpi-dev \
# python-pip
    python3 python3-setuptools python3-dev python3-pip && \
# Add environment variables
    echo "JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/" >> /etc/environment && \
    echo "MPI_HOME=/usr/lib/openmpi" >> /etc/environment && \
    echo "LD_LIBRARY_PATH=/usr/lib/openmpi/lib" >> /etc/environment && \
    echo "EXTRAE_MPI_HEADERS=/usr/include/x86_64-linux-gnu/mpi" >> /etc/environment && \
    mkdir /run/sshd && \
    apt-get autoclean && \
    rm -rf /var/lib/apt/lists/*

# SET UP ENVIRONMENT
ENV AGENT_HOST=localhost
ENV REST_AGENT_PORT=0
ENV COMM_AGENT_PORT=0
ENV DEBUG=debug 
ENV APP_PATH=/app 
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64/

WORKDIR /tmp

# COPY COMPSs INSTALLATION
COPY --from=builder /opt/COMPSs /opt/COMPSs

CMD /opt/COMPSs/Runtime/scripts/user/compss_agent_start --hostname=${AGENT_HOST} --comm_port=${COMM_AGENT_PORT} --rest_port=${REST_AGENT_PORT} -d ${DEBUG} --classpath=${APP_PATH} --pythonpath=${APP_PATH} --log_dir=/tmp

