FROM  alpine:3.20

ARG VERSION=2.52.0
ENV VERSION=${VERSION}

RUN apk add --no-cache \
    git~=${VERSION} \
    ca-certificates \
    openssh-client \
    bash \
    curl \
    && git --version
