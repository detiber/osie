FROM ubuntu:xenial

MAINTAINER David Laube <dlaube@packet.net>
LABEL Description="Ubuntu baremetal installation image" Vendor="Packet.net" Version="1.0"

VOLUME /statedir
ENTRYPOINT ["/entrypoint.sh"]

ARG GITVERSION
ARG GITBRANCH
ARG DRONEBUILD

ENV OSIE_VERSION ${GITVERSION}
ENV OSIE_BRANCH ${GITBRANCH}
ENV DRONE_BUILD ${DRONEBUILD}

ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=C.UTF-8

# apt sources fix
COPY sources.list.aarch64 /tmp/osie/
RUN if [ $(uname -m) = 'aarch64' ]; then \
        mv /tmp/osie/sources.list.aarch64 /etc/apt/sources.list; \
    fi && \
    rm -rf /tmp/osie

# runtime packages
COPY get-package-list.sh git-lfs-linux-*-v2.5.1-4-g2f166e02 /tmp/osie/
RUN apt-get update -y && \
    apt-get install -y $(/tmp/osie/get-package-list.sh) && \
    apt-get -qy autoremove && \
    apt-get -qy clean && \
    rm -rf /var/lib/apt/lists/* /tmp/osie

# build openssl'd git, done here so we can keep it cached as long as possible
COPY build-git-openssl.sh /tmp/osie/
RUN apt-get update -y && \
    /tmp/osie/build-git-openssl.sh && \
    dpkg --unpack /tmp/osie/git_*.deb && \
    apt-get install -f -y --no-install-recommends && \
    apt-get -y autoremove && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/* /tmp/osie

# build lshw, done here so we can keep it cached as long as possible
COPY build-lshw.sh /tmp/osie/
RUN apt-get update -y && \
    apt-get install -y build-essential && \
    (cd /tmp/osie/ && ./build-lshw.sh) && \
    apt-get -qy remove build-essential && \
    apt-get -qy autoremove && \
    apt-get -qy clean && \
    rm -rf /var/lib/apt/lists/* /tmp/osie/

# build nvme cli, done here so we can keep it cached as long as possible
COPY build-nvme-cli.sh /tmp/osie/
RUN apt-get update -y && \
    apt-get install -y build-essential && \
    (cd /tmp/osie/ && ./build-nvme-cli.sh) && \
    apt-get -qy remove build-essential && \
    apt-get -qy autoremove && \
    apt-get -qy clean && \
    rm -rf /var/lib/apt/lists/* /tmp/osie/

# build mstflint, done here so we can keep it cached as long as possible
COPY build-mstflint.sh /tmp/osie/
RUN apt-get update -y && \
    apt-get install -y build-essential && \
    (cd /tmp/osie/ && ./build-mstflint.sh) && \
    apt-get -qy remove build-essential && \
    apt-get -qy autoremove && \
    apt-get -qy clean && \
    rm -rf /var/lib/apt/lists/* /tmp/osie/

# build smartmontools, done here so we can keep it cached as long as possible
COPY build-smartmontools.sh /tmp/osie/
RUN apt-get update -y && \
    apt-get install -y build-essential && \
    (cd /tmp/osie/ && ./build-smartmontools.sh) && \
    apt-get -qy remove build-essential && \
    apt-get -qy autoremove && \
    apt-get -qy clean && \
    rm -rf /var/lib/apt/lists/* /tmp/osie/

ARG PACKET_HARDWARE_COMMIT=413cdf9392a7c962081403d942666152afd1d843
ARG PACKET_NETWORKING_COMMIT=dd13b2a94daace22ecd97f053b878dbfcf20cb80

RUN curl https://bootstrap.pypa.io/get-pip.py | python3 && \
    pip3 install git+https://github.com/packethost/packet-hardware.git@${PACKET_HARDWARE_COMMIT} && \
    pip3 install git+https://github.com/packethost/packet-networking.git@${PACKET_NETWORKING_COMMIT} && \
    rm -rf ~/.cache/pip*

# static prebuilt git-lfs packages
COPY lfs/git-lfs-linux-*-v2.5.1-4-g2f166e02 /tmp/osie/
RUN mv /tmp/osie/git-lfs-linux-$(uname -m)-* /usr/bin/git-lfs && \
    chmod 755 /usr/bin/git-lfs && \
    git-lfs install && \
    rm -rf /tmp/osie

# LSI CLI
COPY lfs/megacli-noarch-bin.tar /tmp/osie/
RUN tar -xvC / -f /tmp/osie/megacli-noarch-bin.tar && \
    ln -nsf /opt/MegaRAID/MegaCli/MegaCli64 /usr/bin/ && \
    rm -rf /tmp/osie

# PERC CLI
COPY lfs/perccli-1.17.10-1.tar.gz /tmp/osie/
RUN tar -zxvC / -f /tmp/osie/perccli-1.17.10-1.tar.gz && \
    ln -nsf /opt/MegaRAID/perccli/perccli64 /usr/bin/ && \
    rm -rf /tmp/osie

# Marvell CLI
COPY lfs/mvcli-4.1.13.31_A01.zip /tmp/osie/
RUN cd /tmp/osie && \
    unzip mvcli-4.1.13.31_A01.zip && \
    cd mvcli-4.1.13.31_A01/x64/cli && \
    cp -f mvcli /usr/bin && \
    cp -f libmvraid.so /usr/lib && \
    chmod 755 /usr/bin/mvcli && \
    cd && \
    rm -r /tmp/osie

# IPMICFG
COPY lfs/ipmicfg /tmp/osie/
RUN cd /tmp/osie && \
    install -m 755 /tmp/osie/ipmicfg /usr/bin/ipmicfg && \
    rm -rf /tmp/osie

# RACADM
COPY dchipm.ini /tmp/osie/
RUN cd /tmp/osie && \
    if [ $(uname -m) != 'aarch64' ]; then \
      apt-get update && apt-get install -y alien && \
      rpm --import http://linux.dell.com/repo/pgp_pubkeys/0x1285491434D8786F.asc && \
      wget \
        https://dl.dell.com/FOLDER05920767M/1/DellEMC-iDRACTools-Web-LX-9.4.0-3732_A00.tar.gz \
        http://linux.dell.com/repo/community/openmanage/940/bionic/pool/main/s/srvadmin-omilcore/srvadmin-omilcore_9.4.0_amd64.deb && \
      tar -xvf DellEMC-iDRACTools-Web-LX-9.4.0-3732_A00.tar.gz && \
      alien -i iDRACTools/racadm/RHEL8/x86_64/*.rpm && \
      dpkg -i *.deb && \
      apt-get purge -y alien && \
      apt-get autoremove -y && \
      cp dchipm.ini /opt/dell/srvadmin/etc/srvadmin-hapi/ini/ ; \
    fi && \
    rm -rf /tmp/osie

# URL=http://www.mellanox.com/downloads/firmware/mlxup
# VERSION=4.6.0
COPY lfs/mlxup-* /tmp/osie/
RUN install -m755 -D /tmp/osie/mlxup-$(uname -m) /opt/mellanox/mlxup && rm -rf /tmp/osie/

ARG ECLYPSIUM_AGENT_VERSION=2.0.0
ARG ECLYPSIUM_AGENT_SHA256=4bb5cfe3c6c3988b99c36e864f35abd8296120d197864a26f10a4c1868c9e31a
ARG ECLYPSIUM_AGENT_FILENAME=eclypsiumapp-${ECLYPSIUM_AGENT_VERSION}.deb

COPY lfs/${ECLYPSIUM_AGENT_FILENAME} /tmp/
RUN if [ $(uname -m) = 'x86_64' ]; then \
        cd /tmp && \
        echo "${ECLYPSIUM_AGENT_SHA256}  ${ECLYPSIUM_AGENT_FILENAME}" | sha256sum -c && \
        dpkg --unpack "${ECLYPSIUM_AGENT_FILENAME}" && \
        sed -i 's/try_restart_service /#try_restart_service /g' /var/lib/dpkg/info/eclypsiumapp.postinst && \
        dpkg --configure eclypsiumapp && \
        rm -f "${ECLYPSIUM_AGENT_FILENAME}"; \
    fi ;

# freebsd ufs fs fuse
COPY lfs/osie-fuse-* /tmp/osie/
RUN mv /tmp/osie/osie-fuse*$(uname -m).deb /tmp/ && rm -rf /tmp/osie/

RUN useradd packet -d /home/packet -m -U && \
    chown -R packet:packet /home/packet
WORKDIR /home/packet

ADD https://raw.githubusercontent.com/packethost/metal-block-storage/master/metal-block-storage-attach \
    https://raw.githubusercontent.com/packethost/metal-block-storage/master/metal-block-storage-detach \
    /home/packet/

COPY entrypoint.sh /entrypoint.sh
COPY scripts/ /home/packet/

# ensure we always have up to date packages
RUN apt-get -y update && \
    apt-get -y dist-upgrade && \
    apt-get -y autoremove && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/* /home/packet/requirements.txt
