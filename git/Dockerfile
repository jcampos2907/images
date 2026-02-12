FROM alpine:3.20

ARG VERSION=2.52.0

RUN echo "https://mirror.nju.edu.cn/alpine/v3.20/main" > /etc/apk/repositories && \
    echo "https://mirror.nju.edu.cn/alpine/v3.20/community" >> /etc/apk/repositories && \
    apk add --no-cache \
    git \
    ca-certificates \
    openssh-client \
    bash \
    curl \
    && git --version