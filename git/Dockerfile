FROM alpine:3.20 AS builder

ARG VERSION=2.47.2

RUN apk add --no-cache \
    make gcc musl-dev \
    curl-dev expat-dev openssl-dev zlib-dev pcre2-dev \
    perl autoconf gettext-dev

RUN wget -qO- "https://mirrors.edge.kernel.org/pub/software/scm/git/git-${VERSION}.tar.gz" \
       | tar -xz -C /tmp \
    && cd /tmp/git-${VERSION} \
    && make prefix=/usr/local \
       NO_TCLTK=1 \
       USE_LIBPCRE2=1 \
       all -j"$(nproc)" \
    && make prefix=/usr/local install

FROM alpine:3.20

RUN apk add --no-cache ca-certificates openssh-client bash curl \
    libcurl expat pcre2 zlib

COPY --from=builder /usr/local/bin/git* /usr/local/bin/
COPY --from=builder /usr/local/libexec/git-core/ /usr/local/libexec/git-core/
COPY --from=builder /usr/local/share/git-core/ /usr/local/share/git-core/

RUN git --version

ENTRYPOINT ["git"]
