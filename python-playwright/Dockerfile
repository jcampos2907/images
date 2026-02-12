FROM ubuntu:24.04

ARG VERSION=1.49.1

ENV PYTHONUNBUFFERED=1
ENV DEBIAN_FRONTEND=noninteractive
ENV PLAYWRIGHT_BROWSERS_PATH=/ms-playwright

RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 \
    python3-pip \
    curl \
    && rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir --break-system-packages playwright==${VERSION} \
    && playwright install-deps chromium \
    && playwright install chromium \
    && rm -rf /var/lib/apt/lists/* \
    && python3 -c "from playwright.sync_api import sync_playwright; print('Playwright OK')"