FROM alpine:3.20

ARG VERSION
ENV VERSION=${VERSION:-v3.16.3}

RUN apk add --no-cache \
    ca-certificates curl bash git openssh-client \
    && curl -L --fail "https://get.helm.sh/helm-${VERSION}-linux-amd64.tar.gz" \
    -o /tmp/helm.tgz \
    && tar -xzf /tmp/helm.tgz -C /tmp \
    && mv /tmp/linux-amd64/helm /usr/local/bin/helm \
    && chmod +x /usr/local/bin/helm \
    && rm -rf /tmp/* \
    && helm version

ENTRYPOINT ["helm"]
CMD ["help"]
