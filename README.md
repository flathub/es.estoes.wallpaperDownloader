# Wallpaper Downloader

This repository contains all the files necessary to create a Flatpak build of Wallpaper Downloader, a cross-platform Java program that manages and downloads wallpapers from various sites, including: [DeviantArt](https://www.deviantart.com/), [Wallhaven](https://wallhaven.cc/), [Bing Daily Wallpaper](https://bing.wallpaper.pics/), [Social Wallpapering](https://www.socwall.com/), [WallpaperFusion](https://www.wallpaperfusion.com/), [Dual Monitor Backgrounds](https://www.dualmonitorbackgrounds.com/). 

The main repository is [here](https://bitbucket.org/eloy_garcia_pca/wallpaperdownloader).

## Installation

It's recommended to install Wallpaper Downloader via [Flathub](https://flathub.org/apps/details/es.estoes.wallpaperDownloader). But you can keep reading if you want to build it yourself.

## Build instructions

**Beware:** These instructions were only tested on Ubuntu 18.04 LTS.

First, clone this repository:

```bash
git clone https://github.com/flathub/es.estoes.wallpaperDownloader.git
cd es.estoes.wallpaperDownloader/
```

Then, add the [official Flatpak PPA](https://flatpak.org/setup/Ubuntu/), in order to get the most recent version of Flatpak and its tools:

```bash
sudo add-apt-repository ppa:alexlarsson/flatpak
sudo apt update
```

Now, install `flatpak` and `flatpak-builder`:

```bash
sudo apt install flatpak flatpak-builder
```

Next, add the official FreeDesktop SDK remote to Flatpak. It will be installed for the current user only:

```bash
flatpak --user remote-add --if-not-exists freedesktop-sdk https://cache.sdk.freedesktop.org/freedesktop-sdk.flatpakrepo
```

**Beware:** If you're running this next command for the first time, Flatpak will download about 1.2 GiB of files!

Now, we'll use `flatpak-builder` to build Wallpaper Downloader:

```bash
flatpak-builder --user --force-clean --install-deps-from=freedesktop-sdk --repo=repo/ --sandbox build es.estoes.wallpaperDownloader.yml
```

Running the command above, will:

* Automatically download and install the required software to build and run Wallpaper Downloader.
* Download the source code of Wallpaper Downloader from the [official repository](https://bitbucket.org/eloy_garcia_pca/wallpaperdownloader).
* Download Maven's build dependencies.
* Create a clean Flatpak build of Wallpaper Downloader.
* ... And finally, if the build succeeds, it will be exported to a [local Flatpak repository](https://docs.flatpak.org/en/latest/flatpak-builder.html#exporting-to-a-repository), which can later be used to install it or package it as a [single-file bundle](https://docs.flatpak.org/en/latest/single-file-bundles.html), used mostly for software distribution.

For more technical details, you can simply read the `es.estoes.wallpaperDownloader.yml` file in this repository. And for more details on Flatpak manifests in general, [check out this page](https://docs.flatpak.org/en/latest/manifests.html).

The building process should not give you any errors. If you get any, please [open a new issue](https://github.com/flathub/es.estoes.wallpaperDownloader/issues).

Anyway, now that the Flatpak build is finished, you can try to run Wallpaper Downloader without having to actually install it. However, this method is recommended for testing only:

```bash
flatpak-builder --run build-dir/ es.estoes.wallpaperDownloader.yml es.estoes.wallpaperDownloader.sh
```

### Cleaning up

To remove the FreeDesktop SDK repository:

```bash
flatpak --user remote-delete freedesktop-sdk
```

To uninstall unused Flatpak runtimes/SDKs (**Note:** Inspect the packages before removing them):

```bash
flatpak --user uninstall --unused
```

After that, just delete this repository's folder.

### Miscellaneous

#### Creating a single-file bundle

Finish the build process, as mentioned above. Then, create a single-bundle file named `WallpaperDownloader.flatpak` using this command:

```bash
flatpak build-bundle repo/ WallpaperDownloader.flatpak es.estoes.wallpaperDownloader master
```

You can now install or distribute this single-file bundle, but keep in mind that auto-updates won't work, obviously.

To install it:

```bash
flatpak --user install --reinstall WallpaperDownloader.flatpak
```

#### Generating or updating maven-dependencies.yml

I'm currently using a very hacky and ugly method to generate the `maven-dependencies.yml` file, which is required if:

* You're using the `--sandbox` option in `flatpak-builder` (which Flathub does, by default).
* You want to build a Java program entirely from source, as opposed to packaging a pre-compiled `.jar` file.

To generate this file, however, we will have to modify the manifest file in this repository first.

So, open `es.estoes.wallpaperDownloader.yml` and add the following key under `build-options:`:

```yaml
build-args:
  - --share=network
```

Additionally, **delete** the following line, which can be found under the `sources:` key:

```yaml
- maven-dependencies.yml
```

Then, delete the existing build directory (if any):

```bash
rm -rf .flatpak-builder/
```

Now, build it:

```bash
flatpak-builder --user --build-only --force-clean --keep-build-dirs build es.estoes.wallpaperDownloader.yml
```

The build should succeed.

Now, to finally generate the `maven-dependencies.yml` file, we will use an awesome program called [fd](https://github.com/sharkdp/fd), which is basically a faster alternative to this other awesome program, called `find`.

Once `fd` is installed, run this command:

```bash
(cd .flatpak-builder/build/wallpaperdownloader-1/.m2/repository/ && fd -j 1 -t f '\.(jar|pom)$' -x bash -c 'echo -e "- type: file\n  dest: .m2/repository/{//}\n  url: https://repo.maven.apache.org/maven2/{}\n  sha1: $(sha1sum {} | cut -c 1-40)"') > maven-dependencies.yml
```

To test if this generated file actually works, just **revert** the changes you did to `es.estoes.wallpaperDownloader.yml` in the previous steps.

Then, try building Wallpaper Downloader with the `--sandbox` option:

```bash
flatpak-builder --user --force-clean --sandbox build es.estoes.wallpaperDownloader.yml
```

### Bugs

Any bugs in Wallpaper Downloader itself (that is, **not related** to the build process), should be reported to the [main repository](https://bitbucket.org/eloy_garcia_pca/wallpaperdownloader) instead.
