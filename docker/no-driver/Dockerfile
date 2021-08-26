FROM ubuntu:18.04 as ubuntu

ARG FALCO_VERSION=0.27.0
ARG VERSION_BUCKET=bin

ENV FALCO_VERSION=${FALCO_VERSION}
ENV VERSION_BUCKET=${VERSION_BUCKET}

RUN apt-get -y update && apt-get -y install gridsite-clients curl

WORKDIR /

RUN curl -L -o falco.tar.gz \
    https://download.falco.org/packages/${VERSION_BUCKET}/x86_64/falco-$(urlencode ${FALCO_VERSION})-x86_64.tar.gz && \
    tar -xvf falco.tar.gz && \
    rm -f falco.tar.gz && \
    mv falco-${FALCO_VERSION}-x86_64 falco

RUN sed -e 's/time_format_iso_8601: false/time_format_iso_8601: true/' < /falco/etc/falco/falco.yaml > /falco/etc/falco/falco.yaml.new \
    && mv /falco/etc/falco/falco.yaml.new /falco/etc/falco/falco.yaml


FROM registry.access.redhat.com/ubi8/ubi

LABEL maintainer="cncf-falco-dev@lists.cncf.io"

LABEL usage="docker run -i -t --privileged -v /var/run/docker.sock:/host/var/run/docker.sock -v /dev:/host/dev -v /proc:/host/proc:ro --name NAME IMAGE"
# NOTE: for the "least privileged" use case, please refer to the official documentation

ENV HOST_ROOT /host
ENV HOME /root

RUN yum install kmod dkms -y

COPY --from=ubuntu /falco /

COPY ./docker-entrypoint.sh /

ENTRYPOINT ["/docker-entrypoint.sh"]

CMD ["/usr/bin/falco", "-o", "time_format_iso_8601=true"]
