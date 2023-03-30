# Installation

The easiest way to get the latest RoadRunner version is to use one of the pre-built release binaries, which are available for
OSX, Linux, FreeBSD, and Windows. Instructions for using these binaries are on the GitHub [releases page](https://github.com/roadrunner-server/roadrunner/releases).

## Docker:

To get the roadrunner binary file you can use our docker image: `ghcr.io/roadrunner-server/roadrunner:2023.X.X` (more information about
image and tags can be found [here](https://github.com/roadrunner-server/roadrunner/pkgs/container/roadrunner)).

```dockerfile
FROM ghcr.io/roadrunner-server/roadrunner:2023.X.X AS roadrunner
FROM php:8.2-cli

COPY --from=roadrunner /usr/bin/rr /usr/local/bin/rr

# USE THE RR
```

Configuration located in the `.rr.yaml` file ([full sample](https://github.com/roadrunner-server/roadrunner/blob/master/.rr.yaml)):


## Installation via Composer
You can also install RoadRunner automatically using command shipped with the composer package, run:

```bash
composer require spiral/roadrunner:v2.0 nyholm/psr7
./vendor/bin/rr get-binary
```

Server binary will be available at the root of your project.

> **INFO**
> PHP's extensions `php-curl` and `php-zip` are required to download RoadRunner automatically.
> PHP's extensions `php-sockets` need to be installed to run roadrunner.
> Check with `php --modules` your installed extensions.


## Installation option for the Debian-derivatives (Ubuntu, Mint, MX, etc)

```bash
wget https://github.com/roadrunner-server/roadrunner/releases/download/v2023.X.X/roadrunner-2023.X.X-linux-amd64.deb
sudo dpkg -i roadrunner-2023.X.X-linux-amd64.deb
```

## Dowload the latest release via curl:
```bash
curl --proto '=https' --tlsv1.2 -sSf  https://raw.githubusercontent.com/roadrunner-server/roadrunner/master/download-latest.sh | sh
```
