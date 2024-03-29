m4_include([ax_bam.m4])
m4_include([ax_check_zlib.m4])

define([svnversion], esyscmd([sh -c "svnversion|tr -d '\n'"]))dnl

AC_INIT([tophat],[1.4.1],[cole@cs.umd.edu])
AC_DEFINE(SVN_REVISION, "svnversion", [SVN Revision])

AC_CONFIG_SRCDIR([config.h.in])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_AUX_DIR([build-aux])

AM_INIT_AUTOMAKE

 # Make sure CXXFLAGS is defined so that AC_PROG_CXX doesn't set it.
CXXFLAGS="$CXXFLAGS"
CFLAGS="$CFLAGS"

AC_LANG(C)

AC_ARG_ENABLE(debug,
	AC_HELP_STRING([--enable-debug],[build debug library (default=no)]),,enable_debug=no)

# declare some precious variables
AC_ARG_VAR(PYTHON, [python program])
AC_ARG_VAR(PERL, [perl program])
AC_ARG_VAR(TOPHAT_SHELL, [bash program])

CFLAGS="$CFLAGS"
CXXFLAGS="$CXXFLAGS"

# check the platform
AC_CANONICAL_HOST

# Checks for programs.
AC_LANG(C++)
AC_PROG_CC
AC_PROG_CXX
AC_PROG_AWK
AC_PROG_INSTALL
AC_PROG_RANLIB
AC_CHECK_TOOL(AR,ar,false)
AC_PATH_PROG(PERL, [perl], [:])
AC_PATH_PROG(TOPHAT_SHELL, bash, no)
AM_PATH_PYTHON([2.4])
AX_BAM
AX_CHECK_ZLIB()

# Checks for header files.
AC_CHECK_HEADERS([stdlib.h string.h unistd.h])

# Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADERS([stdlib.h string.h unistd.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_HEADER_STDBOOL
AC_C_CONST
AC_SYS_LARGEFILE

# Checks for libraries.
AC_CHECK_LIB([z], [gzread])

# Checks for library functions.
AC_FUNC_MALLOC
AC_CHECK_FUNCS([memset strdup strrchr strtol strsep])

# ---------------------------------------------------------------------
# Debug and profile
# ---------------------------------------------------------------------

# set CFLAGS and CXXFLAGS
user_CFLAGS="${CXXFLAGS}"
generic_CFLAGS="-Wall -Wno-strict-aliasing"
ext_CFLAGS=""
debug_CFLAGS="-g"
#echo "${host_cpu}-${host_os}"
case "${host_cpu}-${host_os}" in
   i*86-*linux*)
    ext_CFLAGS="-march=i686";;
   i*86-darwin*)
    AC_ARG_ENABLE(universal, [  --enable-universal      enable universal binary (Intel Apple only)],
				  [is_uni=1], [is_uni=0])
	case $is_uni in
	  1) CFLAGS="-m64"
	     AC_COMPILE_IFELSE([AC_LANG_PROGRAM],
						   [ext_CFLAGS="-arch x86_64 -arch i386 -arch ppc64 -arch ppc"],
						   [ext_CFLAGS="-arch i386 -arch ppc"]);;
	  0) CFLAGS="-m64"
	  	 AC_COMPILE_IFELSE([AC_LANG_PROGRAM], [ext_CFLAGS="-arch x86_64"], []);;
	esac;;
  *)
    AC_MSG_CHECKING([if gcc accepts -m64])
    CFLAGS="-m64"
    AC_COMPILE_IFELSE([AC_LANG_PROGRAM], [ext_CFLAGS="-m64"; AC_MSG_RESULT([yes])],
					  [ext_CFLAGS="-D_FILE_OFFSET_BITS=64"; AC_MSG_RESULT([no])]);;
esac

AC_ARG_ENABLE(intel64,      [  --enable-intel64        optimize for Intel64 CPU such as Xeon and Core2],
			  [ext_CFLAGS="${ext_CFLAGS} -mtune=nocona"], [])

AC_ARG_ENABLE([debug],
              [AS_HELP_STRING([--enable-debug],
	                      [enable debugging info (default is no)])],
              [], [enable_debug=no])
AC_ARG_ENABLE([optim],
              [AS_HELP_STRING([--enable-optim@<:@=0|1|2|3@:>@],
	                      [set optimization level (default is 3)])],
              [if test "x$enable_optim" = xyes; then enable_optim=3; fi],
              [enable_optim=3])

AS_IF([test "x$enable_optim" != xno], [ext_CFLAGS="$ext_CFLAGS -O$enable_optim"])	
AS_IF([test "x$enable_debug" = xyes],
      [debug_CFLAGS="-DDEBUG"; ext_CFLAGS="-arch i386"],
      [debug_CFLAGS="-DNDEBUG"])
	  
CFLAGS="${generic_CFLAGS} ${ext_CFLAGS} ${user_CFLAGS} ${debug_CFLAGS}"
CXXFLAGS="$CFLAGS $BAM_CPPFLAGS"

AM_CXXFLAGS="$CXXFLAGS -I./SeqAn-1.2"

AC_SUBST(CC)
AC_SUBST(AM_CXXFLAGS)
AC_SUBST(AM_CFLAGS)

AM_INIT_AUTOMAKE([-Wall foreign tar-pax foreign])

# makefiles to configure
AC_CONFIG_FILES([Makefile src/Makefile])

# make it happen
AC_OUTPUT

# dump some configuration confirmations
echo \
"
-- ${PACKAGE_STRING} Configuration Results --
  C compiler:          ${CC} ${CFLAGS}
  C++ compiler:        ${CXX} ${CXXFLAGS}"

if test x"${GCC}" = x"yes" ; then
   gcc_version=`${CC} --version | head -n 1`
   echo "  GCC version:         ${gcc_version}"
else
   gcc_version=''
fi

echo \
"  Host System type:    ${host}
  Install prefix:      ${prefix}
  Install eprefix:     ${exec_prefix}

  See config.h for further configuration information.
  Email <${PACKAGE_BUGREPORT}> with questions and bug reports.
"

if test x"${PYTHON}" = x":"  ||  ! test -x "${PYTHON}"; then
  echo "WARNING! python was not found and is required to run tophat"
  echo "  Please install python and point configure to the installed location"
fi
