FROM gcr.io/google_containers/ubuntu-slim:0.6
ENV DEBIAN_FRONTEND noninteractive

# Fluent Bit version
ENV FLB_MAJOR 0
ENV FLB_MINOR 11
ENV FLB_PATCH 0
ENV FLB_VERSION 0.11.0


MAINTAINER Eduardo Silva <eduardo@treasure-data.com>
LABEL Description="Fluent Bit docker image" Vendor="Fluent Organization" Version="1.1"
USER root

# Install build tools
RUN apt-get -qq update && \
    apt-get install -y -qq curl ca-certificates build-essential cmake make bash sudo wget unzip && \
    apt-get install -y -qq --reinstall lsb-base lsb-release && \
    wget -O "/tmp/fluent-bit-$FLB_VERSION-dev.zip" "http://github.com/fluent/fluent-bit/archive/master.zip" && \
    cd /tmp && \
    unzip "fluent-bit-$FLB_VERSION-dev.zip" && \
    cd "fluent-bit-master/build/" && \
    cmake -DFLB_DEBUG=On -DFLB_TRACE=On \
          -DCMAKE_INSTALL_PREFIX=/fluent-bit/ -DFLB_JEMALLOC=On ../&& \
    make && make install && \
    rm -rf /tmp/* /fluent-bit/include /fluent-bit/lib* && \
    apt-get remove -y -qq cmake make wget unzip

COPY fluent-bit.conf /fluent-bit/etc/
COPY parsers.conf /fluent-bit/etc/

CMD ["/fluent-bit/bin/fluent-bit", "-c", "/fluent-bit/etc/fluent-bit.conf"]
