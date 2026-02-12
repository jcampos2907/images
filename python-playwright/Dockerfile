FROM  ubuntu:24.04



ARG VERSION=1.49.1
ENV VERSION=${VERSION}


ENV PYTHONUNBUFFERED=1
ENV DEBIAN_FRONTEND=noninteractive
ENV PLAYWRIGHT_BROWSERS_PATH=/ms-playwright

RUN apt-get update && apt-get install -y \
    python3.12 \
    python3-pip \
    python3.12-venv \
    curl \
    wget \
    && rm -rf /var/lib/apt/lists/*

RUN ln -s /usr/bin/python3.12 /usr/bin/python

RUN pip install playwright==1.49.1 --break-system-packages

RUN playwright install-deps chromium \
    && playwright install chromium \
    && rm -rf /var/lib/apt/lists/*

RUN python -c "from playwright.sync_api import sync_playwright; print('Playwright OK')"