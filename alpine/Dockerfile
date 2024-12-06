FROM snowdreamtech/golang:1.22.9-alpine3.20 AS builder

ENV SFTPGO_VERSION=2.6.4 \
    GOFLAGS="-mod=readonly"

# This ARG allows to disable some optional features and it might be useful if you build the image yourself.
# For example you can disable S3 and GCS support like this:
# --build-arg FEATURES=nos3,nogcs
# FEATURES=

RUN apk -U upgrade --no-cache && apk add --update --no-cache bash ca-certificates curl git gcc g++

RUN mkdir -p /workspace
WORKDIR /workspace

RUN set -eux \
    && wget -c https://github.com/drakkan/sftpgo/archive/refs/tags/v2.6.4.tar.gz -O sftpgo-${SFTPGO_VERSION}.tar.gz \
    && tar zxvf sftpgo-${SFTPGO_VERSION}.tar.gz \
    && cd sftpgo-${SFTPGO_VERSION} \
    && go build $(if [ -n "${FEATURES}" ]; then echo "-tags ${FEATURES}"; fi) -trimpath -ldflags "-s -w -X github.com/drakkan/sftpgo/v2/internal/version.date=`date -u +%FT%TZ`" -v -o sftpgo 



FROM snowdreamtech/alpine:3.20.3

# OCI annotations to image
LABEL org.opencontainers.image.authors="Snowdream Tech" \
    org.opencontainers.image.title="SFTPGo Image Based On Alpine" \
    org.opencontainers.image.description="Docker Images for SFTPGo on Alpine. (i386, amd64, arm32v6, arm32v7, arm64, ppc64le,riscv64, s390x)" \
    org.opencontainers.image.documentation="https://hub.docker.com/r/snowdreamtech/sftpgo" \
    org.opencontainers.image.base.name="snowdreamtech/sftpgo:alpine" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.source="https://github.com/snowdreamtech/sftpgo" \
    org.opencontainers.image.vendor="Snowdream Tech" \
    org.opencontainers.image.version="3.20.3" \
    org.opencontainers.image.url="https://github.com/snowdreamtech/sftpgo"

# keep the docker container running
ENV KEEPALIVE=1 \
    SFTPGO_VERSION=2.6.4

# Set to "true" to install jq and the optional git and rsync dependencies
ARG INSTALL_OPTIONAL_PACKAGES=false

RUN apk -U upgrade --no-cache && apk add --update --no-cache ca-certificates tzdata mailcap

RUN if [ "${INSTALL_OPTIONAL_PACKAGES}" = "true" ]; then apk add --update --no-cache jq git rsync; fi

RUN mkdir -p /etc/sftpgo /var/lib/sftpgo /usr/share/sftpgo /srv/sftpgo/data /srv/sftpgo/backups

RUN addgroup -g 1000 -S sftpgo && \
    adduser -u 1000 -h /var/lib/sftpgo -s /sbin/nologin -G sftpgo -S -D -H -g "SFTPGo user" sftpgo

COPY --from=builder /workspace/sftpgo-${SFTPGO_VERSION}/sftpgo.json /etc/sftpgo/sftpgo.json
COPY --from=builder /workspace/sftpgo-${SFTPGO_VERSION}/templates /usr/share/sftpgo/templates
COPY --from=builder /workspace/sftpgo-${SFTPGO_VERSION}/static /usr/share/sftpgo/static
COPY --from=builder /workspace/sftpgo-${SFTPGO_VERSION}/openapi /usr/share/sftpgo/openapi
COPY --from=builder /workspace/sftpgo-${SFTPGO_VERSION}/sftpgo /usr/local/bin/

# Log to the stdout so the logs will be available using docker logs
ENV SFTPGO_LOG_FILE_PATH=""

# Modify the default configuration file
RUN sed -i 's|"users_base_dir": "",|"users_base_dir": "/srv/sftpgo/data",|' /etc/sftpgo/sftpgo.json && \
    sed -i 's|"backups"|"/srv/sftpgo/backups"|' /etc/sftpgo/sftpgo.json

RUN chown -R sftpgo:sftpgo /etc/sftpgo /srv/sftpgo && chown sftpgo:sftpgo /var/lib/sftpgo && chmod 700 /srv/sftpgo/backups

WORKDIR /var/lib/sftpgo
USER 1000:1000

COPY docker-entrypoint.sh /usr/local/bin/

ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 8080 2022 

CMD ["sftpgo", "serve"]