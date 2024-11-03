#ARG BUILD_FROM
#FROM $BUILD_FROM

FROM node:19-alpine

ENV LANG C.UTF-8

RUN echo "https://dl-cdn.alpinelinux.org/alpine/edge/community" \
    >> /etc/apk/repositories
RUN echo "https://dl-cdn.alpinelinux.org/alpine/edge/main" \
    >> /etc/apk/repositories

RUN adduser -G abuild -g "Alpine Package Builder" -s /bin/sh -D builder
COPY APKBUILD /home/builder
RUN echo "builder ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

#RUN apk add --no-cache ebusd=24.1
RUN touch /etc/apk/keys
RUN apk add alpine-sdk abuild-rootbld cmake sudo apk-tools
RUN su -l builder -- `which abuild-keygen` -n -a
RUN su -l builder -- `which abuild` >&2
RUN apk add --allow-untrusted /home/builder/packages/home/`uname -m`/ebusd-24.1-r0.apk

LABEL Description="ebusd"

COPY run.sh /
RUN chmod a+x /run.sh
CMD [ "/run.sh" ]

# Health check
HEALTHCHECK --interval=5m --timeout=3s --start-period=90s \
   CMD nc -z 127.0.0.1 8888 || exit 1
