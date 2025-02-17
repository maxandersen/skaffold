# Copyright 2019 The Skaffold Authors All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Download kubectl
FROM alpine:3.10 as download-kubectl
ENV KUBECTL_VERSION v1.12.8
ENV KUBECTL_URL https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl
RUN wget -O kubectl "${KUBECTL_URL}"
RUN chmod +x kubectl

# Download helm
FROM alpine:3.10 as download-helm
ENV HELM_VERSION v2.12.0
ENV HELM_URL https://storage.googleapis.com/kubernetes-helm/helm-${HELM_VERSION}-linux-amd64.tar.gz
RUN wget -O helm.tar.gz "${HELM_URL}"
RUN tar -xvf helm.tar.gz --strip-components 1

# Download kustomize
FROM alpine:3.10 as download-kustomize
ENV KUSTOMIZE_VERSION 2.1.0
ENV KUSTOMIZE_URL https://github.com/kubernetes-sigs/kustomize/releases/download/v${KUSTOMIZE_VERSION}/kustomize_${KUSTOMIZE_VERSION}_linux_amd64
RUN wget -O kustomize "${KUSTOMIZE_URL}"
RUN chmod +x kustomize

# Download kompose
FROM alpine:3.10 as download-kompose
ENV KOMPOSE_VERSION v1.18.0
ENV KOMPOSE_URL https://github.com/kubernetes/kompose/releases/download/${KOMPOSE_VERSION}/kompose-linux-amd64
RUN wget -O kompose "${KOMPOSE_URL}"
RUN chmod +x kompose

# Download container-structure-test
FROM alpine:3.10 as download-container-structure-test
ENV CONTAINER_STRUCTURE_TEST_VERSION v1.5.0
ENV CONTAINER_STRUCTURE_TEST_URL https://storage.googleapis.com/container-structure-test/${CONTAINER_STRUCTURE_TEST_VERSION}/container-structure-test-linux-amd64
RUN wget -O container-structure-test "${CONTAINER_STRUCTURE_TEST_URL}"
RUN chmod +x container-structure-test

# Download kind
FROM alpine:3.10 as download-kind
ENV KIND_VERSION v0.4.0
ENV KIND_URL https://github.com/kubernetes-sigs/kind/releases/download/${KIND_VERSION}/kind-linux-amd64
RUN wget -O kind "${KIND_URL}"
RUN chmod +x kind

# Download gcloud
FROM alpine:3.10 as download-gcloud
ENV GCLOUD_VERSION 245.0.0
ENV GCLOUD_URL https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-${GCLOUD_VERSION}-linux-x86_64.tar.gz
RUN wget -O gcloud.tar.gz "${GCLOUD_URL}"
RUN tar -zxf gcloud.tar.gz

# Download bazel
FROM alpine:3.10 as download-bazel
ENV BAZEL_VERSION 0.27.0
ENV BAZEL_URL https://github.com/bazelbuild/bazel/releases/download/${BAZEL_VERSION}/bazel-${BAZEL_VERSION}-linux-x86_64
RUN wget -O bazel "${BAZEL_URL}"
RUN chmod +x bazel

# Download pack
FROM alpine:3.10 as download-pack
ENV PACK_VERSION 0.4.1
ENV PACK_URL https://github.com/buildpack/pack/releases/download/v${PACK_VERSION}/pack-v${PACK_VERSION}-linux.tgz
RUN wget -O pack.tgz "${PACK_URL}"
RUN tar -zxf pack.tgz

FROM gcr.io/gcp-runtimes/ubuntu_16_0_4 as runtime_deps

RUN apt-get update && \
    apt-get install --no-install-recommends --no-install-suggests -y \
    git python unzip && \
    rm -rf /var/lib/apt/lists/*

COPY --from=docker:18.09.6 /usr/local/bin/docker /usr/local/bin/
COPY --from=download-kubectl kubectl /usr/local/bin/
COPY --from=download-helm helm /usr/local/bin/
COPY --from=download-kustomize kustomize /usr/local/bin/
COPY --from=download-kompose kompose /usr/local/bin/
COPY --from=download-container-structure-test container-structure-test /usr/local/bin/
COPY --from=download-bazel bazel /usr/local/bin/
COPY --from=download-gcloud google-cloud-sdk/ /google-cloud-sdk/
COPY --from=download-pack pack /usr/local/bin/

# Finish installation of bazel
RUN bazel version

# Finish installation of gcloud
RUN CLOUDSDK_PYTHON="python2.7" /google-cloud-sdk/install.sh \
    --usage-reporting=false \
    --bash-completion=false \
    --disable-installation-options
ENV PATH=$PATH:/google-cloud-sdk/bin
RUN gcloud auth configure-docker

FROM runtime_deps as builder
RUN apt-get update && apt-get install --no-install-recommends --no-install-suggests -y \
    ca-certificates \
    curl \
    build-essential \
    gcc \
    python-setuptools \
    lsb-release \
    openjdk-8-jdk \
    software-properties-common \
    apt-transport-https && \
    rm -rf /var/lib/apt/lists/*
COPY --from=golang:1.12 /usr/local/go /usr/local/go
ENV PATH /usr/local/go/bin:/go/bin:$PATH
WORKDIR /skaffold
COPY . .

FROM builder as integration
ARG VERSION
COPY --from=download-kind kind /usr/local/bin/
RUN make clean && make out/skaffold-linux-amd64 VERSION=$VERSION && mv out/skaffold-linux-amd64 /usr/bin/skaffold
CMD ["make", "integration"]

FROM runtime_deps as distribution
COPY --from=integration /usr/bin/skaffold /usr/bin/skaffold
RUN skaffold credits -d /THIRD_PARTY_NOTICES