# Change Log

## [ 2.0.0 ] - 2026-05-27

Relocated the repository from https://github.com/truenas/py-libzfs to
https://github.com/asomers/py-libzfs and started this changelog.  The changelog
includes all changes since the repository was moved, beginning with git hash
923b04a4d49 .

### Added

- Vdev stats now include trim statistics.
  ([#14](https://github.com/asomers/py-libzfs/pull/14))

- Added the ability for user to look up only those ZFS properties that they
  care about.
  ([#5](https://github.com/asomers/py-libzfs/pull/5))
  ([#12](https://github.com/asomers/py-libzfs/pull/12))
  ([#13](https://github.com/asomers/py-libzfs/pull/13))

### Changed

- py-libzfs will no longer fetch hidden ZFS properties by default.  Hidden
  properties are things most users don't care about, like `prevsnap`,
  `pbkdf2salt`, etc.
  ([#1](https://github.com/asomers/py-libzfs/pull/1))
  ([#3](https://github.com/asomers/py-libzfs/pull/3))

### Fixed

- Fixed the build with Cython-3.x  and Python 3.13.
  ([#7](https://github.com/asomers/py-libzfs/pull/7))

- Fixed intermittent EINVAL in `ZFSPool.sync`
  ([#2](https://github.com/asomers/py-libzfs/pull/2))

- Fixed the build on FreeBSD stable/13, stable/14, and 16.0-CURRENT.
  ([#3](https://github.com/asomers/py-libzfs/pull/3))

### Removed

- Dropped support for FreeBSD 12 and older.
  ([#9](https://github.com/asomers/py-libzfs/pull/9))
