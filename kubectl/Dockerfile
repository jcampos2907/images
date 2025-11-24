FROM alpine:3.20


ARG VERSION
ENV KUBECTL_VERSION=${VERSION:-v1.31.3}
# Basic tools Jenkins + kubectl need
RUN apk add --no-cache \
    ca-certificates \
    curl \
    bash \
    coreutils

# Install kubectl
RUN curl -L --fail "https://dl.k8s.io/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl" \
    -o /usr/local/bin/kubectl \
    && chmod +x /usr/local/bin/kubectl \
    && kubectl version --client=true

# # Keepalive so Jenkins can exec sh steps
# CMD ["sh", "-c", "sleep infinity"]

