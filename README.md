# Certificate Authority

I wanted an easy way to create a certificate authority / manage certs when needed. This solution uses Docker, Alpine linux's Configuration Framework (ACF) and the acf-openssl plugin to create a personal Certificate Authority (CA).

## Run

First build the docker container from source:

`docker build -t acf acf/`

Then run the image. The web service is exposed via HTTPS on Port 443.

`docker run -d -p443:443 acf`

## Docker Compose

There is also an included docker compose file to make manually launching this trivial. Bring up the CA as usual:

`docker-compose up`

## Site Security

The default server certificate for HTTPS is self-signed, so you will probably get a warning about the site being unsafe and be asked to accept this responsibility to visit the site anyway.

Of course, once your CA is up and running you can use it to sign your site and set up your CA as a trusted source. ;-)

## Initial Login

First login uses the credentials `root:alpine`. This can be changed. Unfortunately the `/etc/acf/passwd` file is not mounted into a volume right now, so these changes stay in the container. Grouping and volumizing the appropriate files is a work in progress.

## Managing ACF

Once the site is up, use the ACF interface as usual. More information can be found with the Alpine documentation.

[Managing ACF](https://wiki.alpinelinux.org/wiki/Managing_ACF)

## Setting up a CA

Get started generating CA/certs with this reference.

[Generating SSL certs with ACF 1.9](https://wiki.alpinelinux.org/wiki/Generating_SSL_certs_with_ACF_1.9#Configure)

## Persistent Data

A volume (named `certs` in docker compose) is mounted to `/etc/ssl`. This is the directory that contains all the certificates and some of the server settings.

| File                   | Location                             |
| ---------------------- | ------------------------------------ |
| Site certificate       | `/etc/ssl/mini_httpd/server.pem`     |
| Site settings          | `/etc/ssl/mini_httpd/mini_httpd.cnf` |
| CA certificate         | `/etc/ssl/cacert.pem`                |
| CA key                 | `/etc/ssl/private/cakey.pem`         |
| Generated certificates | `/etc/ssl/cert`                      |
| Generated keys         | `/etc/ssl/certs`                     |
| CA settings            | `/etc/ssl/openssl-ca-acf.cnf`        |
| Certificate template   | `/etc/ssl/openssl.cnf`               |
| x509v3 settings        | `/etc/ssl/x509v3.cnf`                |

All certificates and keys except the CA key can be downloaded from the ACF site as an end-user. You should not need to dig through the `cert(s)/` directories.

As mentioned above, the usernames and passwords for the ACF site itself, which are located at `/etc/acf/passwd`, are not captured in this volume. This is a work in progress.

## Exporting Data

All certificates, keys and OpenSSL ACF site settings can be exported by mounting to `/etc/ssl` and copying everything out. Since this container is built on Alpine 3.8, it is convenient to use this to do so.

Assuming the container is running (name: `alpine-ca_acf_1`), the syntax would be as follows.

```shell
sudo docker cp alpine-ca_acf_1:/etc/ssl ./export
sudo chown -R $(whoami):$(id -g) ./export
sudo chmod -R u+rwx ./export
```

`sudo`, `chown` and `chmod` are necessary because of permissions issues with files created by docker on the local file system. The last two commands make you the owner with full permissions.
