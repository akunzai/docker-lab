FROM docker.io/library/debian:buster-slim

ENV DEBIAN_FRONTEND noninteractive

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends squid procps; \
    rm -rf /var/lib/apt/lists/*; \
    sed -i -E 's/^#(http_access allow localnet)/\1/' /etc/squid/conf.d/debian.conf; \
    # ensure log directory exists and fixes permission
    mkdir -p /var/log/squid; \
    chmod -R 755 /var/log/squid; \
    chown -R proxy:proxy /var/log/squid; \
    # ensure cache directory exists and fixes permission
    mkdir -p /var/spool/squid; \
    chown -R proxy:proxy /var/spool/squid;

COPY --chmod=755 ./entrypoint.sh /

VOLUME /var/spool/squid

EXPOSE 3128
ENTRYPOINT ["/entrypoint.sh"]
CMD ["/usr/sbin/squid","--foreground","-YC"]