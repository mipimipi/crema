# crema

[Custom repositories](https://wiki.archlinux.org/index.php/Pacman/Tips_and_tricks#Custom_local_repository) are personal [Arch Linux](https://www.archlinux.org/) repositories that can contain local packages (i.e. where the [PKGBUILD file](https://wiki.archlinux.org/index.php/PKGBUILD) is located in the local filesystem) or packages from the [AUR](https://aur.archlinux.org/), the Arch Linux user repository. crema (**C**ustom **Re**pository  **Ma**nager) helps to manage them and is particularly suitable for **remote** custom repositories. 

Some use cases for custom repositories:
* During the installation of Arch Linux you want to pacstrap AUR packages. Therefore, these packages must be provided by a repository.
* You want to use self-defined [meta packages](https://disconnected.systems/blog/archlinux-meta-packages/) to make Arch Linux installation more efficient
* You are running Arch Linux on severals machines in your local network and want to provide all of them with packages / package updates from a custom repository in your local network
* etc. etc. etc.

## Features

crema supports the following tasks:

* Building packages
* Adding and removing packages
* Updating packages
* Signing / unsigning packages and repositories
* Cleaning up repositories

## Installation

There is an [AUR package for crema](https://aur.archlinux.org/packages/crema-git/) that can be installed with tools such as [pacaur](https://github.com/E5ten/pacaur), [trizen](https://github.com/trizen/trizen) or [yay](https://github.com/Jguer/yay). The crema script is then stored in `/usr/bin`.

Another option is a manual installation. For this, clone this repository and copy the crema script to a directory of your choice.

## Usage

cream has sub commands for the different tasks:

* `crema build` builds and adds packages incl. corresponding package tarballs to a custom repository
* `crema cleanup` cleans up one or all custom repositories (e.g. if the database and the package files are not consistent anymore)
* `crema env` lists the repositories and their diretories
* `crema ls` lists all packages of one or all repositories
* `crema repo-add` Add a custom repository to the crema configuration
* `crema repo-rm` Remove a custom repository from the crema configuration
* `crema rm` removes AUR packages and the corresponding package tarballs from a custom repository.
* `cream sign` sign a entire repository (incl. the database and the package files) with [gpg](https://gnupg.org/)
* `crema unsign` remove signatures from a repository (incl. the database and the package files)
* `cream update` updates all outdated AUR packages in one or all custom repositories.

## Configuration

crema requires information about the repositories (name and directory). This needs to be stored in the configuration file `$XDG_CONFIG_HOME/crema.conf` in the form:

    <repository-name>|<directory-of-the-repository>

Since `rsync` is used to copy files from and to the (remote) custom repositories, the path must be "`rsync`-compliant". [SSH](https://en.wikipedia.org/wiki/Secure_Shell) is supported, i.e. the path can be of the form `<user>@<host>:<path>`. 

The configuration file can be maintained either manually via text editor or with the commands `crema repo-add` and `crema repo-rm`.

## Details

Basically, crema is a wrapper around `repo-add`, `repo-remove` (both part of the [pacman](https://wiki.archlinux.org/index.php/Pacman) package), [makepkg](https://wiki.archlinux.org/index.php/Makepkg), [makechrootpkg](https://wiki.archlinux.org/index.php/DeveloperWiki:Building_in_a_clean_chroot) and [aurutils](https://github.com/AladW/aurutils). It uses [rsync](https://wiki.archlinux.org/index.php/Rsync), to transfer repositories between remote locations and the local filesystem and the above-mentioned tools to manipulate the local copy. At the end of each crema command the local copy is removed since it's obsolete.

## Known Issues

### Adding packages to a custom repository leads to an `rsync` error.

**Reason:** This error can happen, if in the PKGBUILD file of the package the `epoch`is set. In this case, the name of the package contains a colon. If the system that hosts the repository does not allow colons in file names, `rsync` throws an error.

**Solution:** Unfortunately, there's no other solution than either changing the remote system / hoster or to not add such a package to the repository. Since inconsistencies can occur (after such an error occurred, the repository database can, for example, contain a package but the corresponding package tarball is not stored), `cream cleanup` helps to make the repository consistent again.

### During the build process, the error `error: <package-name>: signature from "<name, mail address>" is invalid` occurs.

**Reason:** In the chroot environment of `makechrootpkg` that key is not known.

**Solution:** Make the key known by executing `sudo arch-chroot /var/lib/aurbuild/x86_64/root pacman-key -r <key-id>`, provided `/var/lib/aurbuild/x86_64/root` is the root directory of the chroot environment (which is the default).

### Other errors occur during package build
    
**Reason:** This can be caused by an inconsistent chroot environment.
    
**Solution:** Deleting the directory `/var/lib/aurbuild` can help.

## License

[GNU Public License v3.0](https://github.com/mipimipi/crema/blob/master/LICENSE)
