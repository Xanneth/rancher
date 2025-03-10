FROM ubuntu:18.04
RUN apt-get update && apt-get install -y git curl ca-certificates unzip xz-utils && \
    useradd rancher && \
    mkdir -p /var/lib/rancher/etcd /var/lib/cattle /opt/jail /opt/drivers/management-state/bin && \
    chown -R rancher /var/lib/rancher /var/lib/cattle /usr/local/bin
RUN mkdir /root/.kube && \
    ln -s /etc/rancher/k3s/k3s.yaml /root/.kube/k3s.yaml  && \
    ln -s /etc/rancher/k3s/k3s.yaml /root/.kube/config && \
    ln -s /usr/bin/rancher /usr/bin/reset-password && \
    ln -s /usr/bin/rancher /usr/bin/ensure-default-admin && \
    rm -f /bin/sh && ln -s /bin/bash /bin/sh
WORKDIR /var/lib/rancher

ARG ARCH=amd64
ARG IMAGE_REPO=rancher
ARG SYSTEM_CHART_DEFAULT_BRANCH=v2.3-dev

ENV CATTLE_SYSTEM_CHART_DEFAULT_BRANCH=$SYSTEM_CHART_DEFAULT_BRANCH
ENV CATTLE_HELM_VERSION v2.14.3-rancher1
ENV CATTLE_K3S_VERSION v0.8.0
ENV CATTLE_MACHINE_VERSION v0.15.0-rancher16-1
ENV CATTLE_ETCD_VERSION v3.3.14
ENV LOGLEVEL_VERSION v0.1.2
ENV TINI_VERSION v0.18.0
ENV TELEMETRY_VERSION v0.5.10
ENV KUBECTL_VERSION v1.16.1
ENV DOCKER_MACHINE_LINODE_VERSION v0.1.8
ENV LINODE_UI_DRIVER_VERSION v0.3.0

RUN mkdir -p /var/lib/rancher-data/local-catalogs/system-library && \
    mkdir -p /var/lib/rancher-data/local-catalogs/library && \
    git clone -b $CATTLE_SYSTEM_CHART_DEFAULT_BRANCH --single-branch https://github.com/rancher/system-charts /var/lib/rancher-data/local-catalogs/system-library && \
    git clone -b master --single-branch https://github.com/rancher/charts /var/lib/rancher-data/local-catalogs/library


RUN curl -sLf https://github.com/rancher/machine-package/releases/download/${CATTLE_MACHINE_VERSION}/docker-machine-${ARCH}.tar.gz | tar xvzf - -C /usr/bin && \
    curl -sLf https://github.com/rancher/loglevel/releases/download/${LOGLEVEL_VERSION}/loglevel-${ARCH}-${LOGLEVEL_VERSION}.tar.gz | tar xvzf - -C /usr/bin && \
    curl -LO https://github.com/linode/docker-machine-driver-linode/releases/download/${DOCKER_MACHINE_LINODE_VERSION}/docker-machine-driver-linode_linux-amd64.zip && \
    unzip docker-machine-driver-linode_linux-amd64.zip -d /opt/drivers/management-state/bin && \
    rm docker-machine-driver-linode_linux-amd64.zip

ENV TINI_URL_amd64=https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini \
    TINI_URL_arm64=https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-arm64 \
    TINI_URL=TINI_URL_${ARCH}

ENV HELM_URL_amd64=https://github.com/rancher/helm/releases/download/${CATTLE_HELM_VERSION}/rancher-helm \
    HELM_URL_arm64=https://github.com/rancher/helm/releases/download/${CATTLE_HELM_VERSION}/rancher-helm-arm64 \
    HELM_URL=HELM_URL_${ARCH} \
    TILLER_URL_amd64=https://github.com/rancher/helm/releases/download/${CATTLE_HELM_VERSION}/rancher-tiller \
    TILLER_URL_arm64=https://github.com/rancher/helm/releases/download/${CATTLE_HELM_VERSION}/rancher-tiller-arm64 \
    TILLER_URL=TILLER_URL_${ARCH} \
    K3S_URL_amd64=https://github.com/rancher/k3s/releases/download/${CATTLE_K3S_VERSION}/k3s \
    K3S_URL_arm64=https://github.com/rancher/k3s/releases/download/${CATTLE_K3S_VERSION}/k3s-arm64 \
    K3S_URL=K3S_URL_${ARCH} \
    ETCD_URL_amd64=https://github.com/etcd-io/etcd/releases/download/${CATTLE_ETCD_VERSION}/etcd-${CATTLE_ETCD_VERSION}-linux-amd64.tar.gz \
    ETCD_URL_arm64=https://github.com/etcd-io/etcd/releases/download/${CATTLE_ETCD_VERSION}/etcd-${CATTLE_ETCD_VERSION}-linux-arm64.tar.gz \
    ETCD_URL=ETCD_URL_${ARCH}

RUN curl -sLf ${!TINI_URL} > /usr/bin/tini && \
    curl -sLf ${!HELM_URL} > /usr/bin/rancher-helm && \
    curl -sLf ${!TILLER_URL} > /usr/bin/rancher-tiller && \
    curl -sLf ${!K3S_URL} > /usr/bin/k3s && \
    curl -sfL ${!ETCD_URL} | tar xvzf - --strip-components=1 -C /usr/bin/ etcd-${CATTLE_ETCD_VERSION}-linux-${ARCH}/etcd etcd-${CATTLE_ETCD_VERSION}-linux-${ARCH}/etcdctl && \
    curl -sLf https://github.com/rancher/telemetry/releases/download/${TELEMETRY_VERSION}/telemetry-${ARCH} > /usr/bin/telemetry && \
    curl -sLf https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/${ARCH}/kubectl > /usr/bin/kubectl && \
    chmod +x /usr/bin/rancher-helm /usr/bin/tini /usr/bin/telemetry /usr/bin/rancher-tiller /usr/bin/k3s /usr/bin/kubectl

ENV CATTLE_UI_PATH /usr/share/rancher/ui
ENV CATTLE_UI_VERSION 2.3.12
ENV CATTLE_CLI_VERSION v2.3.0

# Please update the api-ui-version in pkg/settings/settings.go when updating the version here.
ENV CATTLE_API_UI_VERSION 1.1.6

RUN mkdir -p /var/log/auditlog
ENV AUDIT_LOG_PATH /var/log/auditlog/rancher-api-audit.log
ENV AUDIT_LOG_MAXAGE 10
ENV AUDIT_LOG_MAXBACKUP 10
ENV AUDIT_LOG_MAXSIZE 100
ENV AUDIT_LEVEL 0
VOLUME /var/log/auditlog

RUN mkdir -p /usr/share/rancher/ui && \
    cd /usr/share/rancher/ui && \
    curl -sL https://releases.rancher.com/ui/${CATTLE_UI_VERSION}.tar.gz | tar xvzf - --strip-components=1 && \
    mkdir -p assets/rancher-ui-driver-linode && \
    cd assets/rancher-ui-driver-linode && \
    curl -O https://linode.github.io/rancher-ui-driver-linode/releases/${LINODE_UI_DRIVER_VERSION}/component.js && \
    curl -O https://linode.github.io/rancher-ui-driver-linode/releases/${LINODE_UI_DRIVER_VERSION}/component.css && \
    curl -O https://linode.github.io/rancher-ui-driver-linode/releases/${LINODE_UI_DRIVER_VERSION}/linode.svg && \
    mkdir -p /usr/share/rancher/ui/api-ui && \
    cd /usr/share/rancher/ui/api-ui && \
    curl -sL https://releases.rancher.com/api-ui/${CATTLE_API_UI_VERSION}.tar.gz | tar xvzf - --strip-components=1 && \
    cd /var/lib/rancher

ENV CATTLE_CLI_URL_DARWIN  https://releases.rancher.com/cli2/${CATTLE_CLI_VERSION}/rancher-darwin-amd64-${CATTLE_CLI_VERSION}.tar.gz
ENV CATTLE_CLI_URL_LINUX   https://releases.rancher.com/cli2/${CATTLE_CLI_VERSION}/rancher-linux-amd64-${CATTLE_CLI_VERSION}.tar.gz
ENV CATTLE_CLI_URL_WINDOWS https://releases.rancher.com/cli2/${CATTLE_CLI_VERSION}/rancher-windows-386-${CATTLE_CLI_VERSION}.zip

ARG VERSION=dev
ENV CATTLE_SERVER_VERSION ${VERSION}
COPY entrypoint.sh rancher /usr/bin/
COPY jailer.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh

ENV CATTLE_AGENT_IMAGE ${IMAGE_REPO}/rancher-agent:${VERSION}
ENV CATTLE_SERVER_IMAGE ${IMAGE_REPO}/rancher
ENV ETCD_UNSUPPORTED_ARCH=${ARCH}
ENV ETCDCTL_API=3

ENV SSL_CERT_DIR /etc/rancher/ssl
VOLUME /var/lib/rancher

ENTRYPOINT ["entrypoint.sh"]
