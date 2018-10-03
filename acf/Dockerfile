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

# Create volume folders.
RUN mkdir -p /volume/cert
RUN mkdir -p /volume/certs
RUN mkdir -p /volume/mini_httpd
RUN mkdir -p /volume/private
RUN mkdir -p /volume/req

# set initial acf passwd
# root:alpine
# Store in volume
RUN echo "root:$(mkpasswd -m sha256 alpine):Admin account:ADMIN" > /etc/acf/passwd
RUN mv    /etc/acf/passwd /volume/
RUN ln -s /volume/passwd  /etc/acf/passwd

# Change $dir location to volume.
RUN sed -i -r 's:dir\s+=\s+/etc/ssl:dir = /volume:g' /etc/ssl/openssl-ca-acf.cnf

# Monkeypatch $new_cert_dir
# No setting for `req/` location
# See Alpine Bug #9505
# https://bugs.alpinelinux.org/issues/9505
RUN rm -rf /etc/ssl/cert
RUN rm -rf /etc/ssl/req
RUN ln -s /volume/cert /etc/ssl/cert
RUN ln -s /volume/req  /etc/ssl/req

# Move httpd settings, cert
RUN mv /etc/mini_httpd/mini_httpd.conf    /volume/mini_httpd/
RUN mv /etc/ssl/mini_httpd/server.pem     /volume/mini_httpd/
RUN ln -s -n /volume/mini_httpd/mini_httpd.conf /etc/mini_httpd/mini_httpd.conf
RUN ln -s -n /volume/mini_httpd/server.pem      /etc/ssl/mini_httpd/server.pem

# Move CA settings
# TODO: this does not seem to work.
# acf-openssl does not like symbolic links,
# and hard links break across fs -> volume...
# RUN mv /etc/ssl/openssl.cnf        /volume/openssl.cnf
# RUN mv /etc/ssl/openssl-ca-acf.cnf /volume/openssl-ca-acf.cnf
# RUN mv /etc/ssl/x509v3.cnf         /volume/x509v3.cnf
# RUN ln -s -n /volume/openssl.cnf         /etc/ssl/openssl.cnf
# RUN ln -s -n /volume/openssl-ca-acf.cnf  /etc/ssl/openssl-ca-acf.cnf
# RUN ln -s -n /volume/x509v3.cnf          /etc/ssl/x509v3.cnf

# Export
VOLUME /volume
EXPOSE 443
ENTRYPOINT ["/usr/sbin/mini_httpd", "-D", "-C", "/etc/mini_httpd/mini_httpd.conf"]
