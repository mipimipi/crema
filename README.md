# crema

[Custom repositories](https://wiki.archlinux.org/index.php/Pacman/Tips_and_tricks#Custom_local_repository) are personal [Arch Linux](https://www.archlinux.org/) repositories. They are stored on an FTP server in a local network, for example. crema (**C**ustom **Re**pository  **Ma**nager) helps to manage such repositories. It supports repositories that contain [Meta packages](https://disconnected.systems/blog/archlinux-meta-packages/) and packages from [AUR](https://aur.archlinux.org/), the Arch Linux user repository.

Meta packages (and thus also custom repositories), are a great means to automate the [installation of Arch Linux](https://wiki.archlinux.org/index.php/installation_guide). Get detailed information about how a (more) automated installation of Arch Linux can be achieved [here](docs/automation.md).

## Features

crema supports the following tasks:

* Building meta packages from a [PKGBUILD](https://wiki.archlinux.org/index.php/PKGBUILD) file
* Adding and removing AUR packages
* Updating AUR packages

## Installation

There is an AUR package for crema that can be installed with tools such as [pacaur](https://github.com/E5ten/pacaur) or [trizen](https://github.com/trizen/trizen).

Another option is a manual installation. For this, clone this repository and copy the crema script to a directory of your choice.

## Configuration

The only information crema needs is the name and path of the custom repository. This information needs to be stored in two environment variables:

* `CREMAREPO` (name of the repository)
* `CREMAREPODIR` (path of the repository)

## Usage

cream has sub commands for the different tasks:

* `crema add PACKAGE [PACKAGE] ...`  adds AUR packages incl. corresponding package tarballs to the custom repository
* `crema build` builds packages from a PKGBUILD file and adds them and the corresponding package tarballs to the custom repository. The command must be executed in the directory where the PKGBUILD file is stored.
* `crema env` lists the cream environment variables (i.e. the name and path of the custom repository)
* `crema rm PACKAGE [PACKAGE] ...` removes AUR packages and the corresponding package tarballs from the custom repository.
* `cream update` updates all outdated AUR packages in the custom repository.

## License

[GNU Public License v3.0](https://github.com/mipimipi/crema/blob/master/LICENSE)
