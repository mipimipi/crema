Meta packages are a means to automate the installation of software package in Arch Linux. They are [created](https://wiki.archlinux.org/index.php/Creating_packages) like "normal" packages, but their main purpose is to depend on other packages to facilitate their installation.

## Contents

* [Usage](#usage)
  - [Installation of software packages](#inst)
  - [Configuration of software packages](#config)
    - [New configuration files](#config-new)
    - [Changing existing configuration files](#config-change)
  - [Enabling services](#services)
* [Local repository](#repo)
* [Building meta packages](#build)
* [AUR packages](#aur)

## <a name="usage"></a>Usage

[Michael Daffin](https://github.com/mdaffin) described the usage of meta packages very well in his blog post [Managing Arch Linux with Meta Packages](https://disconnected.systems/blog/archlinux-meta-packages/). Since there's no need to repeat that, this page focuses on the specifics of the alime approach.

With respect to installation automation, meta packages can be used for different purposes. These are in particular

1. Installation of software packages
1. Configuration of software packages
1. Enabling of services

### <a name="inst"></a>Installation of software packages

As already mentioned, the main purpose of meta packages is to depend on other packages. Thus, the installation of a meta package triggers the installation of these dependencies. So one could create a meta package that depends on all the software packages that one would like to have installed in an Arch Linux system. Pacstrapping that package would then trigger the installation of all these software packages.

Meta packages can depend on other meta packages. Thus, one could create a hierarchy of meta packages. E.g.:

    base-meta-pkg
      |
      |- desktop-meta-pkg
      |
      |- laptop-meta-pkg
      |
      ...

In this example, there's a meta package for each of the different machine types that Arch Linux is intended to be installed upon (desktop, laptop etc.). They all depend on a base meta package. That package in turn depends on the software packages that all machines need. The specialized meta packages for desktop, laptop etc. only depend of software packages that are specific for the installation on a desktop or laptop.

For more information about how meta packages can be created, see [Michael Daffins blog posts](https://disconnected.systems/blog/archlinux-meta-packages/#creating-a-meta-package) and [an example in the alime Github repository](https://github.com/mipimipi/alime/tree/master/pkg).

### <a name="config"></a>Configuration of software packages

In addition to its installation, some software packages require configuration. This can also be automated with meta packages.

#### <a name="config-new"></a>New configuration files

Also this scenario is described very well in [Michael Daffins blog post](https://disconnected.systems/blog/archlinux-meta-packages/#adding-config-files). So, there is no need to elaborate on this topic here in further detail. Just one hint for [Emacs](https://wiki.archlinux.org/index.php/Emacs) users: There are dedicated [major modes](https://www.emacswiki.org/emacs/MajorMode) available PKGBUILD files (e.g. [PKGBUILD mode](https://github.com/juergenhoetzel/pkgbuild-mode)). A very handy feature is the automated calculation of checksums for files that are listed in the source array of PKGBUILD. 

#### <a name="config-change"></a>Changing existing configuration files

If existing configuration files need to be changed or overwritten, a different approach must be pursued as [Michael Daffin explained](https://disconnected.systems/blog/archlinux-meta-packages/#overwriting-existing-configs). Different from Michael, alime uses the configuration management tool [holo](https://github.com/holocm/holo) for this task. holo uses [alpm hooks](https://www.archlinux.org/pacman/alpm-hooks.5.html) to manage configuration files. In a nutshell, the following steps are necessary to change or overwrite existing configuration files with holo:

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
    
    does the job. In [this example](https://github.com/mipimipi/alime/blob/master/pkg/mipi-base.ntp.conf.holoscript), a local time server is appended to `ntp.conf`.  

1. `holo apply` must be executed in the post_install(), post_upgrade() and post_remove() hooks of [PKGBUILD](https://wiki.archlinux.org/index.php/PKGBUILD#install).  

holo then makes the required changes (either file overwriting or change) during the installation of the meta package.

### <a name="services"></a>Enabling services

Another task that must be performed often when software packages have been installed is the enablement of services. Arch Linux by default disables all services of a package that has been installed. So, the required services need to be enabled explicitely. It is an obvious idea, to put a statement like

 `systemctl enable <name-of-service>`
 
 in one of the hooks of PKGBUILD and let the package installation process execute the enablement of the service. But this does not work as the process stops with an error:
 
 `System has not been booted with systemd as init system (PID 1). Can't operate. Failed to connect to bus: Host is down`
 
 (see the [discussion in the Arch Linux forum](https://bbs.archlinux.org/viewtopic.php?id=241969)).

Luckily, there is a nice feature in [systemd](https://wiki.archlinux.org/index.php/Systemd) that allows to preset service enablements (see the [man pages of systemd](https://www.systutorials.com/docs/linux/man/5-systemd.preset/)). In a nutshell, to enable a specific service, an enablement statement needs to be put into a certain configuration file of systemd during installation time (e.g. via some logic in a PKGBUILD hook). This configuration is interpreted, and during the next boot, the corresponding service enablements are done. That's exactly what is needed here.

See an example in the alime Github repo:

1. The file [mipi-base.systemd.preset](https://github.com/mipimipi/alime/blob/master/pkg/mipi-base.systemd.preset) contains the presets for the mipi-base meta package.
1. It is copied to the system-preset directory of systemd by a statement in the PKGBUILD:

    `install -Dm0644 mipi-base.systemd.preset "${pkgdir}"/usr/lib/systemd/system-preset/10-mipi-base.preset`

1. The statement `systemctl preset-all` is put into the post_install() and the post_upgrade() hook. It triggers the interpretation on the preset configuration. If `systemctl preset-all` is not executed, the new preset configuration is not read, i.e. the services will not be enabled.

## <a name="repo"></a>Local repository

Meta packages need to be stored in and provided by a repository. We suggest to create a [custom local repository](https://wiki.archlinux.org/index.php/Pacman/Tips_and_tricks#Custom_local_repository) for these purposes.

The following describes how to create such a repository using an FTP server.

1. Set up an FTP server in your local network and allow anonymous access to it
1. To store the new repository, create a new folder (e.g. `arch-repo` and a sub folder for the architecture, e.g. `x86_64`)
1. Enable SSH access to these folders. It is recommended to enable [authentification via public key](https://wiki.archlinux.org/index.php/SSH_keys) since this is more convenient for updating the repository later.

To make the new repository known to [pacman](https://wiki.archlinux.org/index.php/Pacman), add the following to your `/etc/pacman.conf`:

    [<repo name>]
    SigLevel = Optional TrustAll
    Server = ftp://<path-to-repo>/x86_64

This is exactly what is done during the [alime installation script](Installation-script) to enable the installation of the custom meta packages during the installation process. Therefore, the name of the repository, its URL and the name of the meta package need to be specified in the configuration file of the script.

## <a name="build"></a>Building meta packages

For the management of meta packages we suggest to create a dedicated directory that contains

* The PKGBUILD file
* The other relevant files such as configuration files and scripts

To facilitate building meta packages and adding them to a local repository, alime contains the script [`repobuild`](https://github.com/mipimipi/alime/blob/master/repobuild). It requires the configuration file `alimerepo.conf` in the user-specific configuration directory (normally, that's `~/.config`). `alimerepo.conf` just contains the name of the local repository and its URL:

    repo=<name of repository>
    repodir=<ssh path to repository directory>

To install `repobuild`, download and copy it to a directory of your choice.

To build the meta packages and update the local repository, execute `repobuild`in the meta package directory. 

## <a name="aur"></a>AUR packages

The installation of [AUR](https://aur.archlinux.org/) packages need some more attention. In contrast to packages that are provided by the [official Arch Linux repositories](https://wiki.archlinux.org/index.php/Official_repositories), it is not sufficient to only let a meta package depend on an AUR package to trigger its installation. The pacstrap command, which is executed during the Arch Linux installation process, would throw an error: `could not resolve dependencies` (see the [discussion in the Arch Linux Forum](https://bbs.archlinux.org/viewtopic.php?id=241682)). This can be solved with the help of [aurutils](https://github.com/AladW/aurutils). It contains scripts that automate the management of AUR packages and can be [installed from AUR](https://aur.archlinux.org/packages/aurutils). Of special interest for our use case is the script `aursync` it supports downloading AUR packages, adding them to local repositories and updating them if a new package version is available.

The alime script [reposync](https://github.com/mipimipi/alime/blob/master/reposync) is a wrapper around `aursync`.

To use it, download and copy it to a directory of your choice. `reposync` also relies on the configuration file `alimerepo.conf` (see above).

To download and add AUR packages to your local repository, execute `reposync <name of package 1> ... <name of package n>`

To update all AUR packages execute `reposync -u`