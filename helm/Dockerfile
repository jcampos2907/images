FROM alpine:3.20

ARG VERSION=v3.16.4
ENV VERSION=${VERSION}
ARG TARGETARCH

RUN apk add --no-cache \
    ca-certificates curl bash git openssh-client \
    && curl -L --fail "https://get.helm.sh/helm-${VERSION}-linux-${TARGETARCH}.tar.gz" \
    -o /tmp/helm.tgz \
    && tar -xzf /tmp/helm.tgz -C /tmp \
    && mv /tmp/linux-${TARGETARCH}/helm /usr/local/bin/helm \
    && chmod +x /usr/local/bin/helm \
    && rm -rf /tmp/* \
    && helm version

ENTRYPOINT ["helm"]
CMD ["help"]