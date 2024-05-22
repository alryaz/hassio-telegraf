FROM golang:alpine AS builder
ARG YJ_VERSION=5.1.0

# Require some dependencies
RUN apk update && apk add --no-cache git
WORKDIR /workspace

# Download yj utility
RUN \
    git clone https://github.com/sclevine/yj.git /workspace \
    && git checkout "v${YJ_VERSION}"

# Build yj utility
RUN go build -ldflags "-X main.Version=${YJ_VERSION}" .

# Build Telegraf container
ARG BUILD_FROM
FROM $BUILD_FROM as base

# Set up base system
ENV LANG C.UTF-8
#SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install necessary packages
RUN \
    apk update && apk add --no-cache \
        iputils-ping net-snmp procps smartmontools ipmitool \
        lm-sensors lm-sensors-sensord lm-sensors-detect

# Copy over yj utility
COPY --from=builder /workspace/yj /bin/yj

# Copy root filesystem
COPY rootfs /

# Set permissions on root file system
RUN chown root /etc/cont-init.d/telegraf.sh
RUN chown root /etc/telegraf/telegraf.conf
RUN chown root /etc/services.d/telegraf/run
RUN chown root /etc/services.d/telegraf/finish
RUN chmod 775 /etc/cont-init.d/telegraf.sh
RUN chmod 775 /etc/telegraf/telegraf.conf
RUN chmod 775 /etc/services.d/telegraf/run
RUN chmod 775 /etc/services.d/telegraf/finish

# Expose necessary ports
EXPOSE \
    8092/udp \
    8094 \
    8125/udp \
    9273/tcp \
    6514

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Sacrementus" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Sacrementus's addons" \
    org.opencontainers.image.authors="Sacrementus" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
