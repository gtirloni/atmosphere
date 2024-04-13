# Copyright (c) 2024 VEXXHOST, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License"); you may
# not use this file except in compliance with the License. You may obtain
# a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
# License for the specific language governing permissions and limitations
# under the License.

FROM golang:1.20 AS ovn-kubernetes
ARG OVN_KUBERNETES_REF=cbff639b83af00e4887b540fc06b880108662780
ADD https://github.com/ovn-org/ovn-kubernetes.git#${OVN_KUBERNETES_REF} /src
COPY patches/ovn-kubernetes /patches/ovn-kubernetes
RUN git -C /src apply --verbose /patches/ovn-kubernetes/*
RUN <<EOF bash -xe
cd /src/go-controller
go build -o /usr/bin/ovn-kube-util ./cmd/ovn-kube-util
EOF

FROM registry.atmosphere.dev/library/openvswitch:zed
ADD --chmod=755 https://dl.k8s.io/release/v1.29.3/bin/linux/amd64/kubectl /usr/local/bin/kubectl
ARG OVN_SERIES=23.03
ARG OVN_VERSION=${OVN_SERIES}.0-69
RUN <<EOF sh -xe
dnf -y --setopt=install_weak_deps=False --setopt=tsflags=nodocs install \
    firewalld-filesystem hostname ovn${OVN_SERIES}-${OVN_VERSION}.el9s procps-ng
dnf -y clean all
rm -rf /var/cache/dnf
EOF
ARG OVN_COMPONENT
RUN <<EOF sh -xe
dnf -y --setopt=install_weak_deps=False --setopt=tsflags=nodocs install \
    ovn${OVN_SERIES}-${OVN_COMPONENT}-${OVN_VERSION}.el9s
dnf -y clean all
rm -rf /var/cache/dnf
EOF
COPY --from=ovn-kubernetes --link /src/dist/images/ovndb-raft-functions.sh /root/ovndb-raft-functions.sh
COPY --from=ovn-kubernetes --link /src/dist/images/ovnkube.sh /root/ovnkube.sh
COPY --from=ovn-kubernetes --link /usr/bin/ovn-kube-util /usr/bin/ovn-kube-util