# Build binaries 
FROM golang:1.27.1-alpine3.24 AS builder

ENV MINIO_VERSION=RELEASE.2025-10-15T17-29-55Z
ENV MC_VERSION=RELEASE.2025-08-13T08-35-41Z

# Install minimal build deps
RUN apk add --no-cache git

WORKDIR /workspace

# Clone & build MinIO
RUN git clone --depth=1 --branch ${MINIO_VERSION} https://github.com/minio/minio.git && \
    cd minio && \
    go mod edit -go=1.27 -toolchain=go1.27.1 \
      -require=golang.org/x/crypto@v0.57.0 \ 
      -require=github.com/go-jose/go-jose/v4@v4.1.5 \
      -require=go.opentelemetry.io/otel/sdk@v1.46.0 \
      -require=google.golang.org/grpc@v1.84.0 \
      -require=github.com/buger/jsonparser@v1.6.1 \
      -require=github.com/rabbitmq/amqp091-go@v1.15.0 \
      -require=github.com/go-openapi/spec@v1.0.1 \
      -require=github.com/go-openapi/validate@v1.0.0 \
      -require=github.com/go-openapi/swag@v0.29.2 \
      -require=github.com/prometheus/prometheus@v0.313.0-rc.1 \
      -require=github.com/apache/thrift@v0.24.0 \
      -require=golang.org/x/text@v0.42.0 \
      -require=golang.org/x/net@v0.59.0 \
      -require=github.com/eclipse/paho.mqtt.golang@v1.5.1 \
      -require=github.com/lestrrat-go/jwx/v4@v4.5.0 \
      -require=filippo.io/edwards25519@v1.2.0 && \
    go mod tidy && \
    go mod download && \
    CGO_ENABLED=0 GOOS=linux GO111MODULE=on go build -ldflags "-X github.com/minio/minio/cmd.Version=${MINIO_VERSION} -X github.com/minio/minio/cmd.ReleaseTag=${MINIO_VERSION}" -a -o /tmp/minio .

# Clone & build mc
RUN git clone --depth=1 --branch ${MC_VERSION} https://github.com/minio/mc.git && \
    cd mc && \
    go mod edit -go=1.27 -toolchain=go1.27.1 \
      -require=golang.org/x/crypto@v0.57.0 \
      -require=google.golang.org/grpc@v1.84.0 \
      -require=github.com/prometheus/prometheus@v0.313.0-rc.1 \
      -require=golang.org/x/text@v0.42.0 \
      -require=golang.org/x/net@v0.59.0 \
      -require=github.com/lestrrat-go/jwx/v4@v4.5.0 && \      
    go mod tidy && \
    go mod download && \
    CGO_ENABLED=0 GOOS=linux GO111MODULE=on go build -a -o /tmp/mc .

# Runtime image
# FROM registry.access.redhat.com/ubi10/ubi-micro:latest
FROM alpine:3.23.3

# Copy artifacts
COPY --from=builder \
    /tmp/minio \
    /tmp/mc \
    /workspace/minio/dockerscripts/docker-entrypoint.sh \
    /usr/bin/

RUN chmod +x /usr/bin/docker-entrypoint.sh

ENV MC_CONFIG_DIR=/tmp

EXPOSE 9000

ENTRYPOINT ["/usr/bin/docker-entrypoint.sh"]

VOLUME ["/data"]

CMD ["minio"]
