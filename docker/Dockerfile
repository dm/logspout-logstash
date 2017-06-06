FROM gliderlabs/logspout:master
ENV SYSLOG_FORMAT rfc3164

ADD ./entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
