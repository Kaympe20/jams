---
title: 'Turquoise'
description: 'You ship a linux distro, we ship a HC skirt (or pants) and a DVD with your ISO'
contributor: 'Kaympe20'
contributorSlackId: 'U07HY92M9GA'
thumbnail: 'https://wallpapers.com/images/featured-full/linux-desktop-nf65sk0rdgsvfl3u.jpg'
timeEstimate: '60 Min'
difficulty: 'Intermediate'
keywords: 'turquoise, help me, oh dear god, help me please, linux, distro, ysws, you ship we ship'
presentation: ''
presentationPlay: ''
presentationPDF: ''
notes: ''
poster: ''
video: ''
slug: 'turquoise'
---
# Turquoise

<sub><sup> Much appreciation goes out to the [blue-build community](https://discord.gg/FuZkAmWZA8) as of this guide is made using the [blue-build official docs](https://blue-build.org/learn/getting-started/). </sup></sub>

## Preface
There are many ways to create a Linux distro. Many people will end up deciding to use [LinuxFromScratch](https://www.linuxfromscratch.org/), [NixOS](https://nixos.org/), or [Cubic](https://github.com/PJ-Singh-001/Cubic). This guide will go over creating a Fedora Silverblue derivative using [blue-build](https://blue-build.org/).
If at any point you get stuck, please use the [blue-build docs](https://blue-build.org/learn/getting-started/). If you are still stuck, feel free to ask in [#turquoise](https://hackclub.slack.com/archives/C08BAEP4SCX) on the [Hack Club Slack](https://hackclub.com/slack)

For this guide, we are making an immutable distro. That means that the user's filesystem layers on our underlying image without actually replacing anything. This approach is the future of Linux and comes with many benefits such as how the whole system is updated in one go, and an update will not apply if anything goes wrong, meaning you will always have a working computer. This will be important to understand

This guide assumes you know some basic linux terminology. Please use the following list and research what each word you don't know means (if any):
* kernel
* package manager
* distro
* flatpak
* bash

## Development Environment Setup

### Automatic setup using the BlueBuild Workshop

The automatic setup will apply some default configuration and set up cosign for you.

The Workshop is a currently work-in-progress part of BlueBuild that allows managing custom image repositories using a web interface. Report issues in the GitHub repository: [blue-build/workshop](https://github.com/blue-build/workshop).

1. Open https://workshop.blue-build.org/.
2. Press "Log in with GitHub".

![](https://hc-cdn.hel1.your-objectstorage.com/s/v3/d4310301408d5e031b845376c6932eae70f494ca_image.png)

3. Press "New custom image repository".

![](https://hc-cdn.hel1.your-objectstorage.com/s/v3/9fbab1c5372ec74101e8247c051573c63f314482_image.png)

4. Choose a name for your repository and press "Create repository".
    - This name should not contain spaces, slashes, or other special non-ascii characters. Using a dash (`-`) for delimitating words is recommended.

![](https://hc-cdn.hel1.your-objectstorage.com/s/v3/8ffb79c8c82f61f52a4c59a3cf714738615e62c0_image.png)

5. Wait for the repository to be set up

![](https://hc-cdn.hel1.your-objectstorage.com/s/v3/58aa8f7af07009c1d7bfa27ab9f652f35cd81fcd_image.png)

6. When prompted with setting up container signing, it is recommended to do it **automatically**.
    - If you have security concerns, skipping is an option too. Read more about this in the wizard.

![](https://hc-cdn.hel1.your-objectstorage.com/s/v3/e5fe5194fd1ce3392cdd8e2d1b4f9e463afbdd17_image.png)

7. After that step is completed, you're done! You can now clone your repository, open `recipes/recipe.yml` and start customizing! The [reference section](https://blue-build.org/reference/recipe/) of the documentation is your friend here.

Below is the default recipe:

```yaml 
---
# yaml-language-server: $schema=https://schema.blue-build.org/recipe-v1.json
# image will be published to ghcr.io/<user>/<name>
name: KaylOS
# description will be included in the image's metadata
description: This is my personal OS image.

# the base image to build on top of (FROM) and the version tag to use
base-image: ghcr.io/ublue-os/silverblue-main
image-version: 41 # latest is also supported if you want new updates ASAP

# module configuration, executed in order
# you can include multiple instances of the same module
modules:
  - type: files
    files:
      - source: system
        destination: / # copies files/system/* (* means everything inside it) into your image's root folder /

  - type: rpm-ostree
    repos:
      - https://copr.fedorainfracloud.org/coprs/atim/starship/repo/fedora-%OS_VERSION%/atim-starship-fedora-%OS_VERSION%.repo
    install:
      - micro
      - starship
    remove:
      # example: removing firefox (in favor of the flatpak)
      # "firefox" is the main package, "firefox-langpacks" is a dependency
      - firefox
      - firefox-langpacks # also remove firefox dependency (not required for all packages, this is a special case)

  - type: default-flatpaks
    notify: true # Send notification after install/uninstall is finished (true/false)
    system:
      # If no repo information is specified, Flathub will be used by default
      install:
        - org.mozilla.firefox
        - org.gnome.Loupe
      remove:
        - org.gnome.eog
    user: {} # Also add Flathub user repo, but no user packages

  - type: signing # this sets up the proper policy & signing files for signed images to work fully

```

## Preinstalled Applications
### `rpm-ostree` ([reference](https://blue-build.org/reference/modules/rpm-ostree/))

`rpm-ostree` is the system package manager of all Fedora immutable distros and derivatives. It is very comparable to the non-immutable variant `dnf`. It requires a reboot and, unlike most other package managers we will cover, has the potential to make your system unstable if used with the wrong packages. For this reason, we recomend you use it ***sparingly***.

To use it, you have 6 things you can add to the `rpm-ostree:` configuration under `modules` in `recipe.yml`:
* **[`install:`](https://blue-build.org/reference/modules/rpm-ostree/#install-optional-array) for applications that you want to install**
* **[`remove:`](https://blue-build.org/reference/modules/rpm-ostree/#remove-optional-array) for applications on the default image you want to uninstall**
* **[`replace:`](https://blue-build.org/reference/modules/rpm-ostree/#install-optional-array) for applications on the default image you would like to replace**
* [`repos:`](https://blue-build.org/reference/modules/rpm-ostree/#repos-optional-array) for specifiying CoPr or third party repositories
* [`keys:`](https://blue-build.org/reference/modules/rpm-ostree/#keys-optional-array) for importing GPG keys for those repositories
* [`optfix:`](https://blue-build.org/reference/modules/rpm-ostree/#optfix-optional-array) for a list of directories of applications that need to install in the `/opt` directory

Here I want to install vscode from the Microsoft repos. To do that I would add:
```yaml
- https://packages.microsoft.com/yumrepos/vscode/config.repo
```
to `repos:`
```yaml
- https://packages.microsoft.com/keys/microsoft.asc
```
to `keys:`

and
```yaml
- code
```
to `install:`

This would give me a complete `rpm-ostree` section of
```yaml
- type: rpm-ostree
    repos:
      - https://copr.fedorainfracloud.org/coprs/atim/starship/repo/fedora-%OS_VERSION%/atim-starship-fedora-%OS_VERSION%.repo
      - https://packages.microsoft.com/yumrepos/vscode/config.repo
    keys:
      - https://packages.microsoft.com/keys/microsoft.asc
    install:
      - micro
      - starship
      - code
    remove:
      # example: removing firefox (in favor of the flatpak)
      # "firefox" is the main package, "firefox-langpacks" is a dependency
      - firefox
      - firefox-langpacks
```