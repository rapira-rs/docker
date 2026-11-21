# PHP images

This repository builds the PHP images `ghcr.io/rapira-rs/php`.

## Images

- PHP 8.4, 8.5 and 8.6.
- Suites: `trixie` and `bookworm` (Debian), `alpine3.24` and `alpine3.23` (Alpine).
- Variants: `nts` (not thread-safe) and `zts` (thread-safe). Both are the cli build of the official image, with `libphp.so`.
- Platforms: `linux/amd64` and `linux/arm64`.

| Tag | Example | Image |
|---|---|---|
| `<X.Y.Z>-<suite>-<variant>` | `8.5.13-alpine3.24-nts` | one patch release |
| `<X.Y>-<suite>-<variant>` | `8.5-alpine3.24-nts` | the newest patch release of the minor |
| `<X.Y.Z>-<variant>` and `<X.Y>-<variant>` | `8.5-nts` | the same as the `trixie` tags |

The images have the PHP extensions of the official image and its helper commands: `docker-php-ext-install`, `docker-php-ext-enable`, `docker-php-ext-configure` and `docker-php-source`. Add extensions with these commands:

```dockerfile
FROM ghcr.io/rapira-rs/php:8.5-nts
RUN docker-php-ext-install bcmath
```

The Rapira image `ghcr.io/rapira-rs/rapira` uses the `nts` trixie image and adds its own extensions.

## Differences from the official image

- PHP comes from the signed php-src tag `php-X.Y.Z`. The build verifies the tag with the keys of the release managers of the minor and makes the source tarball with `scripts/dev/makedist`.
- Each minor has an `nts` and a `zts` image. The official 8.6 images are ZTS only.
- Each image has `libphp.so`.
- The build installs the newest packages of the suite.

## Updates

`versions.json` holds the PHP version of each minor. Renovate checks the php-src tags every 15 minutes, with the config in `renovate.json`. For a new patch tag `php-X.Y.Z`, it opens a pull request that changes the version of the minor, and merges it when the `Verify Templating` check passes.

The `PHP images` workflow builds the images of a minor when its version changes on `main`. It builds the images of all minors when a generated Dockerfile or a `docker-php-*` script changes. A weekly `workflow_dispatch` with `force` builds all images again, so that they get the newest packages of the suite.

Renovate and the weekly dispatch run as CronJobs in a Kubernetes cluster of the owner and sign in as the GitHub App `rapira-renovate`.

## Change the Dockerfiles

The Dockerfiles in `<minor>/<suite>/<variant>/` are generated. Do not edit them.

1. Edit `Dockerfile-linux.template` or `versions.json`.
2. Run `./apply-templates.sh`. It needs bash, gawk, jq and wget.
3. Commit the template, `versions.json` and the generated files together. The `Verify Templating` workflow fails when they differ.

`versions.json` lists the minors, the PHP version of each minor and its variants. A new minor also needs the keys of its release managers in the `GPG_KEYS` block of the template.

Build one image locally from the repository root:

```sh
docker build -f 8.5/alpine3.24/nts/Dockerfile --build-arg PHP_VERSION=8.5.13 .
```

`PHP_VERSION` must be 8.4.28, 8.5.13, 8.6.0 or a newer patch release of the minor. For older tags, `makedist` downloads the PEAR installer with wget, and the source stage has no wget.

The template is based on `Dockerfile-linux.template` of [docker-library/php](https://github.com/docker-library/php) at commit `3b25f4162a9bd144d3f447d03cb8fe16c0d2c9cc`. Upstream changes are copied by hand.

## License

MIT, see `LICENSE`. The template, `apply-templates.sh` and the `docker-php-*` scripts come from docker-library/php under the MIT license, Copyright (c) 2014 Docker, Inc., see `LICENSE.docker-library`.
