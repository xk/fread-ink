dnl  Copyright © 2003 Keith Packard
dnl 
dnl  Permission to use, copy, modify, distribute, and sell this software and its
dnl  documentation for any purpose is hereby granted without fee, provided that
dnl  the above copyright notice appear in all copies and that both that
dnl  copyright notice and this permission notice appear in supporting
dnl  documentation, and that the name of Keith Packard not be used in
dnl  advertising or publicity pertaining to distribution of the software without
dnl  specific, written prior permission.  Keith Packard makes no
dnl  representations about the suitability of this software for any purpose.  It
dnl  is provided "as is" without express or implied warranty.
dnl 
dnl  KEITH PACKARD DISCLAIMS ALL WARRANTIES WITH REGARD TO THIS SOFTWARE,
dnl  INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS, IN NO
dnl  EVENT SHALL KEITH PACKARD BE LIABLE FOR ANY SPECIAL, INDIRECT OR
dnl  CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE,
dnl  DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER
dnl  TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR
dnl  PERFORMANCE OF THIS SOFTWARE.
dnl
dnl Process this file with autoconf to create configure.

AC_PREREQ([2.57])

AC_INIT([libXfont],
	1.4.1,
	[https://bugs.freedesktop.org/enter_bug.cgi?product=xorg],
	libXfont)
dnl
dnl This is the package version number, not the shared library
dnl version.  This same version number must appear in Xfont.h
dnl Yes, it is a pain to synchronize version numbers.  Unfortunately, it's
dnl not possible to extract the version number here from Xfont.h
dnl
AM_INIT_AUTOMAKE([foreign dist-bzip2])
AM_CONFIG_HEADER(config.h)
AC_CONFIG_HEADERS([include/X11/fonts/fontconf.h])

# Require xorg-macros version >= 1.3.0 for XORG_DEFAULT_OPTIONS
m4_ifndef([XORG_MACROS_VERSION], [AC_FATAL([must install xorg-macros 1.2 or later before running autoconf/autogen])])
XORG_MACROS_VERSION(1.3)

# Check for progs
AC_PROG_CC
AC_PROG_LIBTOOL
dnl If the first PKG_CHECK_MODULES appears inside a conditional, pkg-config
dnl must first be located explicitly.
PKG_PROG_PKG_CONFIG

XORG_DEFAULT_OPTIONS

# xmlto is used to convert doc/fontlib.xml from DocBook to PDF/HTML
AC_ARG_VAR([XMLTO], [Path to xmlto command])
AC_PATH_PROG([XMLTO], [xmlto])
AM_CONDITIONAL([HAVE_XMLTO], [test "x$XMLTO" != "x"])

# Option to enable support for building developer internals docs
AC_ARG_ENABLE(devel-docs,
	AS_HELP_STRING([--enable-devel-docs],
	    [Build internals documentation for developers (default: enabled)]),
	    [DEVEL_DOCS=$enableval], [DEVEL_DOCS=yes])
AM_CONDITIONAL([MAKE_DEVEL_DOCS], [test "x$DEVEL_DOCS" = "xyes"])

# Look for headers
AC_CHECK_HEADERS([endian.h poll.h sys/poll.h])
AC_CHECK_FUNCS([poll])

m4_ifdef([AS_HELP_STRING], , [m4_define([AS_HELP_STRING], m4_defn([AC_HELP_STRING]))])

#
# select libraries to include
#

XFONT_FONTFILE=no

#
# Scalable fonts in files:
# 	FreeType
#	Bitstream Speedo rasterizer
#
AC_ARG_ENABLE(freetype, [ --disable-freetype],[XFONT_FREETYPE=$enableval],[XFONT_FREETYPE=yes])
AM_CONDITIONAL(XFONT_FREETYPE, [test x$XFONT_FREETYPE = xyes])
if test x$XFONT_FREETYPE = xyes; then
	AC_DEFINE(XFONT_FREETYPE,1,[Support FreeType rasterizer for nearly all font file formats])
	AC_ARG_WITH(freetype-config, [  --with-freetype-config=PROG   Use FreeType configuration program PROG], freetype_config=$withval, freetype_config=auto)
	
	if test "$freetype_config" = "auto" ; then
		PKG_CHECK_MODULES(FREETYPE, freetype2, 
			freetype_config=no, freetype_config=yes)
	fi

	if test "$freetype_config" = "yes"; then 
		AC_PATH_PROG(ft_config,freetype-config,no)
		if test "$ft_config" = "no"; then
			AC_MSG_ERROR([You must have freetype installed; see http://www.freetype.org/])
		fi
	else
		ft_config="$freetype_config"
	fi

	if test "$freetype_config" != "no"; then 	
		FREETYPE_CFLAGS="`$ft_config --cflags`"
		FREETYPE_LIBS="`$ft_config --libs`"
	fi
	FREETYPE_REQUIRES="freetype2"
	XFONT_FONTFILE=yes
else
	FREETYPE_CFLAGS=""
	FREETYPE_LIBS=""
	FREETYPE_REQUIRES=""
fi
AC_SUBST(FREETYPE_LIBS)
AC_SUBST(FREETYPE_CFLAGS)
AC_SUBST(FREETYPE_REQUIRES)

#
# Bitmap fonts in files:
#	Builtins (gzipped pcf format in .text segment)
#	PCF
#	BDF
#	SNF
#

AC_DEFINE(X_GZIP_FONT_COMPRESSION,1,[Support gzip for bitmap fonts])
X_GZIP_FONT_COMPRESSION=1
AC_SUBST(X_GZIP_FONT_COMPRESSION)
AC_CHECK_LIB(z, gzopen, [Z_LIBS=-lz], AC_MSG_ERROR([*** zlib is required]))

AC_ARG_WITH(bzip2, 
	AS_HELP_STRING([--with-bzip2], 
			[Support bzip2 compressed bitmap fonts]),
	[], [with_bzip2=no])
if test "x$with_bzip2" = xyes; then
	AC_CHECK_LIB(bz2, BZ2_bzopen, [Z_LIBS="$Z_LIBS -lbz2"],
		AC_MSG_ERROR([*** libbz2 is required for bzip2 support]))
	AC_DEFINE(X_BZIP2_FONT_COMPRESSION,1,[Support bzip2 for bitmap fonts])
fi
AM_CONDITIONAL(X_BZIP2_FONT_COMPRESSION, [test "x$with_bzip2" = xyes ])
AC_SUBST(Z_LIBS)

AC_ARG_ENABLE(builtins, [ --disable-builtins ], [XFONT_BUILTINS=$enableval], [XFONT_BUILTINS=yes])
AM_CONDITIONAL(XFONT_BUILTINS, [test "x$XFONT_BUILTINS" = xyes ])
if test "x$XFONT_BUILTINS" = xyes; then
	AC_DEFINE(XFONT_BUILTINS,1,[Support built-in fonts])
	XFONT_BITMAP=yes
fi

AC_ARG_ENABLE(pcfformat, [ --disable-pcfformat ], [XFONT_PCFFORMAT=$enableval], [XFONT_PCFFORMAT=yes])
AM_CONDITIONAL(XFONT_PCFFORMAT, [test "x$XFONT_PCFFORMAT" = xyes])
if test "x$XFONT_PCFFORMAT" = xyes; then
	AC_DEFINE(XFONT_PCFFORMAT,1,[Support pcf format bitmap font files])
	XFONT_BITMAP=yes
fi

AC_ARG_ENABLE(bdfformat, [ --disable-bdfformat ], [XFONT_BDFFORMAT=$enableval], [XFONT_BDFFORMAT=yes])
AM_CONDITIONAL(XFONT_BDFFORMAT, [test "x$XFONT_BDFFORMAT" = xyes])
if test "x$XFONT_BDFFORMAT" = xyes; then
	AC_DEFINE(XFONT_BDFFORMAT,1,[Support bdf format bitmap font files])
	XFONT_BITMAP=yes
fi

AC_ARG_ENABLE(snfformat, [ --disable-snfformat ], [XFONT_SNFFORMAT=$enableval], [XFONT_SNFFORMAT=yes])
AM_CONDITIONAL(XFONT_SNFFORMAT, [test "x$XFONT_SNFFORMAT" = xyes])
if test "x$XFONT_SNFFORMAT" = xyes; then
	AC_DEFINE(XFONT_SNFFORMAT,1,[Support snf format bitmap font files])
	XFONT_BITMAP=yes
fi

AM_CONDITIONAL(XFONT_BITMAP, [test "x$XFONT_BITMAP" = xyes ])
if test "x$XFONT_BITMAP" = xyes; then
	AC_DEFINE(XFONT_BITMAP,1,[Support bitmap font files])
	XFONT_FONTFILE=yes
fi

AM_CONDITIONAL(XFONT_FONTFILE, [test "x$XFONT_FONTFILE" = xyes])
if test "x$XFONT_FONTFILE" = xyes; then
	AC_DEFINE(XFONT_FONTFILE,1,[Support fonts in files])
fi

#
# Support connection to font servers?
#

AC_ARG_ENABLE(fc, [ --disable-fc ],[XFONT_FC=$enableval],[XFONT_FC=yes])
AM_CONDITIONAL(XFONT_FC, [test "x$XFONT_FC" = xyes])
if test "x$XFONT_FC" = xyes; then
	AC_DEFINE(XFONT_FC,1,[Support the X Font Services Protocol])
fi


AC_CHECK_LIB(m, hypot, [MATH_LIBS=-lm
AC_SUBST(MATH_LIBS)], AC_MSG_ERROR([*** libm is required]))

PKG_CHECK_MODULES(XFONT, xproto xtrans fontsproto fontenc)

# Transport selection macro from xtrans.m4
XTRANS_CONNECTION_FLAGS

case $host_os in
    linux*)
        OS_CFLAGS="-D_XOPEN_SOURCE"
        ;;
    darwin*)
        OS_CFLAGS="-Wl,-flat_namespace"
	;;
    *)
        OS_CFLAGS=
        ;;
esac
AC_SUBST([OS_CFLAGS])

AC_OUTPUT([Makefile
	   src/Makefile
	   src/fontfile/Makefile
	   src/FreeType/Makefile
	   src/builtins/Makefile
	   src/bitmap/Makefile
	   src/fc/Makefile
	   src/util/Makefile
	   src/stubs/Makefile
           xfont.pc])
