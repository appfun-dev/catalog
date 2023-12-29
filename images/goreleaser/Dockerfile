ARG BUILDPLATFORM
ARG TARGETOS
ARG TARGETARCH
FROM goreleaser/goreleaser:v1.21.1

USER root
VOLUME /go
WORKDIR /go

#ARG PKG_MIRROR_HOST=mirrors.aliyun.com
#RUN sed -i "s/dl-cdn\.alpinelinux\.org/${PKG_MIRROR_HOST}/g" /etc/apk/repositories

ARG HTTPIE_VERSION=3.2.2
ENV HTTPIE_VERSION=${HTTPIE_VERSION}

RUN apk update \
    && apk add --no-cache \
      bash bash-completion bash-doc \
      git jq vim skopeo podman \
      build-base python3-dev \
    && python3 -m ensurepip \
    && python3 -m pip install --no-cache --upgrade pip wheel setuptools \
    && python3 -m pip install httpie==${HTTPIE_VERSION} \
    && apk del build-base python3-dev \
    && rm -rf /var/cache/apk/* \
    && /bin/bash

COPY mikefarah/yq:4             /usr/bin/yq                      /usr/local/bin/
COPY bitnami/kubectl:1.24       /opt/bitnami/kubectl/bin/kubectl /usr/local/bin/
COPY zillownyc/kustomize:4.5.6  /usr/local/bin/kustomize         /usr/local/bin/
COPY alpine/helm:3.12.1         /usr/bin/helm                    /usr/local/bin/
COPY fluxcd/flux-cli:v2.1.2     /usr/local/bin/flux              /usr/local/bin/
COPY oamdev/vela-cli:v1.8.1     /bin/vela                        /usr/local/bin/

#ARG TARGETOS
#ARG TARGETARCH
ENV HELM_PLUGINS=/usr/local/helm-plugins
RUN dir=/usr/local/helm-plugin-push && mkdir -p ${dir} \
    && OS="$(uname | tr '[:upper:]' '[:lower:]')" \
    && ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" \
    && wget -O ${dir}/plugin.tar.gz "https://github.com/chartmuseum/helm-push/releases/download/v0.10.4/helm-push_0.10.4_${OS}_${ARCH}.tar.gz" \
    && tar -zxvf ${dir}/plugin.tar.gz -C ${dir} \
    && yq e -i 'del(.hooks)' ${dir}/plugin.yaml \
    && rm -rf ${dir}/plugin.tar.gz
RUN helm plugin install /usr/local/helm-plugin-push
