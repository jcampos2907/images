# images/git/Dockerfile
FROM alpine:3.20

ARG VERSION
ENV VERSION=${VERSION:-2.52.0}

RUN apk add --no-cache \
    ca-certificates \
    openssh-client \
    bash \
    coreutils \
    curl \
    git=${VERSION} \
    && update-ca-certificates

# Jenkins will exec commands into this container.
# Keep it alive by default.
# CMD ["sh", "-c", "sleep infinity"]
