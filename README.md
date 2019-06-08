# crema

[Custom repositories](https://wiki.archlinux.org/index.php/Pacman/Tips_and_tricks#Custom_local_repository) are personal [Arch Linux](https://www.archlinux.org/) repositories. crema (**C**ustom **Re**pository  **Ma**nager) helps to manage such repositories. They can for example contain [Meta packages](docs/meta-packages.md) or packages from [AUR](https://aur.archlinux.org/), the Arch Linux user repository.

Meta packages (and thus also custom repositories), are a great means to automate the [installation of Arch Linux](https://wiki.archlinux.org/index.php/installation_guide). Get detailed information about how a (more) automated installation of Arch Linux can be achieved [here](docs/automation.md).

## Features

crema supports the following tasks:

* Building meta packages from [PKGBUILD](https://wiki.archlinux.org/index.php/PKGBUILD) files
* Adding and removing AUR packages
* Updating AUR packages

## Installation

There is an [AUR package for crema](https://aur.archlinux.org/packages/crema-git/) that can be installed with tools such as [pacaur](https://github.com/E5ten/pacaur) or [trizen](https://github.com/trizen/trizen). The crema script is then stored in `/usr/bin`.

Another option is a manual installation. For this, clone this repository and copy the crema script to a directory of your choice.

## Configuration

crema requires information about the repositories (name and path). This needs to be stored in the configuration file `$XDG_CONFIG_HOME/crema.conf` in the form:

    <repository-name>|<path-to-repository>

[SSH](https://en.wikipedia.org/wiki/Secure_Shell) is supported, i.e. the path can be of the form `<user>@<server>:<path>`. 

## Usage

cream has sub commands for the different tasks:

* `crema add`  adds AUR packages incl. corresponding package tarballs to a custom repository
* `crema build` builds packages from one or multiple PKGBUILD files and adds them and the corresponding package tarballs to a custom repository
* `crema env` lists the repositories and their diretories
* `crema ls` lists all packages of a repository
* `crema rm` removes AUR packages and the corresponding package tarballs from a custom repository.
* `cream update` updates all outdated AUR packages in a custom repository.

## Details

crema is a wrapper around `repo-add`, `repo-remove` (both part of the [pacman](https://wiki.archlinux.org/index.php/Pacman) package), [makepkg](https://wiki.archlinux.org/index.php/Makepkg) and [aurutils](https://github.com/AladW/aurutils).

## License

[GNU Public License v3.0](https://github.com/mipimipi/crema/blob/master/LICENSE)
