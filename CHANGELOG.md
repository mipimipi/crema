
## [Release 1.4](https://github.com/mipimipi/crema/releases/tag/1.4) (2019-06-29)

### Added

* New commands `sign` and `unsign` to sign / unsign an entire repository with gpg (incl. repository database and package files)
* Additional checks for `cleanup` command:
    * Check for package files that do not have a corresponding entry in the package database
    * Check for signature files that do not have a corresponding data file
* Convenience logic for signing: If a repository is signed, and it shall be changed without but the `--sign` flag isn't set, the flag is set implicitly

### Changed

* Combined `add` and `build`commands:
    * So far, both commands built packages and added them to a repository. The only difference was the sources: Whereas `add` took PKGBUILD's from AUR, `build` took them from the local filesystem.
    * Now, there's only the `add` command which has an addtional flag `--from:<source>` where `<source>` can either be `aur` or `local`
* Per default, `makechrootpkg` is now used to build packages. If `makepkg` shall be used instead, the flag `--nochroot` has to be set.
* Comprehensive refactoring

### Removed

* Flag `--chroot` since it is obsolete.

## [Release 1.3](https://github.com/mipimipi/crema/releases/tag/1.3) (2019-06-19)

### Added

* Support of `makechrootpkg` to build packages. Default is `makepkg`.
* Commands `ls` and `update` can now work for all repositories at once.
* New command `cleanup` to make a repository consistent.

## [Release 1.2](https://github.com/mipimipi/crema/releases/tag/1.2) (2019-06-08)

### Added

* Management of multiple repositories
* Possibility to build packages from multiple PKGBUILD files.
* Possibility to sign packages

### Changed

* Replaced environment variables `CREMAREPO` and `CREMAREPODIR` by configuration file

## [Release 1.1](https://github.com/mipimipi/crema/releases/tag/1.1) (2019-05-20)

### Added

* `ls` sub command: Possibility to list all packages that are contained in the custom repository.


## [Release 1.0](https://github.com/mipimipi/crema/releases/tag/1.0) (2019-05-12)

First release suited for productive use.
