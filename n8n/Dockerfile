ARG VERSION=1.76.4
FROM n8nio/n8n:${VERSION}

USER root

RUN apk add --no-cache \
        python3 \
        py3-pip \
        ffmpeg \
    && pip install --no-cache-dir --break-system-packages 'markitdown[all]' \
    && rm -rf /root/.cache /tmp/*

USER node

RUN markitdown --help >/dev/null
