FROM debian:bookworm-slim

ENV DEBIAN_FRONTEND noninteractive

RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends tini gosu procps net-tools \
        nagios-nrpe-server nagios-nrpe-plugin monitoring-plugins-basic; \
    rm -rf /var/lib/apt/lists/*; \
    sed -i -E 's|^#(log_file=/var/log/nrpe.log)|\1|' /etc/nagios/nrpe.cfg;

COPY --chmod=755 check_mem.sh /usr/lib/nagios/plugins/
COPY --chmod=755 startup.sh /

EXPOSE 5666

CMD /startup.sh

ENTRYPOINT ["/usr/bin/tini","--"]