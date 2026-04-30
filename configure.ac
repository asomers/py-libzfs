AC_INIT([py-libzfs],[1.1])

AC_LANG(C)
AC_PROG_CC


define(SECTION_TITLE,
[
	echo >& AS_MESSAGE_FD
	echo '   $1   ' | sed -e's/./-/g' >&AS_MESSAGE_FD
	echo '   $1' >& AS_MESSAGE_FD
	echo '   $1   ' | sed -e's/./-/g' >&AS_MESSAGE_FD
])


dnl works for checking define statements in c
m4_include([m4/AX_CHECK_DEFINE.m4])


build_linux=no
build_freebsd=no
build_macos=no


host_os=$(uname -s)

# Detect the target system
case "${host_os}" in
	Linux*)
		build_linux=yes
		;;
	FreeBSD*)
		build_freebsd=yes
		;;
	Darwin*)
		build_macos=yes
		;;
	*)
		AC_MSG_ERROR(["OS $host_os is not supported"])
		;;
esac

CFLAGS="-DCYTHON_FALLTHROUGH"
LIBS="-lzfs -lnvpair -lzfs_core"

if [[ "${build_freebsd}" = "yes" ]]; then
	LIBS="-lgeom ${LIBS}"
fi

if [[ "${build_macos}" = "yes" ]]; then
	LIBS="-lzpool ${LIBS}"
fi


dnl We allow setting a zof variable which the users can set to force autoconf to consider the system is zof based

AC_ARG_VAR(zof, AS_HELP_STRING([Force configure to run ZoL(f) checks]))
AC_ARG_VAR(freebsd_src, AS_HELP_STRING([Provide path for freebsd source tree. This defaults to /usr/src/]))

if [[ "$prefix" = "NONE" ]]; then
	if [[ "$build_macos" = "yes" ]]; then
		header_prefix=/usr/local/zfs
	else
		header_prefix=/usr/local
	fi
else
	header_prefix=$prefix
fi



: ${freebsd_src:=/usr/src}

zof=false
AC_SUBST(IS_OPENZFS, 1)
if [[ -f "$header_prefix/include/libzfs/libzfs.h" ]] || [[ "$zof" = true ]]; then
	zof=true

	CFLAGS="${CFLAGS} 
-I${header_prefix}/include/libzfs
-I${header_prefix}/include/libspl
-D_MACHINE_ENDIAN_H_ -DHAVE_ISSETUGID -D_GNU_SOURCE
"

	LDFLAGS="-L${header_prefix}/lib ${LDFLAGS}"

elif [[ -d "${freebsd_src}/sys/contrib/openzfs" ]]; then
	zof=true

	openzfs_prefix="${freebsd_src}/sys/contrib/openzfs"

	CFLAGS="${CFLAGS}
-DIN_BASE -DHAVE_RPC_TYPES
-I${openzfs_prefix}/include
-I${openzfs_prefix}/include/os/freebsd
-I${openzfs_prefix}/lib/libspl/include
-I${openzfs_prefix}/lib/libspl/include/os/freebsd
-I${freebsd_src}/sys
-I${freebsd_src}/cddl/compat/opensolaris/include
-include ${openzfs_prefix}/include/os/freebsd/spl/sys/ccompile.h
-DHAVE_ISSETUGID
"
	LIBS="-lspl -lzutil ${LIBS}"
else
	AC_SUBST(IS_OPENZFS, 0)

        CFLAGS="${CFLAGS} 
-I${freebsd_src}/cddl/lib/libumem
-I${freebsd_src}/sys/cddl/compat/opensolaris
-I${freebsd_src}/cddl/compat/opensolaris/include
-I${freebsd_src}/cddl/compat/opensolaris/lib/libumem
-I${freebsd_src}/cddl/contrib/opensolaris/lib/libzpool/common
-I${freebsd_src}/sys/cddl/contrib/opensolaris/common/zfs
-I${freebsd_src}/sys/cddl/contrib/opensolaris/uts/common/fs/zfs
-I${freebsd_src}/sys/cddl/contrib/opensolaris/uts/common/sys
-I${freebsd_src}/cddl/contrib/opensolaris/head
-I${freebsd_src}/sys/cddl/contrib/opensolaris/uts/common
-I${freebsd_src}/cddl/contrib/opensolaris/lib/libnvpair
-I${freebsd_src}/cddl/contrib/opensolaris/lib/libuutil/common
-I${freebsd_src}/cddl/contrib/opensolaris/lib/libzfs/common
-I${freebsd_src}/cddl/contrib/opensolaris/lib/libzfs_core/common 
-DNEED_SOLARIS_BOOLEAN -D_XPG6
"
fi

if [[ -f "$header_prefix/include/libuutil.h" ]]; then
	LIBS="-luutil ${LIBS}"
else
	LIBS="-lpthread ${LIBS}"
fi

if [[ "$build_macos" = "yes" ]]; then
	CFLAGS="${CFLAGS} -DHAVE_STRLCAT -DHAVE_STRLCPY"
fi

CFLAGS="${CFLAGS} -Werror=implicit-function-declaration"


SECTION_TITLE([RUNNING TESTS ON zfs.h HEADER])


AC_CHECK_HEADERS([sys/types.h], [], [AC_MSG_ERROR(A working sys/types header is required)])
AC_CHECK_HEADERS([sys/fs/zfs.h], [], [AC_MSG_ERROR(A working zfs header is required)],[
	#include <sys/types.h>
])


AC_MSG_NOTICE([Checking existence of pool_raidz_expand_stat_t])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <sys/fs/zfs.h>],
		[
			pool_raidz_expand_stat_t *f = NULL;
		]
	)],
	[AC_SUBST(HAVE_POOL_RAIDZ_EXPAND_STAT_T, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_POOL_RAIDZ_EXPAND_STAT_T, 0) AC_MSG_NOTICE([Not found])]
)



SECTION_TITLE([RUNNING TESTS ON libzutil.h HEADER])

dnl Checking function zpool_read_label existence

AC_MSG_NOTICE([Checking existence of zpool_read_label in libzutil.h header])

dnl Checking function zpool_search_import signature and existence

AC_MSG_NOTICE([Checking existence of zpool_search_import in libzutil.h header])

AC_SUBST(HAVE_ZPOOL_SEARCH_IMPORT_PARAMS, 0)

AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <libzutil.h>],
		[
			zpool_search_import(NULL, NULL, NULL);
		]
	)],
	[AC_SUBST(HAVE_ZPOOL_SEARCH_IMPORT_LIBZUTIL, 1) AC_SUBST(HAVE_ZPOOL_SEARCH_IMPORT_PARAMS, 3) AC_MSG_NOTICE(OK)],
	[AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <libzutil.h>],
		[
			zpool_search_import(NULL, NULL);
		]
	)],
	[AC_SUBST(HAVE_ZPOOL_SEARCH_IMPORT_LIBZUTIL, 1) AC_SUBST(HAVE_ZPOOL_SEARCH_IMPORT_PARAMS, 2) AC_MSG_NOTICE(OK)],
	[AC_SUBST(HAVE_ZPOOL_SEARCH_IMPORT_LIBZUTIL, 0) AC_MSG_NOTICE(NOT FOUND)])]
)


SECTION_TITLE([RUNNING TESTS ON libzfs.h HEADER])


if [[ "$zof" = false ]]; then
	AC_CHECK_HEADERS([libzfs.h], [], [AC_MSG_ERROR(A working libzfs header is required)], [
		#include <sys/types.h>
		#include <sys/mnttab.h>
		#include <sys/fs/zfs.h>
		#include <sys/zfs_ioctl.h>
	])
else
	AC_CHECK_HEADERS([libzfs.h], [], [AC_MSG_ERROR(A working libzfs header is required)], [
		#include <sys/types.h>
		#include <sys/mnttab.h>
		#include <sys/fs/zfs.h>
	])
fi


dnl Checking for parameters of zpool_vdev_attach

AC_MSG_NOTICE([Checking zpool_vdev_attach parameters])

AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			zpool_vdev_attach(NULL, NULL, NULL, NULL, 0, 0);
		]
	)],
	[AC_SUBST(HAVE_ZPOOL_VDEV_ATTACH, 6) AC_MSG_NOTICE([6 params])],
	[AC_SUBST(HAVE_ZPOOL_VDEV_ATTACH, 5) AC_MSG_NOTICE([5 params])]
)


dnl Checking for existence of zfs_foreach_mountpoint function

AC_MSG_NOTICE([Checking for existence of zfs_foreach_mountpoint])

AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			zfs_foreach_mountpoint(NULL, NULL, 0, NULL, NULL, 0);
		],
	)],
	[AC_SUBST(HAVE_ZFS_FOREACH_MOUNTPOINT, 1) AC_MSG_NOTICE(OK)],
	[AC_SUBST(HAVE_ZFS_FOREACH_MOUNTPOINT, 0) AC_MSG_NOTICE(NOT FOUND)]
)

dnl Checking for existence of zpool_ddt_prune function

AC_MSG_NOTICE([Checking for existence of zpool_ddt_prune])

AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			zpool_ddt_prune(NULL, 0, 0);
		],
	)],
	[AC_SUBST(HAVE_ZPOOL_DDT_PRUNE, 1) AC_MSG_NOTICE(OK)],
	[AC_SUBST(HAVE_ZPOOL_DDT_PRUNE, 0) AC_MSG_NOTICE(NOT FOUND)]
)

dnl Checking for existence of zpool_prefetch function

AC_MSG_NOTICE([Checking for existence of zpool_prefetch])

AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			zpool_prefetch(NULL, 0);
		],
	)],
	[AC_SUBST(HAVE_ZPOOL_PREFETCH, 1) AC_MSG_NOTICE(OK)],
	[AC_SUBST(HAVE_ZPOOL_PREFETCH, 0) AC_MSG_NOTICE(NOT FOUND)]
)

dnl Checking parameters of zvol_volsize_to_reservation function

AC_MSG_NOTICE([Checking function signature zvol_volsize_to_reservation])

AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			zvol_volsize_to_reservation(NULL, 0, NULL) /* ; */
		]
	)],
	[AC_SUBST(HAVE_ZVOLSIZE_TO_RESERVATION_PARAMS, 3) AC_MSG_NOTICE([3 PARAMS])],
	[AC_SUBST(HAVE_ZVOLSIZE_TO_RESERVATION_PARAMS, 2) AC_MSG_NOTICE([2 PARAMS])]
)


dnl Checking parameters of zpool_add function

AC_MSG_NOTICE([Checking function signature zpool_add])

AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			zpool_add(NULL, NULL, 0) /* ; */
		]
	)],
	[AC_SUBST(HAVE_ZPOOL_ADD, 3) AC_MSG_NOTICE([3 PARAMS])],
	[AC_SUBST(HAVE_ZPOOL_ADD, 2) AC_MSG_NOTICE([2 PARAMS])]
)


dnl Checking function zpool_search_import existence

AC_MSG_NOTICE([Checking existence of zpool_search_import in libzfs.h header])

AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			zpool_search_import(NULL, NULL);
		],
	)],
	[AC_SUBST(HAVE_ZPOOL_SEARCH_IMPORT_LIBZFS, 1) AC_SUBST(HAVE_ZPOOL_SEARCH_IMPORT_PARAMS, 2) AC_MSG_NOTICE(OK)],
	[AC_SUBST(HAVE_ZPOOL_SEARCH_IMPORT_LIBZFS, 0) AC_MSG_NOTICE(NOT FOUND)]
)


dnl Checking function zpool_events_next existence

AC_MSG_NOTICE([Checking existence of zpool_events_next in libzfs.h header])

AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			zpool_events_next(NULL, NULL, NULL, 0, 0);
		],
	)],
	[AC_SUBST(HAVE_ZPOOL_EVENTS_NEXT, 1) AC_MSG_NOTICE(OK)],
	[AC_SUBST(HAVE_ZPOOL_EVENTS_NEXT, 0) AC_MSG_NOTICE(NOT FOUND)]
)



dnl Checking function signature of zfs_share

AC_MSG_NOTICE([Checking function signature of zfs_share])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			zfs_share(NULL) /* ; */
		]
	)],
	[AC_SUBST(HAVE_ZFS_SHARE, 1) AC_MSG_NOTICE([1 param])],
	[AC_SUBST(HAVE_ZFS_SHARE, 2) AC_MSG_NOTICE([2 params])]
)


dnl Checking function signature of zfs_iter_filesystems

AC_MSG_NOTICE([Checking function signature of zfs_iter_filesystems])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		   [#include <libzfs.h>],
		   [
			(void) zfs_iter_filesystems(NULL, 0, NULL, NULL) /* ; */
		   ]
	)],
	[AC_SUBST(HAVE_ZFS_ITER_FILESYSTEMS, 4) AC_MSG_NOTICE([4 params])],
	[AC_SUBST(HAVE_ZFS_ITER_FILESYSTEMS, 3) AC_MSG_NOTICE([3 params])]
)


dnl Checking function signature of zfs_iter_snapspec

AC_MSG_NOTICE([Checking function signature of zfs_iter_snapspec])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		   [#include <libzfs.h>],
		   [
			(void) zfs_iter_snapspec(NULL, 0, NULL, NULL, NULL) /* ; */
		   ]
	)],
	[AC_SUBST(HAVE_ZFS_ITER_SNAPSPEC, 5) AC_MSG_NOTICE([5 params])],
	[AC_SUBST(HAVE_ZFS_ITER_SNAPSPEC, 4) AC_MSG_NOTICE([4 params])]
)


dnl Checking function signature of zfs_iter_dependents

AC_MSG_NOTICE([Checking function signature of zfs_iter_dependents])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		   [#include <libzfs.h>],
		   [
			(void) zfs_iter_dependents(NULL, 0, B_FALSE, NULL, NULL) /* ; */
		   ]
	)],
	[AC_SUBST(HAVE_ZFS_ITER_DEPENDENTS, 5) AC_MSG_NOTICE([5 params])],
	[AC_SUBST(HAVE_ZFS_ITER_DEPENDENTS, 4) AC_MSG_NOTICE([4 params])]
)


dnl Checking function signature of zfs_iter_bookmarks

AC_MSG_NOTICE([Checking function signature of zfs_iter_bookmarks])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		   [#include <libzfs.h>],
		   [
			(void) zfs_iter_bookmarks(NULL, 0, NULL, NULL) /* ; */
		   ]
	)],
	[AC_SUBST(HAVE_ZFS_ITER_BOOKMARKS, 4) AC_MSG_NOTICE([4 params])],
	[AC_SUBST(HAVE_ZFS_ITER_BOOKMARKS, 3) AC_MSG_NOTICE([3 params])]
)


dnl Checking function signature of zfs_iter_snapshots_sorted

AC_MSG_NOTICE([Checking function signature of zfs_iter_snapshots_sorted])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		   [#include <libzfs.h>],
		   [
			(void) zfs_iter_snapshots_sorted(NULL, 0, NULL, NULL, 0, 0) /* ; */
		   ]
	)],
	[AC_SUBST(HAVE_ZFS_ITER_SNAPSHOTS_SORTED, 6) AC_MSG_NOTICE([6 params])],
	[AC_SUBST(HAVE_ZFS_ITER_SNAPSHOTS_SORTED, 5) AC_MSG_NOTICE([5 params])]
)


dnl Checking function signature of zfs_iter_children

AC_MSG_NOTICE([Checking function signature of zfs_iter_children])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		   [#include <libzfs.h>],
		   [
			(void) zfs_iter_children(NULL, 0, NULL, NULL) /* ; */
		   ]
	)],
	[AC_SUBST(HAVE_ZFS_ITER_CHILDREN, 4) AC_MSG_NOTICE([4 params])],
	[AC_SUBST(HAVE_ZFS_ITER_CHILDREN, 3) AC_MSG_NOTICE([3 params])]
)


dnl Checking function signature of zfs_send_one

AC_MSG_NOTICE([Checking function signature of zfs_send_one])

if [[ "$zof" = false ]]; then
	AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
			[#include <libzfs.h>],
			[
				zfs_send_one(NULL, NULL, 0, 0) /* ; */
			]
		)],
		[AC_SUBST(HAVE_ZFS_SEND_ONE, 4) AC_MSG_NOTICE([4 params])],
		[AC_SUBST(HAVE_ZFS_SEND_ONE, 3) AC_MSG_NOTICE([3 params])]
	)
else
	AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
			[#include <libzfs.h>],
			[
				sendflags_t flag = { 0 };
				zfs_send_one(NULL, NULL, 0, flag) /* ; */
			]
		)],
		[AC_SUBST(HAVE_ZFS_SEND_ONE, 4) AC_MSG_NOTICE([4 params])],
		[AC_SUBST(HAVE_ZFS_SEND_ONE, 3) AC_MSG_NOTICE([3 params])]
	)
fi

dnl Checking function signature of zfs_userspace_cb_t
# This function type gained a fifth argument in FreeBSD 15

AC_MSG_NOTICE([Checking function signature of zfs_userspace_cb_t])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
		 int f(void *, const char *, uid_t, uint64_t, uint64_t);
		 zfs_userspace_cb_t g = f;
		]
	)],
	[AC_SUBST(HAVE_ZFS_USERSPACE_CB_T, 5) AC_MSG_NOTICE([5 params])],
	[AC_SUBST(HAVE_ZFS_USERSPACE_CB_T, 4) AC_MSG_NOTICE([4 params])]
)

dnl Checking existence of an enum value

# NB: the ERRORSCRUBBING, ERRORSCRUB_PAUSED, and SCRUB_PAUSED_TO_CANCEL symbols
# were all added to OpenZFS by a single commit.
AC_MSG_NOTICE([Checking EZFS_ERRORSCRUBBING presence])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			EZFS_ERRORSCRUBBING == EZFS_ERRORSCRUBBING /* ; */
		]
	)],
	[AC_SUBST(HAVE_EZFS_ERRORSCRUBBING, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_EZFS_ERRORSCRUBBING, 0) AC_MSG_NOTICE([NOT FOUND])]
)

dnl Checking existence of an enum value
AC_MSG_NOTICE([Checking EZFS_VDEV_NOTSUP presence])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			EZFS_VDEV_NOTSUP == EZFS_VDEV_NOTSUP /* ; */
		]
	)],
	[AC_SUBST(HAVE_EZFS_VDEV_NOTSUP, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_EZFS_VDEV_NOTSUP, 0) AC_MSG_NOTICE([NOT FOUND])]
)

dnl Checking existence of an enum value
AC_MSG_NOTICE([Checking EZFS_NOT_USER_NAMESPACE presence])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			EZFS_NOT_USER_NAMESPACE == EZFS_NOT_USER_NAMESPACE /* ; */
		]
	)],
	[AC_SUBST(HAVE_EZFS_NOT_USER_NAMESPACE, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_EZFS_NOT_USER_NAMESPACE, 0) AC_MSG_NOTICE([NOT FOUND])]
)

dnl Checking existence of an enum value
AC_MSG_NOTICE([Checking EZFS_RESUME_EXISTS presence])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			EZFS_RESUME_EXISTS == EZFS_RESUME_EXISTS /* ; */
		]
	)],
	[AC_SUBST(HAVE_EZFS_RESUME_EXISTS, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_EZFS_RESUME_EXISTS, 0) AC_MSG_NOTICE([NOT FOUND])]
)

dnl Checking existence of an enum value
AC_MSG_NOTICE([Checking EZFS_SHAREFAILED presence])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			EZFS_SHAREFAILED == EZFS_SHAREFAILED /* ; */
		]
	)],
	[AC_SUBST(HAVE_EZFS_SHAREFAILED, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_EZFS_SHAREFAILED, 0) AC_MSG_NOTICE([NOT FOUND])]
)

dnl Checking existence of an enum value
AC_MSG_NOTICE([Checking EZFS_RAIDZ_EXPAND_IN_PROGRESS presence])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			EZFS_RAIDZ_EXPAND_IN_PROGRESS == EZFS_RAIDZ_EXPAND_IN_PROGRESS /* ; */
		]
	)],
	[AC_SUBST(HAVE_EZFS_RAIDZ_EXPAND_IN_PROGRESS, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_EZFS_RAIDZ_EXPAND_IN_PROGRESS, 0) AC_MSG_NOTICE([NOT FOUND])]
)

dnl Checking existence of an enum value
AC_MSG_NOTICE([Checking EZFS_ASHIFT_MISMATCH presence])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			EZFS_ASHIFT_MISMATCH == EZFS_ASHIFT_MISMATCH /* ; */
		]
	)],
	[AC_SUBST(HAVE_EZFS_ASHIFT_MISMATCH, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_EZFS_ASHIFT_MISMATCH, 0) AC_MSG_NOTICE([NOT FOUND])]
)

dnl Checking ZPOOL_MAXNAMELEN

AX_CHECK_DEFINE([libzfs.h], [ZPOOL_MAXNAMELEN], [AC_SUBST(HAVE_ZPOOL_MAXNAMELEN, 1)], [AC_SUBST(HAVE_ZPOOL_MAXNAMELEN, 0)])


dnl Checking for resumable member in recvflags_t struct

AC_MSG_NOTICE([Checking for resumable member in recvflags_t struct])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			recvflags_t t;
			t.resumable = 1 /* ; */
		]
	)],
	[AC_SUBST(HAVE_RECVFLAGS_T_RESUMABLE, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_RECVFLAGS_T_RESUMABLE, 0) AC_MSG_NOTICE([NOT FOUND])]
)

dnl Checking for pss_skipped in struct pool_scan_stat

AC_MSG_NOTICE([Checking for pss_skipped member in pool_scan_stat struct])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			pool_scan_stat t;
			t.pss_skipped = 1 /* ; */
		]
	)],
	[AC_SUBST(HAVE_POOL_SCAN_STAT_PSS_SKIPPED, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_POOL_SCAN_STAT_PSS_SKIPPED, 0) AC_MSG_NOTICE([NOT FOUND])]
)



dnl Checking typedef sendflags_t in zfs header

AC_MSG_NOTICE([Checking typedef sendflags_t in libzfs header])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			int i = sizeof(sendflags_t);
			i = i; /* ; */
		]
	)],
	[AC_SUBST(HAVE_SENDFLAGS_T_TYPEDEF, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_SENDFLAGS_T_TYPEDEF, 0) AC_MSG_NOTICE([Not found])]
)


dnl Checking attribute name for verbose in sendflags_t struct

AC_MSG_NOTICE([Checking attribute name for verbose in sendflags_t struct])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			sendflags_t t;
			t.verbosity = 1 /* ; */
		]
	)],[
		AC_SUBST(HAVE_SENDFLAGS_T_VERBOSITY, 1)
		AC_SUBST(SENDFLAGS_T_VERBOSITY, "int verbosity")
		AC_MSG_NOTICE([Attribute named as verbosity in sendflags_t struct])
	],[
		AC_SUBST(HAVE_SENDFLAGS_T_VERBOSITY, 0)
		AC_SUBST(SENDFLAGS_T_VERBOSITY, "int verbose")
		AC_MSG_NOTICE([Attribute named as verbose in sendflags_t struct])
	]
)


dnl Checking for dedup member in sendflags_t struct

AC_MSG_NOTICE([Checking for dedup member in sendflags_t struct])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			sendflags_t t;
			t.dedup = 1 /* ; */
		]
	)],[
		AC_SUBST(HAVE_SENDFLAGS_T_DEDUP, 1)
		AC_SUBST(SENDFLAGS_T_DEDUP, "int dedup")
		AC_MSG_NOTICE([OK])
	],[
		AC_SUBST(HAVE_SENDFLAGS_T_DEDUP, 0)
		AC_SUBST(SENDFLAGS_T_DEDUP, "int pad")
		AC_MSG_NOTICE([NOT FOUND])
	]
)


dnl Checking for compress member in sendflags_t struct

AC_MSG_NOTICE([Checking for compress member in sendflags_t struct])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			sendflags_t t;
			t.compress = 1 /* ; */
		]
	)],[
		AC_SUBST(HAVE_SENDFLAGS_T_COMPRESS, 1)
		AC_SUBST(SENDFLAGS_T_COMPRESS, "int compress")
		AC_MSG_NOTICE([OK])
	],[
		AC_SUBST(HAVE_SENDFLAGS_T_COMPRESS, 0)
		AC_SUBST(SENDFLAGS_T_COMPRESS, "")
		AC_MSG_NOTICE([NOT FOUND])
	]
)


dnl Checking for raw member in sendflags_t struct

AC_MSG_NOTICE([Checking for raw member in sendflags_t struct])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			sendflags_t t;
			t.raw = 1 /* ; */
		]
	)],[
		AC_SUBST(HAVE_SENDFLAGS_T_RAW, 1)
		AC_SUBST(SENDFLAGS_T_RAW, "int raw")
		AC_MSG_NOTICE([OK])
	],[
		AC_SUBST(HAVE_SENDFLAGS_T_RAW, 0)
		AC_SUBST(SENDFLAGS_T_RAW, "")
		AC_MSG_NOTICE([NOT FOUND])
	]
)


dnl Checking for backup member in sendflags_t struct

AC_MSG_NOTICE([Checking for backup member in sendflags_t struct])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			sendflags_t t;
			t.backup = 1 /* ; */
		]
	)],[
		AC_SUBST(HAVE_SENDFLAGS_T_BACKUP, 1)
		AC_SUBST(SENDFLAGS_T_BACKUP, "int backup")
		AC_MSG_NOTICE([OK])
	],[
		AC_SUBST(HAVE_SENDFLAGS_T_BACKUP, 0)
		AC_SUBST(SENDFLAGS_T_BACKUP, "")
		AC_MSG_NOTICE([NOT FOUND])
	]
)


dnl Checking for holds member in sendflags_t struct

AC_MSG_NOTICE([Checking for holds member in sendflags_t struct])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			sendflags_t t;
			t.holds = 1 /* ; */
		]
	)],[
		AC_SUBST(HAVE_SENDFLAGS_T_HOLDS, 1)
		AC_SUBST(SENDFLAGS_T_HOLDS, "int holds")
		AC_MSG_NOTICE([OK])
	],[
		AC_SUBST(HAVE_SENDFLAGS_T_HOLDS, 0)
		AC_SUBST(SENDFLAGS_T_HOLDS, "")
		AC_MSG_NOTICE([NOT FOUND])
	]
)


dnl Checking for saved member in sendflags_t struct

AC_MSG_NOTICE([Checking for saved member in sendflags_t struct])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			sendflags_t t;
			t.saved = 1 /* ; */
		]
	)],[
		AC_SUBST(HAVE_SENDFLAGS_T_SAVED, 1)
		AC_SUBST(SENDFLAGS_T_SAVED, "int saved")
		AC_MSG_NOTICE([OK])
	],[
		AC_SUBST(HAVE_SENDFLAGS_T_SAVED, 0)
		AC_SUBST(SENDFLAGS_T_SAVED, "")
		AC_MSG_NOTICE([NOT FOUND])
	]
)


dnl Checking for progressastitle member in sendflags_t struct

AC_MSG_NOTICE([Checking for progressastitle member in sendflags_t struct])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs.h>],
		[
			sendflags_t t;
			t.progressastitle = 1 /* ; */
		]
	)],[
		AC_SUBST(HAVE_SENDFLAGS_T_PROGRESSASTITLE, 1)
		AC_SUBST(SENDFLAGS_T_PROGRESSASTITLE, "int progressastitle")
		AC_MSG_NOTICE([OK])
	],[
		AC_SUBST(HAVE_SENDFLAGS_T_PROGRESSASTITLE, 0)
		AC_SUBST(SENDFLAGS_T_PROGRESSASTITLE, "")
		AC_MSG_NOTICE([NOT FOUND])
	]
)


SECTION_TITLE([RUNNING TESTS ON libzfs_core.h HEADER])


AC_CHECK_HEADERS([libzfs_core.h], [], [AC_MSG_ERROR(A working libzfs_core header is required)], [
	#include <sys/types.h>
	#include <sys/param.h>
])


dnl Checking lzc_bookmark existence

AC_MSG_NOTICE([Verifying lzc_bookmark existence])

exists=0
AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs_core.h>],
		[
			lzc_bookmark(NULL, NULL) /* ; */
		]
	)],
	[exists=1],
	[AC_MSG_NOTICE([NOT FOUND])]
)


if test "$exists" == 1
then
	AC_MSG_NOTICE([OK])
	AC_SUBST(HAVE_LZC_BOOKMARK, 1)

	dnl Verifying existence and params of lzc_send_space
	AC_MSG_NOTICE([Verifying existence and params of lzc_send_space])

	exists=0
	AC_LINK_IFELSE([AC_LANG_PROGRAM(
			[#include <libzfs_core.h>],
			[
				lzc_send_space(NULL, NULL, LZC_SEND_FLAG_EMBED_DATA, NULL) /* ; */
			]
		)],
		[exists=1],
		[AC_MSG_NOTICE([NOT FOUND])]
	)

	if test "$exists" == 1
	then

		AC_MSG_NOTICE([4 params])
		AC_SUBST(HAVE_LZC_SEND_SPACE, 4)

	else

		AC_MSG_NOTICE([3 params])
		AC_SUBST(HAVE_LZC_SEND_SPACE, 3)
	fi

else

	AC_MSG_NOTICE([lzc_send_space function not found])
	AC_SUBST(HAVE_LZC_BOOKMARK, 0)
	AC_SUBST(HAVE_LZC_SEND_SPACE, 0)

fi

dnl Checking lzc_sync existence

AC_MSG_NOTICE([Verifying lzc_sync existence])

exists=0
AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs_core.h>],
		[
			lzc_sync(NULL, NULL, NULL) /* ; */
		]
	)],
	[AC_SUBST(HAVE_LZC_SYNC, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_LZC_SYNC, 0) AC_MSG_NOTICE([NOT FOUND])]
)


dnl Checking lzc_wait existence

AC_MSG_NOTICE([Verifying lzc_wait existence])

AC_LINK_IFELSE([AC_LANG_PROGRAM(
		[#include <libzfs_core.h>],
		[
			lzc_wait(NULL, 0, NULL) /* ; */
		]
	)],
	[AC_SUBST(HAVE_LZC_WAIT, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_LZC_WAIT, 0) AC_MSG_NOTICE([NOT FOUND])]
)


SECTION_TITLE([RUNNING TESTS ON zfeature_common.h HEADER])

AC_MSG_NOTICE([Verifying existence of zfeature_common.h])

AC_CHECK_HEADERS([zfeature_common.h],
	[AC_SUBST(HAVE_ZFEATURE_COMMON_HEADER, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_ZFEATURE_COMMON_HEADER, 0) AC_MSG_NOTICE([NOT FOUND])],
	[]
)

AC_MSG_NOTICE([Checking supported SPA features])

# SPA_FEATURE_CHECK(feature, [success], [failure])
# ------------------------------------------------
AC_DEFUN([SPA_FEATURE_CHECK], [
	AC_LINK_IFELSE([
		AC_LANG_PROGRAM(
			[#include <zfeature_common.h>],
			[
				spa_feature_t f = $1 /* ; */
			]
		)],
		[AC_SUBST(HAVE_$1, 1) AC_SUBST($1, "$1") $2],
		[AC_SUBST(HAVE_$1, 0) AC_SUBST($1, ["# $1 not defined"]) $3]
	)
])

dnl This is the list of features that might not be
dnl supported by the oldest ZFS we support.  We check
dnl for their presence so we can generate the enum for
dnl spa_feature_t appropriately.
dnl
dnl USEROBJ_ACCOUNTING and PROJECT_QUOTA are separate so
dnl we can also generate the enum for zfs_userquota_prop_t
dnl appropriately.
dnl
dnl Current oldest supported ZFS is for FreeBSD 12.3
m4_foreach([feature],
	   [[SPA_FEATURE_ENCRYPTION],
	    [SPA_FEATURE_RESILVER_DEFER],
	    [SPA_FEATURE_BOOKMARK_V2],
	    [SPA_FEATURE_REDACTION_BOOKMARKS],
	    [SPA_FEATURE_REDACTED_DATASETS],
	    [SPA_FEATURE_BOOKMARK_WRITTEN],
	    [SPA_FEATURE_LOG_SPACEMAP],
	    [SPA_FEATURE_LIVELIST],
	    [SPA_FEATURE_DEVICE_REBUILD],
	    [SPA_FEATURE_ZSTD_COMPRESS],
	    [SPA_FEATURE_DRAID],
	    [SPA_FEATURE_ZILSAXATTR],
	    [SPA_FEATURE_HEAD_ERRLOG],
	    [SPA_FEATURE_BLAKE3]],
	   [SPA_FEATURE_CHECK(feature)]
)
SPA_FEATURE_CHECK([SPA_FEATURE_USEROBJ_ACCOUNTING],
	[
		AC_SUBST(ZFS_PROP_USEROBJUSED, "ZFS_PROP_USEROBJUSED")
		AC_SUBST(ZFS_PROP_USEROBJQUOTA, "ZFS_PROP_USEROBJQUOTA")
		AC_SUBST(ZFS_PROP_GROUPOBJUSED, "ZFS_PROP_GROUPOBJUSED")
		AC_SUBST(ZFS_PROP_GROUPOBJQUOTA, "ZFS_PROP_GROUPOBJQUOTA")
	],
	[
		AC_SUBST(ZFS_PROP_USEROBJUSED, ["# ZFS_PROP_USEROBJUSED not defined"])
		AC_SUBST(ZFS_PROP_USEROBJQUOTA, ["# ZFS_PROP_USEROBJQUOTA not defined"])
		AC_SUBST(ZFS_PROP_GROUPOBJUSED, ["# ZFS_PROP_GROUPOBJUSED not defined"])
		AC_SUBST(ZFS_PROP_GROUPOBJQUOTA, ["# ZFS_PROP_GROUPOBJQUOTA not defined"])
	]
)
SPA_FEATURE_CHECK([SPA_FEATURE_PROJECT_QUOTA],
	[
		AC_SUBST(ZFS_PROP_PROJECTUSED, "ZFS_PROP_PROJECTUSED")
		AC_SUBST(ZFS_PROP_PROJECTQUOTA, "ZFS_PROP_PROJECTQUOTA")
		AC_SUBST(ZFS_PROP_PROJECTOBJUSED, "ZFS_PROP_PROJECTOBJUSED")
		AC_SUBST(ZFS_PROP_PROJECTOBJQUOTA, "ZFS_PROP_PROJECTOBJQUOTA")
	],
	[
		AC_SUBST(ZFS_PROP_PROJECTUSED, ["# ZFS_PROP_PROJECTUSED not defined"])
		AC_SUBST(ZFS_PROP_PROJECTQUOTA, ["# ZFS_PROP_PROJECTQUOTA not defined"])
		AC_SUBST(ZFS_PROP_PROJECTOBJUSED, ["# ZFS_PROP_PROJECTOBJUSED not defined"])
		AC_SUBST(ZFS_PROP_PROJECTOBJQUOTA, ["# ZFS_PROP_PROJECTOBJQUOTA not defined"])
	]
)


SECTION_TITLE([RUNNING TESTS ON zfs_ioctl.h HEADER])

AC_MSG_NOTICE([Verifying existence of zfs_ioctl.h])


AC_CHECK_HEADERS([sys/zfs_ioctl.h],
	[AC_SUBST(HAVE_ZFS_IOCTL_HEADER, 1) AC_MSG_NOTICE([OK])],
	[AC_SUBST(HAVE_ZFS_IOCTL_HEADER, 0) AC_MSG_NOTICE([NOT FOUND])],
	[]
)


SECTION_TITLE([RUNNING TESTS ON nvpair.h HEADER])


AC_CHECK_HEADERS([sys/nvpair.h],
	[AC_SUBST(HAVE_NVPAIR_IN_SYS, 1) AC_MSG_NOTICE([Nvpair.h header under sys])],
	[AC_SUBST(HAVE_NVPAIR_IN_SYS, 0) AC_MSG_NOTICE([Nvpair.h header not under sys subdirectory])],
	[]
)


SECTION_TITLE([RUNNING TESTS ON sys/zfs_context_userland.h HEADER])

exists=0
AC_CHECK_HEADERS([sys/zfs_context_userland.h],
	[exists=1],
	[AC_SUBST(HAVE_THREAD_INIT_FINI, 0)],
	[]
)
if test "$exists" == 1
then
	dnl Verifying existence of thread_init and thread_fini
	AC_MSG_NOTICE([Verifying existence of thread_init and thread_fini])
	AC_LINK_IFELSE([AC_LANG_PROGRAM(
			[#include <sys/zfs_context_userland.h>],
			[
				thread_init();
				thread_fini();
			]
		)],
		[AC_SUBST(HAVE_THREAD_INIT_FINI, 1) AC_MSG_NOTICE([OK])],
		[AC_SUBST(HAVE_THREAD_INIT_FINI, 0) AC_MSG_NOTICE([NOT FOUND])]
	)
fi

SECTION_TITLE([GENERATING FILES])

AC_CONFIG_FILES([pxd/config.pxi])
AC_CONFIG_FILES([pxd/sendflags.pxi])
AC_CONFIG_FILES([pxd/spa_features.pxi])
AC_CONFIG_FILES([pxd/userquota_props.pxi])
AC_CONFIG_FILES([pxd/zpool_status.pxi])
AC_CONFIG_FILES([config.py])
AC_CONFIG_FILES([Makefile])

AC_OUTPUT
