FROM alpine:3.8

# install acf + deps
RUN apk update \
    && apk upgrade \
    && apk add alpine-conf acf-core acf-alpine-baselayout acf-openssl

# setup acf
RUN setup-acf

# Not using openrc
RUN poweroff
RUN rc-update del mini_httpd default

# set initial acf passwd
# root:alpine
RUN echo "root:$(mkpasswd -m sha256 alpine):Admin account:ADMIN" > /etc/acf/passwd

# exports
VOLUME /etc/ssl/
EXPOSE 443
ENTRYPOINT ["/usr/sbin/mini_httpd", "-D", "-C", "/etc/mini_httpd/mini_httpd.conf"]
