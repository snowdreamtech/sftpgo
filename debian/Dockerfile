FROM snowdreamtech/golang:1.23.3-bookworm AS builder

# Switch to the user
USER root

# Set the workdir
WORKDIR /root

ENV SFTPGO_VERSION=2.6.6 \
    GOFLAGS=""

# This ARG allows to disable some optional features and it might be useful if you build the image yourself.
# For example you can disable S3 and GCS support like this:
# --build-arg FEATURES=nos3,nogcs
# FEATURES=

RUN apt-get update && apt-get -y upgrade && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get -y upgrade && apt-get install --no-install-recommends -y build-essential && rm -rf /var/lib/apt/lists/*

COPY ./locales /root/locales

RUN set -eux \
    && wget -c https://github.com/drakkan/sftpgo/archive/refs/tags/v${SFTPGO_VERSION}.tar.gz -O sftpgo-${SFTPGO_VERSION}.tar.gz \
    && tar zxvf sftpgo-${SFTPGO_VERSION}.tar.gz \
    && cd sftpgo-${SFTPGO_VERSION} \
    && cp -rfv /root/locales/ ./static/ \
    # && sed -i "s/debug: *false,/debug: true,/g" ./templates/common/base.html \
    && sed -i "s/supportedLngs:.*,/supportedLngs: Object.keys(lngs),/g" ./templates/common/base.html \
    && sed -i "s/load: *'languageOnly',/load: 'all',/g" ./templates/common/base.html \
    && sed -i '/const lngs = {/,/};/c\
    const lngs = {\
        '\''zh-CN'\'': { nativeName: '\''简体中文'\'' },\
        en: { nativeName: '\''English'\'' },\
        it: { nativeName: '\''Italiano'\'' },\
        fr: { nativeName: '\''Français'\'' },\
        ru: { nativeName: '\''Русский'\'' },\
        uk: { nativeName: '\''Українська'\'' }\
    };\
' ./templates/common/base.html \
    && sed -i "s/^go *\([0-9]\+\.[0-9]\+\)\.[0-9]\+/go \1/" go.mod \
    && go mod tidy \
    && go build $(if [ -n "${FEATURES}" ]; then echo "-tags ${FEATURES}"; fi) -trimpath -ldflags "-s -w -X github.com/drakkan/sftpgo/v2/internal/version.date=`date -u +%FT%TZ`" -v -o sftpgo 



FROM snowdreamtech/debian:12.10.0

# OCI annotations to image
LABEL org.opencontainers.image.authors="Snowdream Tech" \
    org.opencontainers.image.title="SFTPGo Image Based On Debian" \
    org.opencontainers.image.description="Docker Images for SFTPGo on Debian. (i386,amd64,arm32v5,arm32v7,arm64,mips64le,ppc64le,s390x)" \
    org.opencontainers.image.documentation="https://hub.docker.com/r/snowdreamtech/sftpgo" \
    org.opencontainers.image.base.name="snowdreamtech/sftpgo:debian" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.source="https://github.com/snowdreamtech/sftpgo" \
    org.opencontainers.image.vendor="Snowdream Tech" \
    org.opencontainers.image.version="2.6.6" \
    org.opencontainers.image.url="https://github.com/snowdreamtech/sftpgo"

# keep the docker container running
ENV KEEPALIVE=1 \
SFTPGO_VERSION=2.6.6

ARG GID=1000 \
    UID=1000  \
    USER=sftpgo \
    WORKDIR=/var/lib/sftpgo
    
# Set to "true" to install jq and the optional git and rsync dependencies
ARG INSTALL_OPTIONAL_PACKAGES=false

RUN apt-get update && apt-get -y upgrade && apt-get install --no-install-recommends -y ca-certificates media-types && rm -rf /var/lib/apt/lists/*

RUN if [ "${INSTALL_OPTIONAL_PACKAGES}" = "true" ]; then apt-get update && apt-get install --no-install-recommends -y jq git rsync && rm -rf /var/lib/apt/lists/*; fi

RUN mkdir -p /etc/sftpgo /var/lib/sftpgo /usr/share/sftpgo /srv/sftpgo/data /srv/sftpgo/backups
    
# Create a user with UID and GID
RUN set -eux \
    &&if [ "${USER}" != "root" ]; then \
    addgroup  --system --gid ${GID} ${USER}; \
    adduser --system --home /var/lib/${USER} --uid ${UID} --gid ${GID} --gecos ${USER} --shell /usr/sbin/nologin --disabled-password ${USER}; \
    # sed -i "/%sudo/c ${USER} ALL=(ALL:ALL) NOPASSWD:ALL" /etc/sudoers; \
    fi \
    && apt-get -qqy --purge autoremove \
    && apt-get -qqy clean \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/* \
    && rm -rf /var/tmp/* 
    
COPY --from=builder /root/sftpgo-${SFTPGO_VERSION}/sftpgo.json /etc/sftpgo/sftpgo.json
COPY --from=builder /root/sftpgo-${SFTPGO_VERSION}/templates /usr/share/sftpgo/templates
COPY --from=builder /root/sftpgo-${SFTPGO_VERSION}/static /usr/share/sftpgo/static
COPY --from=builder /root/sftpgo-${SFTPGO_VERSION}/openapi /usr/share/sftpgo/openapi
COPY --from=builder /root/sftpgo-${SFTPGO_VERSION}/sftpgo /usr/local/bin/sftpgo-plugin-* /usr/local/bin/

# Log to the stdout so the logs will be available using docker logs
ENV SFTPGO_LOG_FILE_PATH=""

# Modify the default configuration file
RUN sed -i 's|"users_base_dir": "",|"users_base_dir": "/srv/sftpgo/data",|' /etc/sftpgo/sftpgo.json && \
    sed -i 's|"backups"|"/srv/sftpgo/backups"|' /etc/sftpgo/sftpgo.json

RUN chown -R sftpgo:sftpgo /etc/sftpgo /srv/sftpgo && chown sftpgo:sftpgo /var/lib/sftpgo && chmod 700 /srv/sftpgo/backups

# Switch to the user
# USER ${USER}
USER root

# Set the workdir
WORKDIR ${WORKDIR}

COPY docker-entrypoint.sh /usr/local/bin/

ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 8080 2022 

CMD ["sftpgo", "serve"]
