FROM alpine:3.20

ARG VERSION=v1.31.3
ARG TARGETARCH

RUN apk add --no-cache ca-certificates curl bash coreutils \
    && curl -fsSL "https://dl.k8s.io/release/${VERSION}/bin/linux/${TARGETARCH}/kubectl" \
       -o /usr/local/bin/kubectl \
    && chmod +x /usr/local/bin/kubectl \
    && kubectl version --client=true

ENTRYPOINT ["kubectl"]
CMD ["help"]