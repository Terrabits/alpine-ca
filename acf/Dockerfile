FROM alpine:3.8

# root passwd
# RUN echo "root:$(mkpasswd -m sha512 alpine)" | chpasswd --crypt-method sha512

# install acf + deps
RUN apk update \
    && apk upgrade \
    && apk add alpine-conf acf-core acf-alpine-baselayout acf-openssl

# setup acf
RUN setup-acf
RUN poweroff
RUN rc-update del mini_httpd default

# update root passwd
# RUN acfpasswd -s root
RUN echo "root:$(mkpasswd -m sha256 alpine):Admin account:ADMIN" > /etc/acf/passwd

# setup s6
ADD https://github.com/just-containers/s6-overlay/releases/download/v1.21.7.0/s6-overlay-amd64.tar.gz /tmp/
RUN tar xzf /tmp/s6-overlay-amd64.tar.gz -C /

# Create service
RUN mkdir -p /etc/services.d/mini_httpd
ADD run /etc/services.d/mini_httpd/

VOLUME /etc/ssl/
EXPOSE 443
ENTRYPOINT ["/init"]
