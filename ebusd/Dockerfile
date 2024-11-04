# Define build argument for the base image
ARG BUILD_FROM
FROM $BUILD_FROM

RUN env

RUN echo "https://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories
RUN echo "https://dl-cdn.alpinelinux.org/alpine/edge/main" >> /etc/apk/repositories

# install required packages to build "alpine linux" packages
RUN apk add --update --no-cache --no-progress alpine-sdk coreutils bash sudo cmake mosquitto-dev openssl-dev samurai apk-tools

# setup directory for built packages
RUN mkdir -p /var/cache/distfiles
RUN chmod a+w /var/cache/distfiles
RUN chgrp abuild /var/cache/distfiles
RUN chmod g+w /var/cache/distfiles

# setup the abuild configuration
RUN echo 'PACKAGER="Piotr Gasidlo <quaker@barbara.eu.org>"' >> /etc/abuild.conf
RUN echo 'MAINTAINER="$PACKAGER"' >> /etc/abuild.conf
RUN echo "%abuild ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/abuild

# setup the build user in container
RUN adduser -D user1
RUN addgroup user1 abuild

# setup directory to pass in build instructions files for abuild
VOLUME /home/user1/aports
WORKDIR /home/user1/aports
RUN chown user1:user1 /home/user1/aports

# create keys for signing packages after the build has been finished

# make user1 the current user in the image
USER user1
RUN mkdir -p /home/user1/aports/ebusd
COPY APKBUILD /home/user1/aports/ebusd

USER root

# Set architecture in an environment variable
ARG ARCH
RUN ARCH=$(uname -m) && echo "Architecture is $ARCH"

RUN su - user1 CBUILD=$ARCH PATH=$PATH bash -c "cd /home/user1 && CBUILD=`uname -m` abuild-keygen -a -n"
RUN su - user1 CBUILD=$ARCH PATH=$PATH bash -c "cd /home/user1/aports/ebusd && CBUILD=`uname -m` abuild validate builddeps clean fetchunpack prepare mkusers build check rootpkg"

RUN apk add --allow-untrusted /home/user1/packages/aports/$ARCH/ebusd-24.1-r0.apk

LABEL Description="ebusd"

COPY run.sh /
RUN chmod a+x /run.sh
CMD [ "/run.sh" ]

# Health check
HEALTHCHECK --interval=5m --timeout=3s --start-period=90s \
   CMD nc -z 127.0.0.1 8888 || exit 1
