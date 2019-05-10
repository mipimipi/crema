### Table of Content
* [Meta Packages](Meta-Packages)
* [Installation script](Installation-script)

With alime, the installation of an Arch Linux system comes down to the following steps:

1. Define a [meta package](Meta-Packages)

    This package contains the software packages that you want to have installed by the installation script as dependencies. Furthermore, it contains logic to create or adjust configuration files and to enable background services. Get further information about the meta package concept [here](Meta-Packages).

1. Run the [installation script](Installation-script)

    You boot your computer from the current [Arch Linux ISO](https://www.archlinux.org/download/) and execute the installation script with the command

        curl -sL https://git.io/fpSKp | bash

    The scripts needs a simple configuration file, that you could store on a local web server, e.g. In there, you specify the hostname, locale, partition sizes, meta package, admin user name etc. Since the scripts installs a LUKS encrypted system, the script asks for a corresponding passphrase and also for a root password. The required software is installed during the installation process by pacstrapping the above-mentioned meta package. Get further information about the installation script [here](Installation-script).

1. Reboot and use your new system

    Once the installation has finished, you remove the Arch Linux ISO media and reboot the system.
