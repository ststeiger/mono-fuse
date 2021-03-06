AC_INIT([mono-fuse], 0.4.3, [jonpryor@vt.edu])
AC_CANONICAL_SYSTEM
AC_PREREQ(2.53)
AM_INIT_AUTOMAKE([1.9 tar-ustar])
AM_MAINTAINER_MODE
AM_CONFIG_HEADER(config.h)
AC_PROG_CC
AC_LIBTOOL_DLOPEN
AM_PROG_LIBTOOL

ASSEMBLY_VERSION=0.4.3.0

AC_PATH_PROG(MONO, mono)
AC_PATH_PROG(CSC, gmcs)

if test "x$MONO" = "x" ; then
	AC_MSG_ERROR([Can't find "mono" in your PATH])
fi

if test "x$CSC" = "x" ; then
  AC_MSG_ERROR([Can't find "gmcs" in your PATH])
fi
AC_SUBST(PATH)
AC_SUBST(LD_LIBRARY_PATH)

dnl Find pkg-config
AC_PATH_PROG(PKG_CONFIG, pkg-config, no)
if test "x$PKG_CONFIG" = "xno"; then
        AC_MSG_ERROR([You need to install pkg-config])
fi

FUSE_REQUIRED_VERSION=2.5.2
GLIB_REQUIRED_VERSION=1.3.11
MONO_REQUIRED_VERSION=1.1.13
PKG_CHECK_MODULES(MONO, mono >= $MONO_REQUIRED_VERSION)

PKG_CHECK_MODULES(FUSE, fuse >= $FUSE_REQUIRED_VERSION)
PKG_CHECK_MODULES(GLIB, glib-2.0 >= $GLIB_REQUIRED_VERSION)

FUSE_CFLAGS=`$PKG_CONFIG --cflags fuse`
FUSE_LIBS=`$PKG_CONFIG --libs fuse`
GLIB_CFLAGS=`$PKG_CONFIG --cflags glib-2.0`
GLIB_LIBS=`$PKG_CONFIG --libs glib-2.0`

AC_SUBST(FUSE_CFLAGS)
AC_SUBST(FUSE_LIBS)
AC_SUBST(GLIB_CFLAGS)
AC_SUBST(GLIB_LIBS)

if test `uname -s` = "Darwin"; then
	LIB_PREFIX=
	LIB_SUFFIX=.dylib
else
	LIB_PREFIX=.so
	LIB_SUFFIX=
fi

dnl hard dependencies
MONODOC_REQUIRED_VERSION=1.0

PKG_CHECK_MODULES(MONODOC, monodoc >= $MONODOC_REQUIRED_VERSION, has_monodoc=yes, has_monodoc=no)
AC_SUBST(MONODOC_LIBS)

MONO_NUNIT_REQUIRED_VERSION=1.1.8
AC_ARG_ENABLE(nunit,
	AC_HELP_STRING([--enable-nunit],
		[enable support for NUnit [default=no]]),
		[PKG_CHECK_MODULES(MONO_NUNIT, mono-nunit >= $MONO_NUNIT_REQUIRED_VERSION, enable_nunit=yes, enable_nunit=no)],
		enable_nunit=no)

AM_CONDITIONAL(ENABLE_NUNIT, test x$enable_nunit = xyes)

AC_MSG_CHECKING([if Mono.Posix.dll has Mono.Unix.Native.NativeConvert.TryCopy methods])
cat > try-copy.cs << EOF
using System;
using Mono.Unix.Native;
class Test {
	public static void Main ()
	{
		Stat stat;
		NativeConvert.TryCopy (IntPtr.Zero, out stat);
		NativeConvert.TryCopy (ref stat, IntPtr.Zero);

		Statvfs statvfs;
		NativeConvert.TryCopy (IntPtr.Zero, out statvfs);
		NativeConvert.TryCopy (ref statvfs, IntPtr.Zero);

		Utimbuf utimbuf;
		NativeConvert.TryCopy (IntPtr.Zero, out utimbuf);
		NativeConvert.TryCopy (ref utimbuf, IntPtr.Zero);
	}
}
EOF
if gmcs try-copy.cs -r:Mono.Posix.dll 2>/dev/null >/dev/null ; then
	AC_MSG_RESULT([yes])
	HAVE_MONO_UNIX_NATIVE_COPY_FUNCS=1
else
	AC_MSG_RESULT([no])
fi
rm try-copy.*

AC_MSG_CHECKING([if Mono.Posix.dll has Mono.Unix.Native.NativeConvert.TryCopy(Flock) support])
cat > try-copy.cs << EOF
using System;
using Mono.Unix.Native;
class Test {
	public static void Main ()
	{
		Flock f;
		NativeConvert.TryCopy (IntPtr.Zero, out f);
		NativeConvert.TryCopy (ref f, IntPtr.Zero);
	}
}
EOF
if gmcs try-copy.cs -r:Mono.Posix.dll 2>/dev/null >/dev/null ; then
	AC_MSG_RESULT([yes])
	HAVE_MONO_UNIX_NATIVE_COPY_FLOCK=1
else
	AC_MSG_RESULT([no])
fi
rm try-copy.*

CSC_FLAGS=
MFH_CFLAGS=

if test "x$HAVE_MONO_UNIX_NATIVE_COPY_FUNCS" = "x1" ; then
	CSC_FLAGS="$CSC_FLAGS -d:HAVE_MONO_UNIX_NATIVE_COPY_FUNCS"
	MFH_CFLAGS="-DHAVE_MONO_UNIX_NATIVE_COPY_FUNCS"
fi

if test "x$HAVE_MONO_UNIX_NATIVE_COPY_FLOCK" = "x1" ; then
	CSC_FLAGS="$CSC_FLAGS -d:HAVE_MONO_UNIX_NATIVE_COPY_FLOCK"
	MFH_CFLAGS="$MFH_CFLAGS -DHAVE_MONO_UNIX_NATIVE_COPY_FLOCK"
fi

AC_SUBST(LIB_SUFFIX)
AC_SUBST(LIB_PREFIX)
AC_SUBST(ASSEMBLY_VERSION)
AC_SUBST(CSC_FLAGS)
AC_SUBST(MFH_CFLAGS)

MF_DIR='$(prefix)/lib/mono-fuse'
AC_SUBST(MF_DIR)

AC_PATH_PROG(MDOC, mdoc, no)
if test "x$MDOC" = "xno" ; then
  enable_docs=no
else
  enable_docs=yes
fi
AC_SUBST(MDOC)

AM_CONDITIONAL(ENABLE_DOCS, test "x$enable_docs" = "xyes")

AC_OUTPUT([
Makefile
mono-fuse.spec
doc/Makefile
example/Makefile
example/HelloFS/Makefile
example/RedirectFS/Makefile
lib/Makefile
lib/mono-fuse/Makefile
lib/pkgconfig/Makefile
lib/pkgconfig/mono-fuse.pc
src/Makefile
src/Mono.Fuse/AssemblyInfo.cs
src/Mono.Fuse/Makefile
src/MonoFuseHelper/Makefile
])

echo ""
echo "Configuration summary"
echo ""
echo "   * Installation prefix = $prefix"
echo "   * C# compiler = $CSC"
echo "   * NUnit support: $enable_nunit"
echo ""

