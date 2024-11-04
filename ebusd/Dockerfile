# Define build argument for the base image
ARG BUILD_FROM
FROM $BUILD_FROM

RUN echo "https://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories
RUN echo "https://dl-cdn.alpinelinux.org/alpine/edge/main" >> /etc/apk/repositories

# store installed packages for cleanup
RUN apk list -I | cut -f1 -d' ' | sed -e 's/-r\d\+$//'| sed 's/\(.*\)-\(.*\)/\1/' >> /etc/apk/packages-before

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
RUN mkdir -p /home/user1/aports/ebusd
COPY APKBUILD /home/user1/aports/ebusd
RUN chown -r user1:user1 /home/user1/aports

# github nosudo hacking
USER root
RUN su user1 PATH=$PATH bash -c "cd /home/user1 && abuild-keygen -a -n"
RUN su user1 PATH=$PATH bash -c "cd /home/user1/aports/ebusd && abuild validate builddeps clean fetch unpack prepare mkusers build check rootpkg"

# install
RUN apk add --allow-untrusted `find /home/user1/packages/aports/ -name ebusd-*.apk`

# cleanup
RUN apk list -I | cut -f1 -d' ' | sed -e 's/-r\d\+$//'| sed 's/\(.*\)-\(.*\)/\1/' >> /etc/apk/packages-after
RUN cat /etc/apk/packages-after | grep -v -f /etc/apk/packages-before >> /etc/apk/packages-remove
RUN apk del `cat /etc/apk/packages-remove`
RUN rm /etc/apk/packages-after /etc/apk/packages-before /etc/apk/packages-remove

RUN rm -rf /home/user1
RUN delgroup abuild
RUN deluser user1

LABEL Description="ebusd"

COPY run.sh /
RUN chmod a+x /run.sh
CMD [ "/run.sh" ]

# Health check
HEALTHCHECK --interval=5m --timeout=3s --start-period=90s \
   CMD nc -z 127.0.0.1 8888 || exit 1
