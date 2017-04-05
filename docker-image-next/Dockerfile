FROM fluent/fluent-bit:0.11-next
MAINTAINER Eduardo Silva <eduardo@treasure-data.com>
USER root

COPY fluent-bit.conf /fluent-bit/etc/
COPY parsers.conf /fluent-bit/etc/

CMD ["/fluent-bit/bin/fluent-bit", "-c", "/fluent-bit/etc/fluent-bit.conf"]
