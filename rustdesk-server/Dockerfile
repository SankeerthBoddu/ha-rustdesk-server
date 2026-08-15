ARG BUILD_FROM=ghcr.io/hassio-addons/base:21.0.1
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Pinned RustDesk Server release and upstream asset digests.
# Digests come from the immutable GitHub release asset metadata.
ARG BUILD_ARCH
ARG RUSTDESK_SERVER_VERSION=1.1.16
ARG RUSTDESK_SERVER_SHA256_AMD64=0565c41affe6c3f409b0bddfaf5a24ccbf3f64f5f8e3fec250b69a0d5f6bdbcf
ARG RUSTDESK_SERVER_SHA256_AARCH64=6a4ae3c5ca257a4278ded72fd17eb2ca4eeb0356a5425e63a3e7fcb0ec6c155c

# Install RustDesk Server
# hadolint ignore=DL3018
RUN \
    apk add --no-cache --virtual .build-deps \
        curl \
        unzip \
    && mkdir -p /opt \
    && if [ "${BUILD_ARCH}" = "aarch64" ]; then \
         ARCH="arm64v8"; \
         SHA256="${RUSTDESK_SERVER_SHA256_AARCH64}"; \
       elif [ "${BUILD_ARCH}" = "amd64" ]; then \
         ARCH="amd64"; \
         SHA256="${RUSTDESK_SERVER_SHA256_AMD64}"; \
       else \
         echo "Unsupported architecture: ${BUILD_ARCH}"; \
         exit 1; \
       fi \
    && curl --fail --location --retry 3 --output /tmp/rustdesk-server.zip \
        "https://github.com/rustdesk/rustdesk-server/releases/download/${RUSTDESK_SERVER_VERSION}/rustdesk-server-linux-${ARCH}.zip" \
    && echo "${SHA256}  /tmp/rustdesk-server.zip" | sha256sum -c \
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
    io.hass.type="app" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Sankeerth Boddu <https://github.com/sankeerthboddu>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Community Apps" \
    org.opencontainers.image.authors="Sankeerth Boddu <https://github.com/sankeerthboddu>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://github.com/sankeerthboddu/ha-rustdesk-server" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/sankeerthboddu/ha-rustdesk-server/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
