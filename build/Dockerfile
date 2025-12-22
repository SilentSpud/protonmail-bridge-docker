FROM golang:1.25-trixie AS builder

# Install dependencies
RUN apt-get update && apt-get install -y git build-essential libsecret-1-dev

# Build
RUN git clone https://github.com/ProtonMail/proton-bridge.git /build/
WORKDIR /build/
RUN make build-nogui

FROM debian:trixie-slim

EXPOSE 25/tcp
EXPOSE 143/tcp

ENV DISABLE_TELEMETRY=true
ENV DISABLE_AUTO_UPDATE=true
ENV USE_TLS=true
ENV INFO_DIR=
ENV SMTP_PORT=25
ENV IMAP_PORT=143

# Add healthcheck to monitor proton-bridge process
HEALTHCHECK --interval=30s --timeout=10s --retries=3 --start-period=60s \
  CMD bash -c "pgrep -f proton-bridge || exit 1"

# Install dependencies and remove apt cache to reduce image size
RUN apt-get update \
    && apt-get install -y --no-install-recommends expect procps socat pass libsecret-1-0 ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Copy protonmail
COPY --from=builder /build/bridge /usr/local/bin/
COPY --from=builder /build/proton-bridge /usr/local/bin/

# Copy bash scripts
ADD bridgectl /usr/local/bin/
ADD bridge-init.exp /usr/local/share/protonmail/
RUN chmod 755 /usr/local/bin/bridgectl /usr/local/share/protonmail/bridge-init.exp

# Add a user 'protonmail'
RUN useradd -d /protonmail protonmail \
    && mkdir -p /protonmail \
    && chown protonmail: /protonmail

# change to non-privileged user for extra security
USER protonmail
VOLUME /protonmail

ENTRYPOINT ["/usr/local/bin/bridgectl"]
