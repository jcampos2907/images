FROM --platform=$TARGETPLATFORM alpine:3.20

ARG VERSION=2.52.0
ENV VERSION=${VERSION}

RUN apk add --no-cache \
    ca-certificates \
    openssh-client \
    bash \
    coreutils \
    curl \
    less \
    perl \
    zlib \
    expat \
    libcurl \
    pcre2 \
    gettext-libs \
    && update-ca-certificates

RUN apk add --no-cache --virtual .build-deps \
    build-base \
    autoconf \
    gettext-dev \
    zlib-dev \
    expat-dev \
    curl-dev \
    pcre2-dev \
    perl-dev \
    tar

WORKDIR /tmp

RUN curl -fsSL "https://www.kernel.org/pub/software/scm/git/git-${VERSION}.tar.gz" -o git.tgz \
    && tar -xzf git.tgz \
    && cd git-${VERSION} \
    && make configure \
    && ./configure --prefix=/usr \
    && make -j"$(nproc)" all \
    && make install \
    && cd / \
    && rm -rf /tmp/git* \
    && apk del .build-deps

RUN git --version
