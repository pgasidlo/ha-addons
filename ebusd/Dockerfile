ARG BUILD_FROM
FROM $BUILD_FROM

ENV LANG C.UTF-8

RUN echo "https://dl-cdn.alpinelinux.org/alpine/edge/community" \
    >> /etc/apk/repositories
RUN echo "https://dl-cdn.alpinelinux.org/alpine/edge/main" \
    >> /etc/apk/repositories

RUN adduser -G abuild -g "Alpine Package Builder" -s /bin/sh -D builder
RUN echo "builder ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

#RUN apk add --no-cache ebusd=24.1
RUN apk add alpine-sdk
RUN apk add abuild-rootbld
RUN su -l builder
RUN abuild alpine

LABEL Description="ebusd"

COPY run.sh /
RUN chmod a+x /run.sh
CMD [ "/run.sh" ]

# Health check
HEALTHCHECK --interval=5m --timeout=3s --start-period=90s \
   CMD nc -z 127.0.0.1 8888 || exit 1
