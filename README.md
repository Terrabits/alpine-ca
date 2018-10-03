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

The default server certificate can be replaced (see Persistent Data below).

## Initial Login

First login uses the credentials `root:alpine`. Users and passwords are persistent data, and can be changed (see below).

## Managing ACF

Once the site is up, use the ACF interface as usual. More information can be found with the Alpine documentation.

[Managing ACF](https://wiki.alpinelinux.org/wiki/Managing_ACF)

## Setting up a CA

Get started generating CA/certs with this reference.

[Generating SSL certs with ACF 1.9](https://wiki.alpinelinux.org/wiki/Generating_SSL_certs_with_ACF_1.9#Configure)

## Persistent Data

A volume (named `-certs` in docker compose) is mounted to `/volume`. This is the directory that contains all the certificates and some of the server settings.

| File                   | Location                            |
| ---------------------- | ----------------------------------- |
| ACF users              | `/volume/passwd`                    |
| Site certificate       | `/volume/mini_httpd/server.pem`     |
| Site settings          | `/volume/mini_httpd/mini_httpd.cnf` |
| CA certificate         | `/volume/cacert.pem`                |
| CA key                 | `/volume/private/cakey.pem`         |
| Generated certificates | `/volume/cert`                      |
| Generated keys         | `/volume/certs`                     |
| CA settings            | `/etc/ssl/openssl-ca-acf.cnf`       |
| Certificate template   | `/etc/ssl/openssl.cnf`              |
| x509v3 settings        | `/etc/ssl/x509v3.cnf`               |

All certificates and keys except the CA key can be downloaded from the ACF site as an end-user. You should not need to dig through the `cert(s)/` directories.

Unfortunately `acf`/`acf-openssl` fails to handle symbolic links correctly, so the `openssl` settings are not persisted.

## Exporting Data to Host

Assuming the container is running (name: `alpine-ca_acf_1`), the syntax is as follows.

```shell
sudo docker cp alpine-ca_acf_1:/volume ./data
sudo chown -R $(whoami):$(id -g) ./export
sudo chmod -R u+rwx ./export
```

`sudo`, `chown` and `chmod` are necessary because of permissions issues with files created by docker on the local file system. The last two commands make you the owner with full permissions.
