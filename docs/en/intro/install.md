# RoadRunner — Installation

There are several ways to install RoadRunner, depending on your needs and preferences.

## Pre-built Binaries

The simplest way to get the latest version of RoadRunner is to download one of the pre-built release binaries, which are
available for various operating systems, including macOS, Linux, FreeBSD, and Windows. You can find these binaries on
the GitHub [releases page](https://github.com/roadrunner-server/roadrunner/releases).

To install RoadRunner, just download the appropriate archive from the releases page and extract it into your desired
application directory.

## Docker

If you prefer to use RoadRunner inside a Docker container, you can use the official RoadRunner Docker
image `ghcr.io/roadrunner-server/roadrunner:latest`.

> **Note**
> More information about available tags can be
> found [here](https://github.com/roadrunner-server/roadrunner/pkgs/container/roadrunner).

**Here is an example of usage**

```dockerfile
FROM ghcr.io/roadrunner-server/roadrunner:2023.X.X AS roadrunner
FROM php:8.x-cli

COPY --from=roadrunner /usr/bin/rr /usr/local/bin/rr

# Install and configure your application
# ...

CMD rr serve -c .rr.yaml
```

> **Warning**
> Don't forget to replace `2023.X.X` with a desired version of RoadRunner.

## Composer

If you use Composer to manage your PHP dependencies, you can install the `spiral/roadrunner-cli` package to download the
latest version of RoadRunner to your project's root directory.

**Install the package**

```terminal
composer require spiral/roadrunner-cli
```

Run the following command to download the latest version of RoadRunner

```terminal
./vendor/bin/rr get-binary
```

Server binary will be available at the root of your project.

> **Warning**
> PHP's extensions `php-curl` and `php-zip` are required to download RoadRunner automatically.
> PHP's extensions `php-sockets` need to be installed to run roadrunner.
> Check with `php --modules` your installed extensions.

## Debian Package

For Debian-based operating systems such as **Ubuntu**, **Mint**, and **MX**, you can download the `.deb` package from
the RoadRunner GitHub releases page and install it using dpkg.

**Just run the following commands**

```bash
wget https://github.com/roadrunner-server/roadrunner/releases/download/v2023.X.X/roadrunner-2023.X.X-linux-amd64.deb
sudo dpkg -i roadrunner-2023.X.X-linux-amd64.deb
```

> **Warning**
> Don't forget to replace `2023.X.X` with a desired version of RoadRunner.

## MacOS package using [Homebrew](https://brew.sh/):
```terminal
brew install roadrunner
```

## CURL

You can also install RoadRunner using curl and the download-latest.sh script from the RoadRunner GitHub repository.

**Just run the following commands**

```bash
curl --proto '=https' --tlsv1.2 -sSf  https://raw.githubusercontent.com/roadrunner-server/roadrunner/master/download-latest.sh | sh
```

## What's Next?

After you have installed RoadRunner, you can proceed to the next steps and configure it for your needs.

1. [RoadRunner — Configuration](./config.md).
2. [Developer Mode](../php/developer.md).