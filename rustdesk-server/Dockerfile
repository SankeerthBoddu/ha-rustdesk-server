ARG BUILD_FROM
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Pinned RustDesk Server release
# https://github.com/rustdesk/rustdesk-server/releases
ARG BUILD_ARCH
ARG RUSTDESK_SERVER_VERSION=1.1.11-1

# Install RustDesk Server
# hadolint ignore=DL3018
RUN \
    apk add --no-cache --virtual .build-deps \
        curl \
        unzip \
    && mkdir -p /opt \
    && if [ "${BUILD_ARCH}" = "aarch64" ]; then ARCH="arm64v8"; \
       elif [ "${BUILD_ARCH}" = "amd64" ]; then ARCH="amd64"; \
       elif [ "${BUILD_ARCH}" = "armv7" ]; then ARCH="armv7"; \
       else echo "Unsupported architecture: ${BUILD_ARCH}" && exit 1; \
       fi \
    && curl -fSL -o /tmp/rustdesk-server.zip \
        "https://github.com/rustdesk/rustdesk-server/releases/download/${RUSTDESK_SERVER_VERSION}/rustdesk-server-linux-${ARCH}.zip" \
    && unzip /tmp/rustdesk-server.zip -d /tmp/rustdesk-server \
    && mv "/tmp/rustdesk-server/${ARCH}" /opt/rustdesk-server \
    && chmod +x /opt/rustdesk-server/hbbs \
    && chmod +x /opt/rustdesk-server/hbbr \
    && chmod +x /opt/rustdesk-server/rustdesk-utils \
    && rm -rf /tmp/rustdesk-server.zip /tmp/rustdesk-server \
    && apk del --no-cache --purge .build-deps

# Copy root filesystem
COPY rootfs /

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
    maintainer="Sankeerth Boddu <https://github.com/sankeerthboddu>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Community Add-ons" \
    org.opencontainers.image.authors="Sankeerth Boddu <https://github.com/sankeerthboddu>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://github.com/sankeerthboddu/ha-rustdesk-server" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/sankeerthboddu/ha-rustdesk-server/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
