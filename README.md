# crema
Manage your custom [Arch Linux](https://www.archlinux.org/) repositories

## Description

[Custom repositories](https://wiki.archlinux.org/index.php/Pacman/Tips_and_tricks#Custom_local_repository) are personal repositories. They are stored on an FTP server in a local network, for example. crema (__C__ustom __Re__pository  __Ma__nager) helps to manage such repositories. It supports custom repositories that contain [Meta packages](https://disconnected.systems/blog/archlinux-meta-packages/) and packages from [AUR](https://aur.archlinux.org/), the Arch Linux user repository.

## Features

cream supports the following tasks:

* Building meta packages from a [PKGBUILD](https://wiki.archlinux.org/index.php/PKGBUILD) file
* Adding and removing AUR packages
* Updating AUR packages

## Installation

### Arch Linux et al.

For Arch Linux and Linux distributions that are based on Arch, there is an AUR package for crema that can be installed with tools such as [pacaur](https://github.com/E5ten/pacaur) or [trizen](https://github.com/trizen/trizen).

### Manual installation

Clone this repository and copy the crema script to a directory of your choice.

## Configuration

The only information crema needs is the name and path of the custom repository. This information needs to be stored in two environment variables:

* `CREMAREPO` (name of the repository)
* `CREMAREPODIR` (path of the repository)

## Usage

cream has sub commands for the different tasks:

* `crema add PACKAGE [PACKAGE] ...` 
  Adds AUR packages incl. corresponding package tarballs to the custom repository
* `crema build`
  Builds packages from a PKGBUILD file and adds them and the corresponding package tarballs to the custom repository. The command must be executed in the directory where the PKGBUILD file is stored.
* `crema env`
  Lists the cream environment variables (i.e. the name and path of the custom repository)
* `crema rm PACKAGE [PACKAGE] ...`
  Removes AUR packages and the corresponding package tarballs from the custom repository.
* `cream update`
  Updates all outdated AUR packages in the custom repository.



are a great means to automate Arch Linux installations. They are stored in [Custom repositories](https://wiki.archlinux.org/index.php/Pacman/Tips_and_tricks#Custom_local_repository). is a tool for managing such repositories. 

## License

[GNU Public License v3.0](https://github.com/mipimipi/crema/blob/master/LICENSE)
