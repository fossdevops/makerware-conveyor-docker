# The details can be found in the following web page.
# https://support.makerbot.com/learn/makerbot-desktop-software/getting-started/how-to-install-makerbot-desktop-for-linux_11262
ARG IMAGE_VERSION=${IMAGE_VERSION:-xenial}
ARG REGISTRY_URI=${REGISTRY_URI:-}
ARG IMAGE_NAME=${IMAGE_NAME:-ubuntu}
FROM ${REGISTRY_URI}${IMAGE_NAME}:${IMAGE_VERSION} as base

# Enable headless (cli) package installs
ARG DEBIAN_FRONTEND=noninteractive

# enable upgrade periodic (daily) trigger, do upgrades, install https support for apt, and install lsb_release
ARG UPDATE_TIMESTAMP
RUN (date > /etc/fossdevops_apt_date) && \
    apt-get update -y && \
    apt-get -o Dpkg::Options::="--force-confold" upgrade -y && \
    apt-get -o Dpkg::Options::="--force-confold" install apt-transport-https lsb-release -y && \
    apt-get -y autoremove && apt-get -y clean && rm -rf /var/lib/apt/lists/*

# Add the repository key
COPY dev@makerbot.com.gpg.key .
COPY fkrull_deadsnakes_ppa.gpg.key .

# Add the fkrull/deadsnakes ppa apt repo (for old python3.4) and install conveyor
# WORK AROUND: conveyor tries to start its service after install right at the end of the postinst script; this will fail,
#              but faking the result from lsb_release and trying to install again SHOULD result in the package working
# WORK AROUND: conveyor install is missing a few required packages and will actually not run without additional package
#              installation; manually added the dependencies
# FYI: apt will always complain about makerbot's repo as it uses sha1 for its gpg signature
ARG UBUNTU_VERSION=${UBUNTU_VERSION:-xenial}
RUN apt-key add dev@makerbot.com.gpg.key && \
    rm dev@makerbot.com.gpg.key && \
    apt-key add fkrull_deadsnakes_ppa.gpg.key && \
    rm fkrull_deadsnakes_ppa.gpg.key && \
    (echo "deb https://downloads.makerbot.com/makerware/ubuntu ${UBUNTU_VERSION} main" > /etc/apt/sources.list.d/makerbot.list) && \
    (echo "deb-src https://downloads.makerbot.com/makerware/ubuntu ${UBUNTU_VERSION} main" >> /etc/apt/sources.list.d/makerbot.list) && \
    (echo "deb http://ppa.launchpad.net/fkrull/deadsnakes/ubuntu ${UBUNTU_VERSION} main" > /etc/apt/sources.list.d/fkrull-ubuntu-deadsnakes-xenial.list) && \
    (echo "# deb-src http://ppa.launchpad.net/fkrull/deadsnakes/ubuntu ${UBUNTU_VERSION} main" >> /etc/apt/sources.list.d/fkrull-ubuntu-deadsnakes-xenial.list) && \
    ls -la /etc/apt/sources.list.d/ && \
    cat /etc/apt/sources.list.d/makerbot.list && \
    apt-get update && \
    apt-get -o Dpkg::Options::="--force-confold" install libopencv-highgui2.4v5 libopencv-calib3d2.4v5 libtinything python3.4 -y && \
    (apt-get -o Dpkg::Options::="--force-confold" install conveyor -y || \
        (which lsb_release && (echo "#!/bin/sh" > lsb_release) && (echo "echo 1" >> lsb_release) && cat lsb_release && chmod +x lsb_release && export "PATH=$(pwd):${PATH}"; \
        apt-get -o Dpkg::Options::="--force-confold" install conveyor -y && rm lsb_release)) && \
    mkdir -p /var/run/conveyor && \
    chown conveyor:conveyor /var/run/conveyor && \
    mkdir -p /var/log/conveyor && \
    chown conveyor:conveyor /var/log/conveyor && \
    apt-get -y autoremove && apt-get -y clean && rm -rf /var/lib/apt/lists/*

# Create empty files for logs for use in shared volumes
RUN cd /var/log/conveyor && \
    touch conveyor-svc.log conveyord.log && \
    chown conveyor:conveyor conveyor-svc.log conveyord.log && \
    chmod 644 conveyor-svc.log conveyord.log

# Start conveyor service
ENV LD_LIBRARY_PATH=/usr/lib/makerbot:/usr/lib
ENTRYPOINT [ "/usr/bin/conveyor-svc", "--config", "/etc/conveyor.conf" ]
CMD []

# Versioning and docker metadata stuff
LABEL org.opencontainers.image.authors='FOSSDevOps <support@fossdevops.com>'
LABEL org.opencontainers.image.url='https://github.com/fossdevops/makerware-conveyor-docker'
LABEL org.opencontainers.image.documentation='https://fossdevops.github.io/makerware-conveyor-docker'
LABEL org.opencontainers.image.source='https://github.com/fossdevops/makerware-conveyor-docker.git'
LABEL org.opencontainers.image.vendor='FOSSDevOps'
#LABEL org.opencontainers.image.licenses=''
LABEL org.opencontainers.image.title='ghcr.io/fossdevops/makerware-conveyor-docker'
LABEL org.opencontainers.image.description="Docker image containing the Makerbot conveyor service"

# set version stuff
ARG VERSION=unset
ARG LONG_FORM_VERSION=unset
RUN (printf 'Makerbot Conveyor Container' > /etc/fossdevops_product) && \
    (printf '%s' "${VERSION}" > /etc/fossdevops_version) && \
    (printf '%s' "${LONG_FORM_VERSION}" > /etc/fossdevops_version_long)

# Changing the user must be last
USER conveyor