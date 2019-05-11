# Meta Packages

Meta packages are [created](https://wiki.archlinux.org/index.php/Creating_packages) like "normal" packages, but their main purpose is to depend on other packages to facilitate the installation of these dependencies.

## Contents

* [Usage](#usage)
  - [Installation of Software Packages](#inst)
  - [Configuration of Software Packages](#config)
    - [New Configuration Files](#config-new)
    - [Changing Existing Configuration Files](#config-change)
  - [Enabling Services](#services)
* [Custom Repository](#repo)
* [Building Meta Packages](#build)
* [AUR Packages](#aur)

## <a name="usage"></a>Usage

[Michael Daffin](https://github.com/mdaffin) described the usage of meta packages very well in his blog post [Managing Arch Linux with Meta Packages](https://disconnected.systems/blog/archlinux-meta-packages/). This page focuses on some additional aspects.

With respect to installation automation, meta packages can be used for different purposes. These are in particular

1. Installation of software packages
1. Configuration of software packages
1. Enabling of services

### <a name="inst"></a>Installation of Software Packages

As already mentioned, the main purpose of meta packages is to depend on other packages. The installation of a meta package triggers the installation of these dependencies. One could create a meta package that depends on all the software packages that one would like to have installed in an Arch Linux system, for example. `pacman -S <name-of-meta-package>` would then trigger the installation of all the software packages that the meta package depends on.

Meta packages can depend on other meta packages. Thus, one could create a hierarchy of meta packages. E.g.:

    base-meta-pkg
      |
      |- desktop-meta-pkg
      |
      |- laptop-meta-pkg
      |
      ...

In this example, there's a meta package for each of the different machine types that Arch Linux is intended to be installed upon (desktop, laptop etc.). They all depend on a base meta package. That package in turn depends on the software packages that all machines need. The specialized meta packages for desktop, laptop etc. only depend on (besides the base meta package) software packages that are specific for the installation on a desktop or laptop.

For more information about how meta packages can be created, see an example [here](https://github.com/mipimipi/metapkg/blob/master/PKGBUILD).

### <a name="config"></a>Configuration of Software Packages

In addition to their installation, software packages often require configuration. This can be automated with meta packages.

#### <a name="config-new"></a>New configuration files

For some type of software packages, their configuration files need to be stored in a certain directory and it's not necessary to overwrite or change existing (default) files. An example is [sudo](https://wiki.archlinux.org/index.php/Sudo), where configuration files can be dropped into the directory `/etc/sudoers.d`. A proper approach for these packages is to make the corresponding configuration files part of the meta package and add an `install` line to the [PKGBUILD](https://wiki.archlinux.org/index.php/PKGBUILD) file, like I did it in [my meta packages](https://github.com/mipimipi/metapkg).  

#### <a name="config-change"></a>Changing Existing Configuration Files

If existing (default) configuration files need to be changed or overwritten, the configuration management tool [holo](https://github.com/holocm/holo) can do the job. It uses [alpm hooks](https://www.archlinux.org/pacman/alpm-hooks.5.html) to manage configuration files. In a nutshell, the following steps are necessary to change or overwrite an existing configuration files with holo:

1. The meta package must depend on the [holo AUR package](https://aur.archlinux.org/packages/holo). I.e. the holo AUR package must be installed - see the [specific approach for meta packages depending on AUR packages](#aur).

1. If an existing configuration file shall be overwritten, the PKGBUILD file of the corresponding meta package (e.g. the function `package_<package-name>()`) must contain a statement to copy the new file to the directory

    `"${pkgdir}"/usr/share/holo/files/<some-custom-identifier>/<name-of-config-file>`
  
    If for example the ntp configuration `/etc/ntp.conf` shall be overwritten, the statement

    `install -Dm0644 mipi-base.ntp.conf "${pkgdir}"/usr/share/holo/files/20-ntp/etc/ntp.conf`
  
    does the job.

1. If an existing configuration file shall be adjusted or changed, create a script with the name `<name-of-config-file>.holoscript` and copy it to the directory

    `"${pkgdir}"/usr/share/holo/files/<some-custom-identifier>/<name-of-config-file>`
    
    If for example the ntp configuration `/etc/ntp.conf` shall be changed, the statement
    
    `install -Dm0750 mipi-base.ntp.conf.holoscript "${pkgdir}"/usr/share/holo/files/20-ntp/etc/ntp.conf.holoscript`
    
    does the job. 

1. `holo apply` must be executed in the `post_install()`, `post_upgrade()` and `post_remove()` hooks of [PKGBUILD](https://wiki.archlinux.org/index.php/PKGBUILD#install).  

holo then makes the required changes (either file overwrite or change) during the installation of the meta package.

### <a name="services"></a>Enabling services

Another task that must be performed often when software packages have been installed is the enablement of services. Arch Linux by default disables all services of a package that has been installed. So, the required services need to be enabled explicitely. It is an obvious idea, to put a statement like

 `systemctl enable <name-of-service>`
 
in one of the hooks of PKGBUILD and let the package installation process execute the enablement of the service. But this does not work as the process stops with an error:
 
 `System has not been booted with systemd as init system (PID 1). Can't operate. Failed to connect to bus: Host is down`
 
(see the [discussion in the Arch Linux forum](https://bbs.archlinux.org/viewtopic.php?id=241969)).

Luckily, there is a nice feature in [systemd](https://wiki.archlinux.org/index.php/Systemd) that allows to preset service enablements (see the [man pages of systemd](https://www.systutorials.com/docs/linux/man/5-systemd.preset/)). In a nutshell, to enable a specific service, an enablement statement needs to be put into a certain configuration file of systemd during installation time (e.g. via some logic in a PKGBUILD hook). This configuration is interpreted, and during the next boot, the corresponding service enablements are done. That's exactly what is needed here.

See an example in [my meta packages](https://github.com/mipimipi/metapkg):

1. The file [mipi-base.systemd.preset](https://github.com/mipimipi/alime/blob/master/pkg/mipi-base.systemd.preset) contains the presets for the mipi-base meta package.
1. It is copied to the system-preset directory of systemd by a statement in the PKGBUILD file:

    `install -Dm0644 mipi-base.systemd.preset "${pkgdir}"/usr/lib/systemd/system-preset/10-mipi-base.preset`

1. The statement `systemctl preset-all` is put into the `post_install()` and the `post_upgrade()` hook. It triggers the interpretation on the preset configuration. If `systemctl preset-all` is not executed, the new preset configuration is not read, i.e. the services will not be enabled.

## <a name="repo"></a>Custom Repository

Meta packages need to be stored in and provided by a repository. We suggest to create a [custom local repository](https://wiki.archlinux.org/index.php/Pacman/Tips_and_tricks#Custom_local_repository) for this purpos.

The following describes how to create such a repository using an FTP server:

1. Set up an FTP server in your local network and allow anonymous access to it
1. To store the new repository, create a new folder (e.g. `my-repo` and a sub folder for the architecture, e.g. `x86_64`)
1. Enable SSH access to these folders. It is recommended to enable [authentification via public key](https://wiki.archlinux.org/index.php/SSH_keys) since this is more convenient for updating the repository later.

To make the new repository known to [pacman](https://wiki.archlinux.org/index.php/Pacman), add the following to your `/etc/pacman.conf`:

    [<repo name>]
    SigLevel = Optional TrustAll
    Server = ftp://my-repo/x86_64

crema helps to manage such a custom repository.

## <a name="build"></a>Building meta packages

For the management of a meta package we suggest to create a dedicated directory that contains

* The PKGBUILD file
* Other relevant files such as configuration files and scripts

crema can be used to build the meta package(s). For this, execute `crema build` in the directory where the PKGBUILD file is stored.

## <a name="aur"></a>AUR packages

The installation of [AUR](https://aur.archlinux.org/) packages need some more attention. In contrast to packages that are provided by the [official Arch Linux repositories](https://wiki.archlinux.org/index.php/Official_repositories), it is not sufficient to only let a meta package depend on an AUR package to trigger its installation. The pacstrap command, which is executed during the Arch Linux installation process, would throw an error: `could not resolve dependencies` (see the [discussion in the Arch Linux Forum](https://bbs.archlinux.org/viewtopic.php?id=241682)). A solution approach is to add the AUR package to the custom repository. Then the meta package can depend on it without getting an error during the installation process.

Also here, crema can help:

* Execute `crema add <package-name>` to add an AUR package to the custom repository
* Execute `crema update` to updated all outdated AUR packages of your custom repository

As a side effect, you can install, remove or update AUR packages from your custom repository with the `pacman` command.
