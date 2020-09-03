FROM ubuntu:bionic
MAINTAINER Ville Salomäki <vsalomaki@gmail.com>
## Forked from Onni Hakala's debian version and Hannu Kumpulas xenial-version

# Build arguments
ARG S6_OVERLAY_VERSION=v1.21.2.0
ARG S6_OVERLAY_SHA256=0b71d24d655a4a5eaa21b90a12dff7debd5a3d33abd90165b0a114b787140dfe
ARG GOSU_VERSION=1.10

# Finland is quite nice place to live.
# Instead of forking this you should move your living address here.
ENV TZ="Europe/Helsinki" \
    DEBIAN_FRONTEND=noninteractive




RUN set -x \

    # Add all deb-src addresses
    #    && echo "deb http://mirrors.nic.funet.fi/ubuntu/ xenial main\n\
    #deb-src http://mirrors.nic.funet.fi/ubuntu/ xenial main\n\
    #deb http://mirrors.nic.funet.fi/ubuntu/ xenial-updates main\n\
    #deb-src http://mirrors.nic.funet.fi/ubuntu/ xenial-updates main" > /etc/apt/sources.list \

    # Update sources and upgrade all packages
    && apt-get update -y \
    && apt-get upgrade -y \
    && apt-get -y install apt-utils \

    ##
    # Rebuild pam with --disable-audit flag
    # Fixes: https://github.com/docker/docker/issues/5899
    # Disabled this because no issue in Xenial
    ##
    #   && apt-get -y build-dep pam \
    #   && cd /tmp/ \
    #   && export CONFIGURE_OPTS=--disable-audit \
    #   && apt-get -b source pam \
    #   && dpkg -i libpam-doc*.deb libpam-modules*.deb libpam-runtime*.deb libpam0g*.deb \
    #   && rm -rf /tmp/* \
    #  && unset CONFIGURE_OPTS \

    # Install ca-certificates
    && apt-get install -y --no-install-recommends curl ca-certificates \

    # This folder is in $PATH by default but isn't created by default
    && mkdir -p /usr/local/sbin \
    && cd /usr/local/sbin \

    # Install sha256sum validator to check that we download the right files
    && curl -L -o validate_sha256sum https://gist.githubusercontent.com/Hooace/94f42d2d107f4e4cf13bb721c5c63218/raw/d3ef37ab4a653e1b7655df55dfeadd54e0bacf84/validate_sha256sum \
    # This is semi meta but validate that our validator is valid
    && sha256sum validate_sha256sum | grep 0f7b790036f7cd00610cbe9e79c5b6b42d5b0e02beaff9549bdc43fc99910709 \
    && echo "Success: validate_sha256sum matches provided sha256sum" || exit 1 \
    && chmod +x validate_sha256sum \

    # Give execution rights to all scripts which we downloaded
    && chmod a+x * \


    ##
    # Add S6-overlay to use S6 process manager
    # source: https://github.com/just-containers/s6-overlay/#the-docker-way
    ##
    && curl -L -o /tmp/s6-overlay-amd64.tar.gz https://github.com/just-containers/s6-overlay/releases/download/${S6_OVERLAY_VERSION}/s6-overlay-amd64.tar.gz \
    && validate_sha256sum /tmp/s6-overlay-amd64.tar.gz ${S6_OVERLAY_SHA256} \
    && tar -zxvf /tmp/s6-overlay-amd64.tar.gz -C / \

    # Add default timezone
    ## TODO: could not get env working so tz is now hardcoded
    && apt-get install -y tzdata \
    &&  /bin/ln -fs /usr/share/zoneinfo/${TZ} /etc/localtime \

    # Install envsubst command for replacing config files in system startup
    # - it needs libintl package
    # - only weights 100KB combined with it's libraries
    
    
    && apt-get install -y gettext \
    && mv /usr/bin/envsubst /usr/local/sbin/envsubst \

    ##
    # Install gosu to fix misbehaving su in some environments
    ##
    && dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')" \
    && curl -L -o /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch" \
    && curl -L -o /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    
    # && mkdir ~/.gnupg \
    # && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \

    && apt-get -y install gpg \
    && gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
    && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
    #"$GNUPGHOME"
    && rm -r /usr/local/bin/gosu.asc \ 
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true \

    ##
    # Create a few aliases
    # - I didn't figure out how to load aliases into sh shell with docker so we add scripts instead
    ##

    # ll
    && echo "#!/bin/sh \nls -lah \"\$@\"" > /usr/local/bin/ll \
    # la
    && echo "#!/bin/sh \nls -A \"\$@\"" > /usr/local/bin/la \
    # l
    && echo "#!/bin/sh \nls -CF \"\$@\"" > /usr/local/bin/l \
    && chmod a+x /usr/local/bin/* \

    # Cleanup
    && apt-get remove --purge -y $(apt-mark showauto) \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/log/apt/*

ENTRYPOINT ["/init"]
