FROM debian:jessie
MAINTAINER Onni Hakala <onni.hakala@geniem.com>

# Build arguments
ARG S6_OVERLAY_VERSION=v1.18.1.3

# Finland is quite nice place to live.
# Instead of forking this you should move your living address here.
ENV TZ="Europe/Helsinki" \
    DEBIAN_FRONTEND=noninteractive

RUN set -x \

    # This folder is in $PATH by default but isn't created by default
    && mkdir -p /usr/local/sbin \
    && cd /usr/local/sbin \

    # Install ca-certificates
    && apt-get update \
    && apt-get install ca-certificates \

    # Install sha256sum validator to check that we download the right files
    && wget -q -O validate_sha256sum https://gist.githubusercontent.com/onnimonni/b49779ebc96216771a6be3de46449fa1/raw/d3ef37ab4a653e1b7655df55dfeadd54e0bacf84/validate_sha256sum \
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
    && wget -q -O /tmp/s6-overlay-amd64.tar.gz https://github.com/just-containers/s6-overlay/releases/download/${S6_OVERLAY_VERSION}/s6-overlay-amd64.tar.gz \
    && validate_sha256sum /tmp/s6-overlay-amd64.tar.gz 5bb2c67db9369494578e0205fdcf6f44011a1913b98abbc6a7aac551a7c1b0a8 \
    && tar -zxvf /tmp/s6-overlay-amd64.tar.gz -C / \

    # Add default timezone
    && cp /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo "${TZ}" > /etc/timezone \

    # Install envsubst command for replacing config files in system startup
    # - it needs libintl package
    # - only weights 100KB combined with it's libraries
    && apt-get install -y gettext \
    && mv /usr/bin/envsubst /usr/local/sbin/envsubst \

    ##
    # Create a few aliases
    # - I didn't figure out how to load aliases into sh shell with docker so we add scripts instead
    ##

    # ll
    && echo -e "#!/bin/sh \nls -lah \"\$@\"" > /usr/local/bin/ll \
    # la
    && echo -e "#!/bin/sh \nls -A \"\$@\"" > /usr/local/bin/la \
    # l
    && echo -e "#!/bin/sh \nls -CF \"\$@\"" > /usr/local/bin/l \

    && chmod a+x /usr/local/bin/* \

    # Cleanup
    && apt-get remove -y gettext \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/*

ENTRYPOINT ["/init"]
