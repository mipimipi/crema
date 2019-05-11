# Automation of Arch Linux Installation

[Installing Arch Linux manually](https://wiki.archlinux.org/index.php/installation_guide) is great to learn how Linux is works. But if multiple machines need to be installed, this process cumbersome and annoying. Therefore, automation is needed.

## Installation of an Arch Linux System

Typically, the installation of Arch Linux consists of three steps:

1. Installation of the Arch Linux base system
1. Installation of additional software packages and its global (i.e. user-independent) configuration
1. Personal (i.e. user-specific) configuration of the system

### Installation of the Arch Linux Base System

For this task, Arch Linux installation scripts can be utilized, such as [Archlinux U Install](https://github.com/helmuthdu/aui), [arch-installer](https://github.com/rstacruz/arch-installer) or [archfi](https://github.com/MatMoul/archfi). Such installers typically install a base Arch Linux system only, i.e. no idditional software packages and only limited configuration.

### Installation of Additional Software Packages

This can be done via [meta packages](https://disconnected.systems/blog/archlinux-meta-packages/). These are Arch Linux packages that depend on other packages. Installating a meta packages triggers the installation of all packages that the meta package depends on. Thus, it can be used to install a specific set of Arch Linux packages. Get more detailed information [here](meta-packages.md).

### Personal Configuration of the System

For the user-specific configuration, [dotfile managers](https://wiki.archlinux.org/index.php/Dotfiles#Tools) can be used.

## A (more) Automated Installation Approach 

Though this approach is not completely automated, it's less manual and much faster, than a [manual Arch Linux installation](https://wiki.archlinux.org/index.php/installation_guide). The installation consists of the following steps:

### Preparation

1. Prepare the meta packages and provide it via a [custom repository](https://wiki.archlinux.org/index.php/Pacman/Tips_and_tricks#Custom_local_repository). Get more detailed information [here](meta-packages.md).
1. Download the [Arch Linux ISO](https://www.archlinux.org/download/) and prepare an installation media.

### Installation

1. But your machine from the Arch Linux ISO.
1. Install the base system using an installer. In case of [archfi](https://github.com/MatMoul/archfi), for example, the [base group](https://www.archlinux.org/groups/x86_64/base/) of packages is installated.
1. Reboot, add your custom repository to `/etc/pacman.conf`. Execute `pacman -Syu` and install the meta package via `pacman -S <your-meta-package>`.
1. Create a user (provided, this hasn't been done before) and create the user-specific configuration with a dotfile manager such as [yadm](https://github.com/TheLocehiliosan/yadm)
1. Use your new system
