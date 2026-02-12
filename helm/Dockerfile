FROM alpine:3.20

ARG VERSION=v3.16.4
ARG TARGETARCH

RUN apk add --no-cache ca-certificates curl bash git openssh-client \
    && curl -fsSL "https://get.helm.sh/helm-${VERSION}-linux-${TARGETARCH}.tar.gz" \
       | tar -xz -C /tmp \
    && mv /tmp/linux-${TARGETARCH}/helm /usr/local/bin/helm \
    && rm -rf /tmp/* \
    && helm version

ENTRYPOINT ["helm"]
CMD ["help"]