
AC_INIT([virt-viewer],[0.5.7])
AC_CONFIG_SRCDIR(src/virt-viewer-main.c)
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_HEADERS([config.h])
dnl Make automake keep quiet about wildcards & other GNUmake-isms
AM_INIT_AUTOMAKE([-Wno-portability])
AC_CANONICAL_HOST

# Use the silent-rules feature when possible.
m4_ifndef([AM_SILENT_RULES], [m4_define([AM_SILENT_RULES],[])])
AM_SILENT_RULES([yes])

GLIB2_REQUIRED=2.22.0
LIBXML2_REQUIRED="2.6.0"
LIBVIRT_REQUIRED="0.10.0"
GTK2_REQUIRED="2.18.0"
GTK3_REQUIRED="3.0"
GTK_VNC1_REQUIRED="0.3.8"
GTK_VNC2_REQUIRED="0.4.0"
SPICE_GTK_REQUIRED="0.20"
SPICE_PROTOCOL_REQUIRED="0.10.1"

AC_MSG_CHECKING([for native Win32])
case "$host_os" in
     *mingw*|*cygwin*)
        os_win32=yes
        case "$host" in
            amd64*|x86_64*)
                WIXL_ARCH="x64"
            ;;
            *)
                WIXL_ARCH="x86"
            ;;
        esac
        AC_SUBST(WIXL_ARCH)
        ;;
     *)
        os_win32=no
        ;;
esac
AC_MSG_RESULT([$os_win32])
AM_CONDITIONAL([OS_WIN32],[test "$os_win32" = "yes"])

AC_PROG_CC
AM_PROG_CC_C_O
LT_INIT

AC_CHECK_PROGS(ICOTOOL, [icotool], [icotool])

AS_IF([test "x$os_win32" = "xyes"], [
     AC_CHECK_TOOL(WINDRES, [windres])

     if test -z "$WINDRES" ; then
       AC_MSG_ERROR("windres is required to compile virt-viewer on this platform")
     fi
])

AC_CONFIG_LIBOBJ_DIR([src])

AC_DEFINE([_GNU_SOURCE], [], [Enable GNU extensions])

# Autoconf 2.61a.99 and earlier don't support linking a file only
# in VPATH builds.  But since GNUmakefile is for maintainer use
# only, it does not matter if we skip the link with older autoconf.
# Automake 1.10.1 and earlier try to remove GNUmakefile in non-VPATH
# builds, so use a shell variable to bypass this.
GNUmakefile=GNUmakefile
m4_if(m4_version_compare([2.61a.100],
        m4_defn([m4_PACKAGE_VERSION])), [1], [],
      [AC_CONFIG_LINKS([$GNUmakefile:$GNUmakefile], [],
        [GNUmakefile=$GNUmakefile])])

VIRT_VIEWER_COMPILE_WARNINGS(maximum)

GETTEXT_PACKAGE=virt-viewer
AC_SUBST(GETTEXT_PACKAGE)
AC_DEFINE_UNQUOTED([GETTEXT_PACKAGE],"$GETTEXT_PACKAGE", [GETTEXT package name])

AM_GLIB_GNU_GETTEXT
IT_PROG_INTLTOOL([0.35.0])

PKG_PROG_PKG_CONFIG
GLIB_MKENUMS=`$PKG_CONFIG --variable=glib_mkenums glib-2.0`
AC_SUBST(GLIB_MKENUMS)

PKG_CHECK_MODULES(GLIB2, glib-2.0 >= $GLIB2_REQUIRED gthread-2.0 gmodule-export-2.0)
PKG_CHECK_MODULES(LIBXML2, libxml-2.0 >= $LIBXML2_REQUIRED)

AC_ARG_WITH([libvirt],
    AS_HELP_STRING([--without-libvirt], [Ignore presence of libvirt and disable it]))

AS_IF([test "x$with_libvirt" != "xno"],
      [PKG_CHECK_MODULES(LIBVIRT,
                         [libvirt >= $LIBVIRT_REQUIRED],
                         [have_libvirt=yes], [have_libvirt=no])],
      [have_libvirt=no])

AS_IF([test "x$have_libvirt" = "xyes"],
      [AC_DEFINE([HAVE_LIBVIRT], 1, [Have libvirt?])],
      [AS_IF([test "x$with_libvirt" = "xyes"],
             [AC_MSG_ERROR([libvirt requested but not found])
      ])
])
AM_CONDITIONAL([HAVE_LIBVIRT], [test "x$have_libvirt" = "xyes"])

AC_MSG_CHECKING([which gtk+ version to compile against])
AC_ARG_WITH([gtk],
  [AS_HELP_STRING([--with-gtk=2.0|3.0],[which gtk+ version to compile against (default: 2.0)])],
  [case "$with_gtk" in
     2.0|3.0) ;;
     *) AC_MSG_ERROR([invalid gtk version specified]) ;;
   esac],
  [with_gtk=2.0])
AC_MSG_RESULT([$with_gtk])

case "$with_gtk" in
  2.0) GTK_API_VERSION=2.0
       GTK_REQUIRED=$GTK2_REQUIRED
       GTK_VNC_REQUIRED=$GTK_VNC1_REQUIRED
       GTK_VNC_API_VERSION=1.0
       SPICE_GTK_API_VERSION=2.0
       ;;
  3.0) GTK_API_VERSION=3.0
       GTK_REQUIRED=$GTK3_REQUIRED
       GTK_VNC_REQUIRED=$GTK_VNC2_REQUIRED
       GTK_VNC_API_VERSION=2.0
       SPICE_GTK_API_VERSION=3.0
       ;;
esac

AC_SUBST([GTK_API_VERSION])
AC_SUBST([GTK_REQUIRED])
AC_SUBST([GTK_VNC_API_VERSION])
AM_CONDITIONAL([HAVE_GTK_2],[test "$with_gtk" = "2.0"])
AM_CONDITIONAL([HAVE_GTK_3],[test "$with_gtk" = "3.0"])

PKG_CHECK_MODULES(GTK, gtk+-$GTK_API_VERSION >= $GTK_REQUIRED)

AC_ARG_WITH([gtk-vnc],
    AS_HELP_STRING([--without-gtk-vnc], [Ignore presence of gtk-vnc and disable it]))

AS_IF([test "x$with_gtk_vnc" != "xno"],
      [PKG_CHECK_MODULES(GTK_VNC,
                         gtk-vnc-$GTK_VNC_API_VERSION >= $GTK_VNC_REQUIRED,
                         [have_gtk_vnc=yes], [have_gtk_vnc=no])],
      [have_gtk_vnc=no])

AS_IF([test "x$have_gtk_vnc" = "xyes"],
      [AC_DEFINE([HAVE_GTK_VNC], 1, [Have gtk-vnc?])],
      [AS_IF([test "x$with_gtk_vnc" = "xyes"],
             [AC_MSG_ERROR([gtk-vnc requested but not found])
      ])
])
AM_CONDITIONAL([HAVE_GTK_VNC], [test "x$have_gtk_vnc" = "xyes"])

AC_ARG_WITH([spice-gtk],
    AS_HELP_STRING([--without-spice-gtk], [Ignore presence of spice-gtk and disable it]))

AS_IF([test "x$with_spice_gtk" != "xno"],
      [PKG_CHECK_MODULES(SPICE_GTK,
                         [spice-client-gtk-$SPICE_GTK_API_VERSION >= $SPICE_GTK_REQUIRED],
                         [have_spice_gtk=yes], [have_spice_gtk=no])],
      [have_spice_gtk=no])

AS_IF([test "x$have_spice_gtk" = "xyes"],
      [PKG_CHECK_MODULES(SPICE_CONTROLLER, [spice-controller])]
      [PKG_CHECK_MODULES(SPICE_PROTOCOL, [spice-protocol >= $SPICE_PROTOCOL_REQUIRED])]
      [AC_DEFINE([HAVE_SPICE_GTK], 1, [Have spice-gtk?])],
      [AS_IF([test "x$with_spice_gtk" = "xyes"],
             [AC_MSG_ERROR([spice-gtk requested but not found])
      ])
])
AM_CONDITIONAL([HAVE_SPICE_GTK], [test "x$have_spice_gtk" = "xyes"])

AC_ARG_WITH([ovirt],
    AS_HELP_STRING([--without-ovirt], [Ignore presence of librest and disable oVirt support]))

AS_IF([test "x$with_ovirt" != "xno"],
      [PKG_CHECK_MODULES([OVIRT], [govirt-1.0 >= 0.1.0],
                         [have_ovirt=yes], [have_ovirt=no])],
      [have_ovirt=no])

AS_IF([test "x$have_ovirt" = "xyes"],
      [AC_DEFINE([HAVE_OVIRT], 1, [Have libgovirt?])],
      [AS_IF([test "x$with_ovirt" = "xyes"],
             [AC_MSG_ERROR([oVirt support requested but libgovirt not found])
      ])
])

dnl Decide if this platform can support the SSH tunnel feature.
AC_CHECK_HEADERS([sys/socket.h sys/un.h windows.h])
AC_CHECK_FUNCS([fork socketpair])


if test "x$have_gtk_vnc" != "xyes" && test "x$have_spice_gtk" != "xyes"; then
    AC_MSG_ERROR([At least one of spice or vnc must be used])
fi

AC_ARG_WITH([buildid],
    AS_HELP_STRING([--with-buildid=id], [Set additional build version details]))
AC_DEFINE_UNQUOTED([BUILDID], "$with_buildid", [Build version details])
if test "x$with_buildid" != x; then
    AC_SUBST([BUILDID], "-$with_buildid")
fi

major=`echo $PACKAGE_VERSION | cut -d. -f1`
minor=`echo $PACKAGE_VERSION | cut -d. -f2`
micro=`echo $PACKAGE_VERSION | cut -d. -f3`
buildid=`echo $with_buildid | cut -d. -f1`
if test "x$buildid" = x; then
   buildid=0
fi
build=`expr $micro \* 256 + $buildid`
WINDOWS_PRODUCTVERSION="$major.$minor.$build"
AC_SUBST([WINDOWS_PRODUCTVERSION])

AC_CONFIG_FILES([
    Makefile
    data/Makefile
    data/virt-viewer.wxs
    data/virt-viewer.nsis
    data/virt-viewer-debug.nsis
    icons/Makefile
    icons/16x16/Makefile
    icons/22x22/Makefile
    icons/24x24/Makefile
    icons/32x32/Makefile
    icons/48x48/Makefile
    icons/256x256/Makefile
    man/Makefile
    mingw-virt-viewer.spec
    po/Makefile.in
    src/Makefile
    src/virt-viewer.rc
    virt-viewer.spec
])
AC_OUTPUT

AC_MSG_NOTICE([])
AC_MSG_NOTICE([Configuration summary])
AC_MSG_NOTICE([=====================])
AC_MSG_NOTICE([])
AC_MSG_NOTICE([ Features:])
AC_MSG_NOTICE([])
AC_MSG_NOTICE([      Gtk: $with_gtk])
AC_MSG_NOTICE([])
AC_MSG_NOTICE([ Libraries:])
AC_MSG_NOTICE([])
AC_MSG_NOTICE([       GLIB2: $GLIB2_CFLAGS $GLIB2_LIBS])
AC_MSG_NOTICE([])
AC_MSG_NOTICE([         GTK: $GTK_CFLAGS $GTK_LIBS])
AC_MSG_NOTICE([])
AC_MSG_NOTICE([     GTK_VNC: $GTK_VNC_CFLAGS $GTK_VNC_LIBS])
AC_MSG_NOTICE([])
AC_MSG_NOTICE([   SPICE_GTK: $SPICE_GTK_CFLAGS $SPICE_GTK_LIBS])
AC_MSG_NOTICE([])
AC_MSG_NOTICE([     LIBXML2: $LIBXML2_CFLAGS $LIBXML2_LIBS])
AC_MSG_NOTICE([])
AC_MSG_NOTICE([     LIBVIRT: $LIBVIRT_CFLAGS $LIBVIRT_LIBS])
AC_MSG_NOTICE([])
AC_MSG_NOTICE([       OVIRT: $OVIRT_CFLAGS $OVIRT_LIBS])
AC_MSG_NOTICE([])
