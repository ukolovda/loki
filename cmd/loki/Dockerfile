FROM golang:1.15.3 as build

# build golang environment
ENV PROTOC_VER 3.14.0

RUN apt-get update && \
    apt-get install -y apt-utils \
                       zip \
                       unzip && \
    wget -q -P /tmp/temp/ https://github.com/protocolbuffers/protobuf/releases/download/v${PROTOC_VER}/protoc-${PROTOC_VER}-linux-x86_64.zip && \
    cd /usr && \
    unzip /tmp/temp/protoc-${PROTOC_VER}-linux-x86_64.zip

RUN go env -w GO111MODULE=on && \
    go get -u github.com/gogo/protobuf/proto \
              github.com/gogo/protobuf/protoc-gen-gogoslick && \
    rm -rf /var/lib/apt/lists/* \
           /tmp/temp/

# TOUCH_PROTOS signifies if we should touch the compiled proto files and thus not regenerate them.
# This is helpful when file system timestamps can't be trusted with make
ARG TOUCH_PROTOS
COPY . /src/loki
WORKDIR /src/loki
RUN make clean && (if [ "${TOUCH_PROTOS}" ]; then make touch-protos; fi) && make BUILD_IN_CONTAINER=false loki

FROM alpine:3.9

RUN apk add --no-cache ca-certificates libcap

COPY --from=build /src/loki/cmd/loki/loki /usr/bin/loki
COPY cmd/loki/loki-docker-config.yaml /etc/loki/local-config.yaml

RUN addgroup -g 10001 -S loki && \
    adduser -u 10001 -S loki -G loki
RUN mkdir -p /loki/rules && \
    mkdir -p /loki/tmprules && \
    chown -R loki:loki /etc/loki /loki

# See https://github.com/grafana/loki/issues/1928
RUN [ ! -e /etc/nsswitch.conf ] && echo 'hosts: files dns' > /etc/nsswitch.conf

USER loki
EXPOSE 3100
ENTRYPOINT [ "/usr/bin/loki" ]
CMD ["-config.file=/etc/loki/local-config.yaml"]
