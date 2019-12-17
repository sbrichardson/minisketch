AC_INIT([minisketch], [0.0.1], [http://github.com/sipa/minisketch/])

m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

AC_PREREQ(2.60)
AC_CONFIG_SRCDIR([src/minisketch.cpp])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_MACRO_DIR([build-aux/m4])
AM_INIT_AUTOMAKE([subdir-objects foreign])

LT_INIT
LT_LANG([C++])
AC_LANG([C++])

AC_PATH_PROG(CCACHE,ccache)

AC_ARG_ENABLE([ccache],
  [AS_HELP_STRING([--disable-ccache],
  [do not use ccache for building (default is to use if found)])],
  [use_ccache=$enableval],
  [use_ccache=auto])

AC_ARG_ENABLE(tests,
    AS_HELP_STRING([--enable-tests],[compile tests (default is yes)]),
    [use_tests=$enableval],
    [use_tests=yes])

AC_ARG_ENABLE(benchmark,
    AS_HELP_STRING([--enable-benchmark],[compile benchmark (default is no)]),
    [use_benchmark=$enableval],
    [use_benchmark=no])

m4_define([SUPPORTED_FIELDS], [2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64])
m4_foreach([FIELD], [SUPPORTED_FIELDS], [enable_field_[]FIELD=no])

AC_MSG_CHECKING([which field sizes to build])
AC_ARG_ENABLE([fields], AS_HELP_STRING([--enable-fields=LIST], [Comma-separated list of field sizes to build. Default=all. Available sizes:] m4_translit(m4_defn([SUPPORTED_FIELDS]), [,], [ ])), [], [enable_fields=SUPPORTED_FIELDS])
enable_fields=`echo "$enable_fields" | sed 's/,/ /g'`
if test "x$enable_fields" = xno || test "x$enable_fields" = x; then
    enable_fields=no
    AC_MSG_RESULT([(none)])
else
    for arg in $enable_fields
    do
	case $arg in m4_foreach([FIELD], [SUPPORTED_FIELDS], [
	    FIELD)
		enable_field_[]FIELD=yes
		AC_DEFINE(HAVE_FIELD_[]FIELD, [1],
		[Define to 1 if field size] FIELD [is enabled.])
		;;])
	    *)
		AC_MSG_RESULT([])
		AC_MSG_ERROR([unknown field size: $arg])
		;;
	esac
    done
    AC_MSG_RESULT([$enable_fields])
fi

dnl Check for pthread compile/link requirements
AX_PTHREAD

AX_CHECK_COMPILE_FLAG([-Werror],[CXXFLAG_WERROR="-Werror"],[CXXFLAG_WERROR=""])

AX_CXX_COMPILE_STDCXX([11], [noext], [mandatory], [nodefault])
enable_clmul=
AX_CHECK_COMPILE_FLAG([-mpclmul],[[enable_clmul=yes]],,[[$CXXFLAG_WERROR]])
if test x$enable_clmul = xyes; then
  CLMUL_CXXFLAGS="-mpclmul"
  AC_DEFINE(HAVE_CLMUL, 1, [Define this symbol if clmul instructions can be used])
fi


AC_MSG_CHECKING(for working clz builtins)
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([], [
  unsigned a = __builtin_clz(1);
  unsigned long b = __builtin_clzl(1);
  unsigned long long c = __builtin_clzll(1);
  ])],
  [
    AC_DEFINE(HAVE_CLZ, 1, [Define this symbol if clz builtins are present and working])
    AC_MSG_RESULT(yes)
  ],[
    AC_MSG_RESULT(no)
  ]
)

AX_CHECK_COMPILE_FLAG([-Wall],[WARN_CXXFLAGS="$WARN_CXXFLAGS -Wall"],,[[$CXXFLAG_WERROR]])
## Some compilers (gcc) ignore unknown -Wno-* options, but warn about all
## unknown options if any other warning is produced. Test the -Wfoo case, and
## set the -Wno-foo case if it works.
AX_CHECK_COMPILE_FLAG([-Wshift-count-overflow],[NOWARN_CXXFLAGS="$NOWARN_CXXFLAGS -Wno-shift-count-overflow"],,[[$CXXFLAG_WERROR]])

if test "x$use_ccache" != "xno"; then
  AC_MSG_CHECKING(if ccache should be used)
  if test x$CCACHE = x; then
    if test "x$use_ccache" = "xyes"; then
      AC_MSG_ERROR([ccache not found.]);
    else
      use_ccache=no
    fi
  else
    use_ccache=yes
    CC="$ac_cv_path_CCACHE $CC"
    CXX="$ac_cv_path_CCACHE $CXX"
  fi
  AC_MSG_RESULT($use_ccache)
fi
if test "x$use_ccache" = "xyes"; then
    AX_CHECK_COMPILE_FLAG([-Qunused-arguments],[NOWARN_CXXFLAGS="$NOWARN_CXXFLAGS -Qunused-arguments"],,[[$CXXFLAG_WERROR]])
fi

VERIFY_DEFINES=-DMINISKETCH_VERIFY
RELEASE_DEFINES=

AC_CONFIG_FILES([
    Makefile
])

AC_SUBST(CLMUL_CXXFLAGS)
AC_SUBST(WARN_CXXFLAGS)
AC_SUBST(NOWARN_CXXFLAGS)
AC_SUBST(VERIFY_DEFINES)
AC_SUBST(RELEASE_DEFINES)
AM_CONDITIONAL([ENABLE_CLMUL],[test x$enable_clmul = xyes])
AM_CONDITIONAL([USE_BENCHMARK], [test x"$use_benchmark" = x"yes"])
AM_CONDITIONAL([USE_TESTS], [test x"$use_tests" != x"no"])
AC_OUTPUT

