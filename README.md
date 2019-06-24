# CAC-protected WordPress Site

Wordpress is included as a git submodule, and the Dockerfile adds MariaDB (mysql) and php.

An existing WordPress database located at /www/database, and SSL certificate(s) under /ssl are assumed (/ssl and /www are currently mounted as docker volumes).

As configured, the database port IS exposed (to the local docker machine network).

NGINX configuration (default.conf) makes /www/WordPress the website root and requires either CAC authentication or a source IP address on the SOFWERX staff or data network. The .TMPL file is for future development that will fill in local configuration values from environment variables.

# From Original README
Upstream source: https://github.com/mpyne-navy/nginx-cac

## Introduction

This is the barest possible NGINX configuration and Docker infrastructure I
could create that would enable developing a Web site that is protected using
client TLS using the DoD public key infrastructure (PKI).

In other words, you can build web sites using NGINX as the SSL terminator that
are CAC-protected starting from this as a baseline.

##Building

You should be on Linux, with the normal command line toolchain (shell,
coreutils, etc.), along with curl and zip.

Of course, you'll need Docker installed as well to actually build the Docker
image and launch new containers.

To build, just run `make`.

This will:

* Download the base Docker image (alpine),
* Generate a new self-signed SSL cert,
* Download the DoD root certs and wrap them into a single file (to serve as the
  client TLS trusted CA in NGINX),
* Install the new client CA and new cert into the Docker image,
* Install nginx and a simple config into the Docker image, along with a sample
  index.html.

By default the new Docker image is called "nginx-cac".

## Launching

#From there you can run it by using `make run`.

This will start a new container based on the image built, expose port 443
(inside the container) to a random port on the host.

Use `docker ps` to ensure the new container is actually running. If so, Docker
will tell you which port on the host corresponds to port 443 in the guest.

## Testing

In my case I had Firefox already configured to be able to authenticate against
CAC-enabled websites, using PCSC Lite, the CACKey middleware, and by installing
the DoD root certs **and intermediate CA certs** into the NSS keystore.

With "real" CAC sites already working, testing for me as a matter of going to
`https://localhost:$PORT/` (replacing `$PORT` with the actual port on the host
mapped to the guest's port 443).

I believe at this point it should already ask for the CAC PIN (as part of SSL
mutual auth) and then show an error page that the site is untrusted.

After confirming a security exception for the NGINX server's self-signed
certificate (I made it a temporary exception but permanent should work as
well), Firefox reloaded the page and this time you should see "It works!".

If you look into the Developer Tools you should also see that NGINX has sent
back your Subject Name information as a server response header.

### Windows Testing

I've also tested exporting the created Docker image (using
`docker save nginx-cac:latest | gzip > nginx-cac.tar.gz`
on the Linux host and using `docker load` on a Win10 machine to import the
image).

The container ran fine on Windows and I was also able to test with Firefox,
Chrome, and Microsoft Edge (with success on all 3).

As with Linux, you need to make sure you've setup your smartcard infrastructure
(in my case I used OpenSC with Firefox) and that you've installed the DoD root
and intermediate certificates into each browser.

## Shutdown

Don't forget to shutdown the Docker container when you're done (Use `docker ps`
to find the short name of the running container and then `docker stop
$short-name` from there).

## Why?????

Because doing this all within DoD is so much harder than doing it in my off
time. :(
