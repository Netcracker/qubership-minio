# Build binaries
FROM golang:1.25.4-alpine3.22 AS builder

ENV MINIO_VERSION=RELEASE.2025-10-15T17-29-55Z
ENV MC_VERSION=RELEASE.2025-08-13T08-35-41Z

# Install minimal build deps
RUN apk add --no-cache git

WORKDIR /workspace

# Clone & build MinIO
RUN git clone --depth=1 --branch ${MINIO_VERSION} https://github.com/minio/minio.git && \
    cd minio && \
    go mod edit -go=1.25 -toolchain=go1.25.4 && \
    go mod tidy && \
    go mod download && \
    CGO_ENABLED=0 GOOS=linux GO111MODULE=on go build -ldflags "-X github.com/minio/minio/cmd.Version=${MINIO_VERSION} -X github.com/minio/minio/cmd.ReleaseTag=${MINIO_VERSION}" -a -o /tmp/minio .

# Clone & build mc
RUN git clone --depth=1 --branch ${MC_VERSION} https://github.com/minio/mc.git && \
    cd mc && \
    go mod edit -go=1.25 -toolchain=go1.25.4 && \
    go mod tidy && \
    go mod download && \
    CGO_ENABLED=0 GOOS=linux GO111MODULE=on go build -a -o /tmp/mc .

# Runtime image
FROM registry.access.redhat.com/ubi9/ubi-micro:latest

# Copy artifacts
COPY --from=builder /tmp/minio /usr/bin/minio
COPY --from=builder /tmp/mc /usr/bin/mc
COPY --from=builder /workspace/minio/dockerscripts/docker-entrypoint.sh /usr/bin/

RUN chmod +x /usr/bin/docker-entrypoint.sh

ENV MC_CONFIG_DIR=/tmp

EXPOSE 9000

ENTRYPOINT ["/usr/bin/docker-entrypoint.sh"]

VOLUME ["/data"]

CMD ["minio"]
