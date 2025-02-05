FROM golang:1.23.6-alpine as build-stage

RUN apk --no-cache add \
    g++ \
    git \
    make \
    bash \
    ca-certificates \
    tzdata

ARG VERSION
ENV VERSION=${VERSION}
RUN adduser \
    --disabled-password \
    --gecos "" \
    --home "/nonexistent" \
    --shell "/sbin/nologin" \
    --no-create-home \
    --uid 65532 \
    minimal
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download && go mod verify
COPY cmd cmd
COPY internal internal
COPY scripts/build/build.sh build.sh
RUN --mount=type=cache,id=gomod,target=/go/pkg/mod \
    --mount=type=cache,id=gobuild,target=/root/.cache/go-build \
    ./build.sh

# Final image.
FROM scratch
COPY --from=build-stage /usr/share/zoneinfo /usr/share/zoneinfo
COPY --from=build-stage /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=build-stage /etc/passwd /etc/passwd
COPY --from=build-stage /etc/group /etc/group
COPY --from=build-stage /src/bin/k8s-sizing-webhook /k8s-sizing-webhook
USER minimal:minimal
ENTRYPOINT ["/k8s-sizing-webhook"]