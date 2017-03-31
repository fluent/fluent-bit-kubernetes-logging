FROM gcr.io/google-containers/debian-base-amd64:0.1

ARG FLB_PREFIX
ARG FLB_VERSION

ENV FLB_TARBALL http://github.com/fluent/fluent-bit/archive/$FLB_PREFIX$FLB_VERSION.zip

RUN apt-get -qq update && \
    apt-get install -y -qq curl ca-certificates build-essential sudo wget nano && \
    apt-get install -y -qq --reinstall lsb-base lsb-release && \
    wget -qO - http://packages.fluentbit.io/fluentbit.key | sudo apt-key add - && \
    echo "deb http://packages.fluentbit.io/kubernetes gcr main" >> /etc/apt/sources.list && \
    apt-get -qq update && apt-get install td-agent-bit


RUN  mkdir -p /etc/td-agent-bit/
COPY fluent-bit.conf /etc/td-agent-bit/
COPY parsers.conf /etc/td-agent-bit/

CMD ["/opt/td-agent-bit/bin/td-agent-bit", "-c", "/etc/td-agent-bit/fluent-bit.conf"]
