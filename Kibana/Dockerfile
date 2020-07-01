FROM ubuntu:18.04

ARG TARGETPLATFORM
ARG BUILDPLATFORM

ARG KIBANA_VERSION
ARG KIBANA_NODE_VERSION

###############################################################################
#                                INSTALLATION
###############################################################################

### install prerequisites


RUN set -e \
 && apt update -y \
 && apt install -y --no-install-recommends ca-certificates \
 && apt install -y --no-install-recommends curl \
 && apt install -y --no-install-recommends gosu \
 && apt install -y --no-install-recommends wget \
 && apt install -y --no-install-recommends libkrb5-dev \
 && apt install -y --no-install-recommends git \
 && apt install -y --no-install-recommends libfontconfig

 RUN set -e \
 && apt install -y --no-install-recommends python \
 && DEBIAN_FRONTEND=noninteractive apt install -y --no-install-recommends krb5-config \
 && apt install -y --no-install-recommends libssl-dev

 RUN set -e \
 && apt install -y --no-install-recommends libsasl2-dev \
 && apt install -y --no-install-recommends libsasl2-modules-gssapi-mit \
 && apt install -y --no-install-recommends gcc

 RUN set -e \
 && apt install -y --no-install-recommends libc-dev \
 && apt install -y --no-install-recommends make \
 && apt install -y --no-install-recommends g++ \
 && apt install -y --no-install-recommends build-essential

 RUN set -e \
 && apt autoremove -y \
 && apt clean \
 && rm -rf /var/lib/apt/lists/*

 ### install Kibana

ENV \
 KIBANA_HOME=/opt/kibana \
 KIBANA_PACKAGE=kibana-${KIBANA_VERSION}-linux-x86_64.tar.gz \
 KIBANA_GID=993 \
 KIBANA_UID=993

RUN mkdir ${KIBANA_HOME} \
 && curl -O https://artifacts.elastic.co/downloads/kibana/${KIBANA_PACKAGE} \
 && tar xzf ${KIBANA_PACKAGE} -C ${KIBANA_HOME} --strip-components=1 \
 && rm -f ${KIBANA_PACKAGE} \
 && groupadd -r kibana -g ${KIBANA_GID} \
 && useradd -r -s /usr/sbin/nologin -d ${KIBANA_HOME} -c "Kibana service user" -u ${KIBANA_UID} -g kibana kibana \
 && mkdir -p /var/log/kibana \
 && chown -R kibana:kibana ${KIBANA_HOME} /var/log/kibana

 ###############################################################################
#                              START-UP SCRIPTS
###############################################################################

### Kibana

RUN set -x \
  && if [ "${TARGETPLATFORM}" = "linux/arm/v7" ] ; then rm -rf /opt/kibana/node \
  && curl -O https://nodejs.org/dist/v${KIBANA_NODE_VERSION}/node-v${KIBANA_NODE_VERSION}-linux-armv6l.tar.gz \
  && tar -xvf node-v${KIBANA_NODE_VERSION}-linux-armv6l.tar.gz \
  && mv node-v${KIBANA_NODE_VERSION}-linux-armv6l /opt/kibana/node ; fi

  RUN set -x \
  && if [ "${TARGETPLATFORM}" = "linux/arm64" ] ; then rm -rf /opt/kibana/node \
  && curl -O https://nodejs.org/dist/v${KIBANA_NODE_VERSION}/node-v${KIBANA_NODE_VERSION}-linux-arm64.tar.gz \
  && tar -xvf node-v${KIBANA_NODE_VERSION}-linux-arm64.tar.gz \
  && mv node-v${KIBANA_NODE_VERSION}-linux-arm64 /opt/kibana/node ; fi

EXPOSE 5601

USER ${KIBANA_UID}

WORKDIR /opt/kibana
RUN chmod a+x /opt/kibana/bin/kibana
ADD kibana.yml /opt/kibana/config/kibana.yml
CMD ["/opt/kibana/bin/kibana"]