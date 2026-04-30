py-libzfs
=========

**Python bindings for libzfs**

py-libzfs is a fairly straight-forward set of Python bindings for libzfs for ZFS on Linux and FreeBSD.


**INSTALLATION**

`./configure && make install`

**FEATURES:**
- Access to pools, datasets, snapshots, properties, pool disks
- Many others!

**QUICK HOWTO:**

`import libzfs`

Get a list of pools:

`pools = list(libzfs.ZFS().pools)`

Get help:

`help(libzfs)`

**HISTORY**

py-libzfs originated at ixSystems, where it was a part of FreeNAS.  They
separated it into its own project in 2015.  In 2026 they deprecated py-libzfs,
replacing it with truenas_pylibzfs[^1], which has a very different design.
However, that library is only maintained for TrueNAS Scale, and isn't portable.
So ConnectWise took over maintenance of the legacy py-libzfs.

[^1]: [truenas_pylibzfs](https://github.com/truenas/truenas_pylibzfs)
