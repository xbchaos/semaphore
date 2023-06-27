# ansible-semaphore production image
FROM golang:1.18.3-alpine3.16 as builder

# replace the default repository address of alpine, if specified
ARG ALPINE_REPOSITORY_MIRROR
RUN if [ "$ALPINE_REPOSITORY_MIRROR" != "" ]; then sed -i "s/dl-cdn.alpinelinux.org/$ALPINE_REPOSITORY_MIRROR/g" /etc/apk/repositories; fi

# set the GOPROXY environment, if specified
ARG GOPROXY
RUN if [ "$GOPROXY" != "" ]; then go env -w GO111MODULE=on GOPROXY=$GOPROXY; fi

COPY ./ /go/src/github.com/ansible-semaphore/semaphore
WORKDIR /go/src/github.com/ansible-semaphore/semaphore

RUN apk add --no-cache -U libc-dev curl nodejs npm git gcc g++ && \
  ./deployment/docker/prod/bin/install

FROM alpine:3.16 as runner
LABEL maintainer="Tom Whiston <tom.whiston@gmail.com>"

# again, replace the default repository address of alpine, if specified
ARG ALPINE_REPOSITORY_MIRROR
RUN if [ "$ALPINE_REPOSITORY_MIRROR" != "" ]; then sed -i "s/dl-cdn.alpinelinux.org/$ALPINE_REPOSITORY_MIRROR/g" /etc/apk/repositories; fi

RUN apk add --no-cache sshpass git curl ansible mysql-client openssh-client-default tini py3-aiohttp && \
    adduser -D -u 1001 -G root semaphore && \
    mkdir -p /tmp/semaphore && \
    mkdir -p /etc/semaphore && \
    mkdir -p /var/lib/semaphore && \
    chown -R semaphore:0 /tmp/semaphore && \
    chown -R semaphore:0 /etc/semaphore && \
    chown -R semaphore:0 /var/lib/semaphore

COPY --from=builder /usr/local/bin/semaphore-wrapper /usr/local/bin/
COPY --from=builder /usr/local/bin/semaphore /usr/local/bin/

RUN chown -R semaphore:0 /usr/local/bin/semaphore-wrapper &&\
    chown -R semaphore:0 /usr/local/bin/semaphore

WORKDIR /home/semaphore
USER 1001

ENTRYPOINT ["/sbin/tini", "--"]
CMD ["/usr/local/bin/semaphore-wrapper", "/usr/local/bin/semaphore", "server", "--config", "/etc/semaphore/config.json"]
