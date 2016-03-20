FROM hypriot/rpi-golang:tar-1.5.2

RUN apt-get update && \
    apt-get install -y wget apache2-utils && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /root

RUN wget https://github.com/hypriot/golang-armbuilds/releases/download/v1.5.2/go1.5.2.linux-armv7.tar.gz && tar -xvzf go1.5.2.linux-armv7.tar.gz -C /usr/local
ENV PATH /usr/local/go/bin:$PATH

ENV DISTRIBUTION_DIR /go/src/github.com/docker/distribution
ENV GOPATH $DISTRIBUTION_DIR/Godeps/_workspace:/go
ENV GOROOT /usr/local/go
ENV DOCKER_BUILDTAGS include_oss include_gcs

WORKDIR $DISTRIBUTION_DIR
COPY . $DISTRIBUTION_DIR
COPY cmd/registry/config-dev.yml /etc/docker/registry/config.yml
RUN make PREFIX=/go clean binaries

VOLUME ["/var/lib/registry"]
EXPOSE 5000
ENTRYPOINT ["registry"]
CMD ["serve", "/etc/docker/registry/config.yml"]
