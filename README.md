# poppler

Container images with the command-line utilities of [Poppler](https://poppler.freedesktop.org/), the PDF rendering library: `pdftotext`, `pdftoppm`, `pdftocairo`, `pdfimages`, `pdfinfo` and the rest. Poppler is compiled from the signed release tarball on Ubuntu and Alpine, for `linux/amd64` and `linux/arm64`, and the images are rebuilt when Poppler publishes a release and when the base image changes.

This is an unofficial build, not affiliated with or endorsed by the Poppler project. Report problems with the image in this repository and Poppler bugs [upstream](https://gitlab.freedesktop.org/poppler/poppler/-/issues).

## Quick start

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/poppler \
  -layout input.pdf output.txt
```

The same images can also be pulled as `randomcontainers.com/poppler`.

The entrypoint runs `pdftotext` under `tini` in `/work`, so file names are relative to the directory you mount. Give `-` as the output file to print the text instead, and leave out `-layout` to get the text in reading order. The other utilities run with `--entrypoint`:

```sh
# Render every page to a 150 dpi PNG: page-1.png, page-2.png and so on
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint pdftoppm \
  ghcr.io/randomcontainers/poppler -png -r 150 input.pdf page

# Show the page count, page size and metadata
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint pdfinfo \
  ghcr.io/randomcontainers/poppler input.pdf

# Merge PDFs, and split a PDF into one file per page
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint pdfunite \
  ghcr.io/randomcontainers/poppler first.pdf second.pdf merged.pdf
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint pdfseparate \
  ghcr.io/randomcontainers/poppler input.pdf page-%d.pdf

# Extract the images, keeping JPEG and JPEG 2000 images as they are stored
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint pdfimages \
  ghcr.io/randomcontainers/poppler -all input.pdf image

# Convert the first page to SVG with Cairo
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint pdftocairo \
  ghcr.io/randomcontainers/poppler -svg -f 1 -l 1 input.pdf page-1.svg

# Check the digital signatures in a PDF
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint pdfsig \
  ghcr.io/randomcontainers/poppler signed.pdf
```

Each utility lists its options with `-h`.

## What is in the image

- The Poppler utilities in `/usr/local/bin`:
  - `pdftotext`: the text of a PDF, in reading order, in the layout of the page with `-layout`, or as HTML with the position of every word with `-bbox`.
  - `pdftoppm`: pages rendered with Poppler's own rasterizer, as PNG, JPEG, TIFF or PPM.
  - `pdftocairo`: pages rendered with Cairo, as PNG, JPEG, TIFF, SVG, PDF, PostScript or EPS.
  - `pdfimages`: lists or extracts the images in a PDF.
  - `pdfinfo` and `pdffonts`: the document information, metadata and page sizes, and the fonts a PDF uses.
  - `pdftohtml` and `pdftops`: HTML or XML, and PostScript.
  - `pdfseparate` and `pdfunite`: split a PDF into pages, and merge PDFs.
  - `pdfattach` and `pdfdetach`: add a file to a PDF, and list or extract the files in it.
  - `pdfsig`: verifies digital signatures with NSS, and signs PDFs with a certificate from an NSS database (`-add-signature`).
- `libpoppler` in `/usr/local/lib`, which the utilities share. It uses these libraries from the distro: libjpeg-turbo, OpenJPEG for JPEG 2000, libpng, libtiff, Little CMS for color management, FreeType, fontconfig, HarfBuzz, zlib and NSS. `pdftocairo` also uses Cairo.
- poppler-data from the distro, in `/usr/share/poppler`: the CMaps and encoding tables Poppler needs for Chinese, Japanese, Korean and Cyrillic text.
- The Liberation fonts. When a PDF names a font without embedding it, fontconfig picks a replacement. For Helvetica, Times and Courier, and for Arial, Times New Roman and Courier New, that is Liberation Sans, Serif or Mono, which have the same character widths, so the text keeps its layout.

`pdfsig` checks signer certificates against an NSS certificate database, and the image has none. Without one it prints `NSS_Init failed`, still checks that each signature matches the document, and cannot tell whether the signer is trusted. Mount a database and pass it with `-nssdir sql:<directory>`.

Not included: the Qt, GLib and C++ interfaces and the headers for building against `libpoppler`, the GnuPG backend of `pdfsig`, libcurl (the utilities open local files only), fonts for Chinese, Japanese and Korean text that a PDF does not embed (see [Extending the slim image](#extending-the-slim-image)), replacements for the Symbol and ZapfDingbats fonts, and the man pages. On Alpine, `pdftohtml` has no `-dataurls` option: Poppler builds it only against glibc. The CMake options are in `/usr/local/share/randomcontainers/poppler/buildinfo`.

## Default or slim

Poppler's default image adds no other tools, so `latest` and `slim` are the same image, with the contents listed above. Use `latest` to run it and the `slim` tags as a base for your own image. The default image of [Tesseract](https://github.com/randomcontainers/tesseract) includes this build of Poppler, for OCR of PDF files.

## Tags

`<version>` is a Poppler release such as `26.09.0`. `<minor>` and `<major>` are its shorter forms, `26.09` and `26`, and follow the newest release in that series. Poppler numbers its releases by year and month, so `26` is the newest release of 2026. Each row lists the default tag and its `slim` twin, which point to the same image.

| Tags | Base |
|---|---|
| `latest`, `slim` | Ubuntu |
| `<version>`, `<version>-slim` | Ubuntu |
| `<minor>`, `<minor>-slim`, `<major>`, `<major>-slim` | Ubuntu |
| `ubuntu`, `slim-ubuntu` | Ubuntu |
| `<version>-ubuntu`, `<version>-slim-ubuntu` | Ubuntu |
| `<minor>-ubuntu`, `<minor>-slim-ubuntu`, `<major>-ubuntu`, `<major>-slim-ubuntu` | Ubuntu |
| `<version>-ubuntu26.04`, `<version>-slim-ubuntu26.04` | Ubuntu 26.04 |
| `alpine`, `slim-alpine` | Alpine |
| `<version>-alpine`, `<version>-slim-alpine` | Alpine |
| `<minor>-alpine`, `<minor>-slim-alpine`, `<major>-alpine`, `<major>-slim-alpine` | Alpine |
| `<version>-alpine3.24`, `<version>-slim-alpine3.24` | Alpine 3.24 |

The images are currently built on Ubuntu 26.04 and Alpine 3.24. Tags without a distro version move to the next distro release when the project does; tags ending in `ubuntu26.04` or `alpine3.24` stay on that release and are no longer rebuilt once the project moves to the next one. Every tag of the current Poppler version, including the exact version, is rebuilt in place (see [Updates](#updates)), so pin a digest when you need the same bytes every time.

## Platforms

`linux/amd64` and `linux/arm64`, for both Ubuntu and Alpine. Both are compiled natively on GitHub-hosted runners, without emulation.

## Files and permissions

The working directory is `/work`. The image runs as UID 1000, and any other UID works too: `HOME` is then `/`, and caches go to `/cache`, which anyone can write to. How to get output files owned by you depends on how you run containers:

| Runtime | Flag |
|---|---|
| Docker on Linux (rootful), GitHub Actions | `--user "$(id -u):$(id -g)"` |
| Rootless Podman | `--userns=keep-id` |
| Rootless Docker | `--user 0:0` (root in the container is your user on the host) |
| Docker Desktop on macOS or Windows | none, file ownership is mapped for you |

`pdftoppm`, `pdftocairo`, `pdfimages` and `pdfseparate` write one file per page or image, named after the prefix or pattern you give, into the current directory.

## Untrusted files

Poppler parses PDF files in C++, and its releases regularly fix crashes and memory errors caused by malformed files. The utilities do not run JavaScript embedded in a PDF (`pdfinfo -js` only prints it), and this build cannot open URLs. For files from unknown sources, take away what the container does not need:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  --network none --read-only --tmpfs /cache --cap-drop ALL --security-opt no-new-privileges \
  --memory 1g --pids-limit 64 \
  ghcr.io/randomcontainers/poppler untrusted.pdf output.txt
```

Mount only the directory the job needs. fontconfig keeps its cache in `/cache`, which `--tmpfs /cache` leaves writable when the rest of the container is read-only. Fixes to the libraries Poppler uses from the distro, such as libjpeg-turbo, OpenJPEG and FreeType, reach the images through the base image rebuilds described under [Updates](#updates).

## Extending the slim image

Use a `slim` tag as the base for your own image. `slim`, `slim-ubuntu` and `slim-alpine` move to each new Poppler release and are rebuilt when the base image changes. Switch to root to install more, then back. This adds the Noto CJK fonts, which `pdftoppm` and `pdftocairo` need to render Chinese, Japanese and Korean text that a PDF does not embed:

```dockerfile
FROM ghcr.io/randomcontainers/poppler:slim-ubuntu@sha256:...
USER root
RUN apt-get update \
 && apt-get install -y --no-install-recommends fonts-noto-cjk \
 && rm -rf /var/lib/apt/lists/*
USER 1000:1000
```

On Alpine, start from `slim-alpine` and use `apk add --no-cache font-noto-cjk`. Programs that run the Poppler utilities, such as the Python package pdf2image, find them on the `PATH`. The entrypoint is `["tini", "--", "pdftotext"]`; set your own `ENTRYPOINT` if your image runs something else. To pick up new Poppler releases and base image fixes, let Dependabot or Renovate update the digest in your `FROM` line.

Everything the image adds is under `/usr/local`. `/usr/local/share/randomcontainers/poppler/` holds the version, the source URLs, the build options, the license files and `runtime-deps`, the list of distro packages Poppler needs at run time, including fontconfig, the Liberation fonts and poppler-data.

## Verifying

Each image has a build provenance attestation from this repository's GitHub Actions run, signed by the shared build workflow in `randomcontainers/ci`:

```sh
gh attestation verify oci://ghcr.io/randomcontainers/poppler:latest \
  --repo randomcontainers/poppler --signer-repo randomcontainers/ci
```

Each platform image also carries an SPDX SBOM that lists every distro package with its version:

```sh
docker buildx imagetools inspect ghcr.io/randomcontainers/poppler:latest --format '{{ json .SBOM }}'
```

Before compiling, the build checks the tarball against the SHA-256 recorded in `package.yml` and its signature against Albert Astals Cid's Poppler release signing key in `keys/poppler-release.gpg` (fingerprint `CA26 2C6C 83DE 4D2F B28A 332A 3A6A 4DB8 39EA A6D7`), which the Poppler website names.

## Updates

The project reads the [release list](https://poppler.freedesktop.org/releases.html) on the Poppler website every 15 minutes. Poppler publishes a release about once a month. A release is picked up at least a day after the date the list gives for it, once its tarball and signature can be downloaded. The new version and the tarball's SHA-256 are then committed to `package.yml`, and the images are rebuilt. Only the newest release is built; tags of older versions stay as they were last built.

The images of the current version are also rebuilt when the Ubuntu or Alpine base image changes and at least every 7 days, so distro security fixes reach the current tags.

## Building

```sh
docker build -f Dockerfile.ubuntu --target slim \
  --build-arg VERSION=<version> \
  --build-arg SOURCE_SHA256=<sha256 from package.yml> \
  -t poppler:local .
```

Use `Dockerfile.alpine` for the Alpine image. `--build-arg JOBS=<n>` limits the number of parallel compile jobs.

## Licenses

Poppler is licensed under the GNU General Public License, version 2 or version 3 (GPL-2.0-only OR GPL-3.0-only): it is based on Xpdf, which may be distributed under either version, and the changes made in Poppler are licensed under version 2 or later. The UTF-8 decoder in `poppler/UTF.cc` is by Bjoern Hoehrmann and licensed under the MIT license. `COPYING` (GPL-2.0), `COPYING3` (GPL-3.0) and `COPYING.utf8-decoder`, the MIT notice from `poppler/UTF.cc`, are in `/usr/local/share/randomcontainers/poppler/licenses/`. The image's license label is `(GPL-2.0-only OR GPL-3.0-only) AND MIT`.

The Ubuntu and Alpine packages in the image keep their own licenses, and the SBOM lists them. poppler-data has the CMaps under Adobe's BSD-3-Clause license and the other tables under GPL-2.0 or GPL-3.0. The Liberation fonts are under the SIL Open Font License 1.1.

The corresponding source for each image:

- Poppler: every version has a GitHub release in this repository, named `v<version>`, with the exact `poppler-<version>.tar.xz` that was compiled and its signature. The build applies no patches. `/usr/local/share/randomcontainers/poppler/source` lists that release and the poppler.freedesktop.org download URLs.
- Build scripts: this repository at the commit in the image's `org.opencontainers.image.revision` label. The Dockerfiles hold every CMake option.
- Ubuntu packages: the source packages on [Launchpad](https://launchpad.net/ubuntu) for the versions listed in the SBOM. `apt-get source <package>=<version>` fetches a version that is still in the Ubuntu archive.
- Alpine packages: Alpine has no source packages. For the versions listed in the SBOM, the source is the APKBUILD and patches in [aports](https://gitlab.alpinelinux.org/alpine/aports/-/tree/3.24-stable), branch `3.24-stable`, and the archives on [distfiles.alpinelinux.org](https://distfiles.alpinelinux.org/distfiles/v3.24/).

Combined images that include Poppler copy it from a slim Poppler image from this repository. Their `com.randomcontainers.members` label records the Poppler version and the digest of that image, whose own `org.opencontainers.image.revision` label names the commit here.

The files in this repository are available under the MIT license, see [LICENSE](LICENSE).

## Requesting a tool

To suggest another tool, use the [Request a tool](https://github.com/randomcontainers/.github/issues/new?template=tool-request.yml) form.
