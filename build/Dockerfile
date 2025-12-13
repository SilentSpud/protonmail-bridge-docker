FROM golang:1.25-trixie AS build

# Install dependencies
RUN apt-get update && apt-get install -y git build-essential libsecret-1-dev

# Build
RUN git clone https://github.com/ProtonMail/proton-bridge.git /build/
WORKDIR /build/
RUN make build-nogui vault-editor

FROM debian:trixie-slim

EXPOSE 25/tcp
EXPOSE 143/tcp

# Add healthcheck to monitor proton-bridge process
HEALTHCHECK --interval=30s --timeout=10s --retries=3 --start-period=60s \
  CMD bash -c "pgrep -f proton-bridge || exit 1"

# Install dependencies and remove apt cache to reduce image size
RUN apt-get update \
    && apt-get install -y --no-install-recommends procps socat pass libsecret-1-0 ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Copy bash scripts
ADD gpgparams entrypoint.sh /protonmail/
RUN chmod 755 /protonmail/entrypoint.sh \
    && chmod 644 /protonmail/gpgparams

# Copy protonmail
COPY --from=build /build/bridge /protonmail/
COPY --from=build /build/proton-bridge /protonmail/
COPY --from=build /build/vault-editor /protonmail/

# Add a user 'protonmail' with UID 8535
RUN useradd -u 8535 -d /home/protonmail protonmail \
    && mkdir -p /home/protonmail \
    && chown protonmail: /home/protonmail

# change to non-privileged user for extra security
USER protonmail

ENTRYPOINT ["bash", "/protonmail/entrypoint.sh"]
