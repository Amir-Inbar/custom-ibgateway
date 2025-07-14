##############################################################################
# Setup Stage: install apps
#
# This is a dedicated stage so that download archives don't end up on
# production image and consume unnecessary space.
##############################################################################

# hadolint global ignore=DL3008
FROM ubuntu:24.04 AS setup

ENV IB_GATEWAY_VERSION=10.30.1x
ENV IB_GATEWAY_RELEASE_CHANNEL=latest
ENV IBC_VERSION=3.24.0

WORKDIR /tmp/setup

# Prepare system
RUN apt-get update -y && \
  DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends --yes \
  curl \
  ca-certificates \
  unzip && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/*

# Install IB Gateway from local file
COPY ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh ./
RUN chmod a+x ./ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh && \
  ./ibgateway-${IB_GATEWAY_VERSION}-standalone-linux-x64.sh -q -dir /root/Jts/ibgateway/${IB_GATEWAY_VERSION}

# Install IBC from local zip
COPY IBCLinux-${IBC_VERSION}.zip ./
RUN mkdir /root/ibc && \
  unzip ./IBCLinux-${IBC_VERSION}.zip -d /root/ibc && \
  chmod -R u+x /root/ibc/*.sh && \
  chmod -R u+x /root/ibc/scripts/*.sh

# Copy templates
COPY ./config/ibgateway/jts.ini.tmpl /root/Jts/jts.ini.tmpl
COPY ./config/ibc/config.ini.tmpl /root/ibc/config.ini.tmpl

# Copy scripts
COPY ./scripts /root/scripts

##############################################################################
# Build Stage: build production image
##############################################################################

FROM ubuntu:24.04

ENV IB_GATEWAY_VERSION=10.30.1x
# IB Gateway user constants
ARG USER_ID="${USER_ID:-1000}"
ARG USER_GID="${USER_GID:-1000}"
# IBC env vars
ENV HOME=/home/ibgateway
ENV TWS_MAJOR_VRSN=${IB_GATEWAY_VERSION}
ENV TWS_PATH=${HOME}/Jts
ENV TWS_INI=jts.ini
ENV TWS_INI_TMPL=${TWS_INI}.tmpl
ENV IBC_PATH=${HOME}/ibc
ENV IBC_INI=${HOME}/ibc/config.ini
ENV IBC_INI_TMPL=${IBC_INI}.tmpl
ENV SCRIPT_PATH=${HOME}/scripts
ENV GATEWAY_OR_TWS=gateway

# Copy installed IB Gateway and IBC from setup stage
COPY --from=setup /usr/local/i4j_jres/ /usr/local/i4j_jres
COPY --chown=${USER_ID}:${USER_GID} --from=setup /root/ ${HOME}

# Fix telnet path in commandsend.sh
RUN sed -i 's|/usr/local/bin/telnet|/usr/bin/telnet|' /home/ibgateway/ibc/commandsend.sh

# Prepare system
RUN apt-get update -y && \
  apt-get upgrade -y && \
  DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends --yes \
  gettext-base socat xvfb x11vnc sshpass openssh-client telnet && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists/* && \
  if id ubuntu; then \
    userdel -rf ubuntu; \
  fi && \
  groupadd --gid ${USER_GID} ibgateway && \
  useradd -ms /bin/bash --uid ${USER_ID} --gid ${USER_GID} ibgateway && \
  chmod a+x ${SCRIPT_PATH}/*.sh


USER ${USER_ID}:${USER_GID}
WORKDIR ${HOME}

CMD ["/home/ibgateway/scripts/run.sh"]

LABEL org.opencontainers.image.source="https://github.com/gnzsnz/ib-gateway-docker"
LABEL org.opencontainers.image.url="https://github.com/gnzsnz/ib-gateway-docker/pkgs/container/ib-gateway"
LABEL org.opencontainers.image.description="Docker image with IB Gateway and IBC"
LABEL org.opencontainers.image.licenses="Apache License Version 2.0"
LABEL org.opencontainers.image.version="${IB_GATEWAY_VERSION}-${IB_GATEWAY_RELEASE_CHANNEL}"
