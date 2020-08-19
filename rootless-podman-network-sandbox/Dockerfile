ARG GOLANG_VERSION=1.15
ARG ALPINE_VERSION=3.12
ARG CNI_VERSION=v0.8.0
ARG CNI_PLUGINS_VERSION=v0.8.6
# Jul 15, 2020
ARG DNSNAME_VESION=8ffd35849663cc69582c860d54f86868a3c19487

FROM golang:${GOLANG_VERSION}-alpine${ALPINE_VERSION} AS golang-base
RUN apk add --no-cache git

FROM golang-base AS cnitool
RUN git clone https://github.com/containernetworking/cni /go/src/github.com/containernetworking/cni
WORKDIR /go/src/github.com/containernetworking/cni
ARG CNI_VERSION
RUN git checkout ${CNI_VERSION}
RUN go build -o /cnitool ./cnitool

FROM golang-base AS dnsname
RUN git clone https://github.com/containers/dnsname /go/src/github.com/containers/dnsname
WORKDIR /go/src/github.com/containers/dnsname
ARG DNSNAME_VERSION
RUN git checkout ${DNSNAME_VERSION}
RUN go build -o /dnsname ./plugins/meta/dnsname

FROM alpine:${ALPINE_VERSION}
RUN apk add --no-cache curl dnsmasq iptables ip6tables iproute2
ARG TARGETARCH
ARG CNI_PLUGINS_VERSION
RUN mkdir -p /opt/cni/bin && \
    curl -fsSL https://github.com/containernetworking/plugins/releases/download/${CNI_PLUGINS_VERSION}/cni-plugins-linux-${TARGETARCH}-${CNI_PLUGINS_VERSION}.tgz | tar xz -C /opt/cni/bin
COPY --from=cnitool /cnitool /usr/local/bin
COPY --from=dnsname /dnsname /opt/cni/bin
ADD alloc /alloc
ADD dealloc /dealloc
ENV CNI_PATH=/opt/cni/bin
CMD ["sleep", "infinity"]

