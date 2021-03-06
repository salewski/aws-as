dnl Process this file with autoconf to produce a configure script. -*-Autoconf-*-

dnl +-------------------------+
dnl |  Autotools Boilerplate  |
dnl +-------------------------+

dnl Ensure that a recent-enough version of Autoconf is being used
AC_PREREQ([2.69])

dnl Defines the following m4 macros, output variables, and preprocessor
dnl symbols. Only the m4 macros have the 'AC_' prefix:
dnl
dnl     [AC_] PACKAGE_NAME
dnl     [AC_]PACKAGE_TARNAME
dnl     [AC_]PACKAGE_VERSION
dnl     [AC_]PACKAGE_STRING     ("<PACKAGE> <VERSION>")
dnl     [AC_]PACKAGE_BUGREPORT
dnl     [AC_]PACKAGE_URL
dnl
AC_INIT([[aws-as]],
        [[0.2.1-snapshot]],
        [[Alan D. Salewski <salewski@att.net>]],
        [[aws-as]],
        [[https://github.com/salewski/aws-as]])

AC_CONFIG_AUX_DIR([build-aux])

dnl Indicate dir where our custom m4 macros may be found (future
dnl proofing; as of may not be used yet...)
AC_CONFIG_MACRO_DIR([autotools])

dnl Turn on all Automake warnings and report them as errors
AM_INIT_AUTOMAKE([-Wall -Werror foreign])


dnl AC_SUBST([DFLT_MAINTAINER_NAME])
dnl AC_SUBST([DFLT_MAINTAINER_EMAIL])
dnl AC_SUBST([DFLT_MAINTAINER_FULL])


dnl +------------------------+
dnl |  Checks for programs.  |
dnl +------------------------+

dnl makes $(LN_S) available in our Makefile.am files
AC_PROG_LN_S

dnl AC_PATH_PROG(AWK_PROG, awk)
dnl if test -z "$AWK_PROG"; then
dnl     AC_MSG_ERROR([awk not found])
dnl fi


dnl ## The 'aws' program as supplied by the 'aws/aws-cli' GitHub repo or the
dnl ## 'awscli' Debian package. Either version 1 or 2 should work fine for our
dnl ## purposes.
dnl ##
dnl ##     https://aws.amazon.com/cli/
dnl ##     https://github.com/aws/aws-cli
dnl ##
dnl ##     https://tracker.debian.org/pkg/awscli
dnl ##
dnl AC_PATH_PROG(AWS_PROG, aws)
dnl if test -z "$AWS_PROG"; then
dnl     AC_MSG_ERROR([aws not found])
dnl fi

## The 'aws-vault' program as supplied by 99designs. Used by `aws-as` for
## caching temporary AWS credentials, and related.
##
##     https://github.com/99designs/aws-vault
##     https://github.com/99designs/aws-vault/blob/master/USAGE.md
##     https://99designs.com/blog/engineering/aws-vault/
##
AC_PATH_PROG(AWS_VAULT_PROG, aws-vault)
if test -z "$AWS_VAULT_PROG"; then
    AC_MSG_ERROR([aws-vault not found])
fi

## We need bash >= 4.x (associative arrays were added in bash-4.0)
##
AC_PATH_PROG(BASH_SH, bash)
AS_IF([test -z "$BASH_SH"], [AC_MSG_ERROR([bash not found])],
      [["${BASH_SH}" -c 'test -n "${BASH_VERSINFO@<:@0@:>@:+set}" && test ${BASH_VERSINFO@<:@0@:>@]} -ge 4 && exit 0'], [:],
      [AC_MSG_ERROR([found version of bash is not new enough; bash 4.0 or newer is required])
])

AC_PATH_PROG([CAT], [cat])
if test -z "$CAT"; then
    AC_MSG_ERROR([cat not found])
fi

dnl AC_PATH_PROG([CHGRP], [chgrp])
dnl if test -z "$CHGRP"; then
dnl     AC_MSG_ERROR([chgrp not found])
dnl fi

dnl AC_PATH_PROG([CHOWN], [chown])
dnl if test -z "$CHOWN"; then
dnl     AC_MSG_ERROR([chown not found])
dnl fi

AC_PATH_PROG([CHMOD_PROG], [chmod])
if test -z "$CHMOD_PROG"; then
    AC_MSG_ERROR([chmod not found])
fi

dnl # See: https://en.wikipedia.org/wiki/CURL
dnl #      https://curl.haxx.se/
dnl #      https://github.com/curl/curl
dnl #
dnl AC_PATH_PROG([CURL_PROG], [curl])
dnl if test -z "$CURL_PROG"; then
dnl     AC_MSG_ERROR([curl not found])
dnl fi

dnl FIXME: We need a date(1) program capable of emitting time in UTC, in the
dnl        ISO-8601 format. We should test for it directly, but the following
dnl        invocation seems to be portable between the date(1) implementations
dnl        from the GNU coreutils and that from FreeBSD.
dnl            $ date -u -Iseconds
dnl
AC_PATH_PROG([DATE_PROG], [date])
if test -z "$DATE_PROG"; then
    AC_MSG_ERROR([date not found])
fi

AC_PATH_PROG([ECHO_PROG], [echo])
if test -z "$ECHO_PROG"; then
    AC_MSG_ERROR([echo not found])
fi

AC_PATH_PROG([EXPR_PROG], [expr])
if test -z "EXPR_PROG"; then
    AC_MSG_ERROR([expr not found])
fi

dnl AC_PATH_PROG([FIND], [find])
dnl if test -z "$FIND"; then
dnl     AC_MSG_ERROR([find not found]);dnl Ha!
dnl fi

dnl AC_PATH_PROG([GREP], [grep])
dnl if test -z "$GREP"; then
dnl     AC_MSG_ERROR([grep not found])
dnl fi

dnl AC_PATH_PROG([HEAD_PROG], [head])
dnl if test -z "$HEAD_PROG"; then
dnl     AC_MSG_ERROR([head not found])
dnl fi

dnl AC_PATH_PROG([ID_PROG], [id])
dnl if test -z "$ID_PROG"; then
dnl     AC_MSG_ERROR([id program not found])
dnl fi

dnl # jq - command line JSON parser and manipulation language
dnl #      see: https://github.com/stedolan/jq
dnl #
dnl AC_PATH_PROG([JQ_PROG], [jq])
dnl if test -z "$JQ_PROG"; then
dnl     AC_MSG_ERROR([jq program not found])
dnl fi

AC_PATH_PROG([MKDIR_PROG], [mkdir])
if test -z "$MKDIR_PROG"; then
    AC_MSG_ERROR([mkdir not found])
fi

AC_PATH_PROG([SED], [sed])
if test -z "$SED"; then
    AC_MSG_ERROR([sed not found])
fi

dnl AC_PATH_PROG([RM], [rm])
dnl if test -z "$RM"; then
dnl     AC_MSG_ERROR([rm not found])
dnl fi

dnl AC_PATH_PROG([RMDIR], [rmdir])
dnl if test -z "$RMDIR"; then
dnl     AC_MSG_ERROR([rmdir not found])
dnl fi

dnl AC_PATH_PROG([TR_PROG], [tr])
dnl if test -z "$TR_PROG"; then
dnl     AC_MSG_ERROR([tr not found])
dnl fi

AC_PATH_PROG([TRUE_PROG], [true])
if test -z "$TRUE_PROG"; then
    AC_MSG_ERROR([true not found])
fi

dnl AC_PATH_PROG([XARGS_PROG], [xargs])
dnl if test -z "$XARGS_PROG"; then
dnl     AC_MSG_ERROR([xargs not found])
dnl fi


dnl require /at least/ perl version 5.005
ads_PROG_PERL([5.005])

# pod2man comes with perl, which the above verifies that we have. Not yet sure
# if we can rely on that (distribution packaging may provide it in an optional
# package, for instance), so we check for it explicitly here. We use pod2man
# at build time to create some of our man pages.
AC_PATH_PROG([POD2MAN], [pod2man])
if test -z "$POD2MAN"; then
    AC_MSG_ERROR([pod2man program not found])
fi


dnl +------------------------+
dnl |  Checks for Libraries  |
dnl +------------------------+
dnl None!

dnl +----------------------+
dnl |  Checks for Headers  |
dnl +----------------------+
dnl None!

dnl +------------------------------------------------------------------+
dnl |  Checks for typedefs, structures, and compiler characteristics.  |
dnl +------------------------------------------------------------------+
dnl None!

dnl +---------------------------------+
dnl |  Checks for library functions.  |
dnl +---------------------------------+
dnl None!

dnl +---------------------+
dnl |  Autotools Output.  |
dnl +---------------------+

## The 'do_subst' Makefile var is the conventional name for manual
## filtering process that needs to be done at build time.
##
## Basically, any file we generate that is not a Makefile should be
## filtered at build time rather than at configure time; at build
## time, autotools variables will be fully expanded (well, expanded
## recursively, which amounts to the same thing) and allows any of the
## make vars to be overridden on the 'make' command line. This also
## avoids other kludgey hacks, such as eval'ing shell variables to
## expand them at configure time).
##
## Note that we replace all of the autoconf installation directory
## vars (see "Installation Directory Variables" in the autoconf
## manual) and most of the autoconf preset output variables (see
## "Preset Output Variables" in the autoconf manual). The exceptions
## to that latter are flag vars to the C/C++/Fortran compilers,
## linkers, etc. (CFLAGS and friends), and the 'configure_input' var,
## which we have to provide our own variation of. We also omit
## 'builddir' (which is always '.'); in this last case, what you
## probably want instead is 'abs_builddir', which we provide.
[
do_subst_command='do_subst = sed \
  -e '"'"'s,[@]PACKAGE[@],$(PACKAGE),g'"'"' \
  -e '"'"'s,[@]PACKAGE_NAME[@],$(PACKAGE_NAME),g'"'"' \
  -e '"'"'s,[@]PACKAGE_TARNAME[@],$(PACKAGE_TARNAME),g'"'"' \
  -e '"'"'s,[@]PACKAGE_VERSION[@],$(PACKAGE_VERSION),g'"'"' \
  -e '"'"'s,[@]PACKAGE_STRING[@],$(PACKAGE_STRING),g'"'"' \
  -e '"'"'s,[@]PACKAGE_BUGREPORT[@],$(PACKAGE_BUGREPORT),g'"'"' \
  -e '"'"'s,[@]PACKAGE_URL[@],$(PACKAGE_URL),g'"'"' \
  \
  -e '"'"'s,[@]VERSION[@],$(VERSION),g'"'"' \
  -e '"'"'s,[@]PATH_SEPARATOR[@],$(PATH_SEPARATOR),g'"'"' \
  -e '"'"'s,[@]configure_input[@],Generated from $@.in; do not edit by hand.,g'"'"' \
  -e "s,[@]BUILD_DATE[@],$$(date '"'"'+%Y-%m-%d %H:%M:%S'"'"'),g" \
  \
  -e '"'"'s,[@]bindir[@],$(bindir),g'"'"' \
  -e '"'"'s,[@]datadir[@],$(datadir),g'"'"' \
  -e '"'"'s,[@]exec_prefix[@],$(exec_prefix),g'"'"' \
  -e '"'"'s,[@]includedir[@],$(includedir),g'"'"' \
  -e '"'"'s,[@]infodir[@],$(infodir),g'"'"' \
  -e '"'"'s,[@]libdir[@],$(libdir),g'"'"' \
  -e '"'"'s,[@]libexecdir[@],$(libexecdir),g'"'"' \
  -e '"'"'s,[@]localstatedir[@],$(localstatedir),g'"'"' \
  -e '"'"'s,[@]mandir[@],$(mandir),g'"'"' \
  -e '"'"'s,[@]oldincludedir[@],$(oldincludedir),g'"'"' \
  -e '"'"'s,[@]prefix[@],$(prefix),g'"'"' \
  -e '"'"'s,[@]sbindir[@],$(sbindir),g'"'"' \
  -e '"'"'s,[@]sharedstatedir[@],$(sharedstatedir),g'"'"' \
  -e '"'"'s,[@]sysconfdir[@],$(sysconfdir),g'"'"' \
  \
  -e '"'"'s,[@]abs_builddir[@],$(abs_builddir),g'"'"' \
  -e '"'"'s,[@]abs_srcdir[@],$(abs_srcdir),g'"'"' \
  -e '"'"'s,[@]abs_top_builddir[@],$(abs_top_builddir),g'"'"' \
  -e '"'"'s,[@]srcdir[@],$(srcdir),g'"'"' \
  -e '"'"'s,[@]top_builddir[@],$(top_builddir),g'"'"' \
  -e '"'"'s,[@]top_srcdir[@],$(top_srcdir),g'"'"' \
  \
  -e '"'"'s,[@]PERL[@],$(PERL),g'"'"' \
  -e '"'"'s,[@]POD2MAN[@],$(POD2MAN),g'"'"' \
  \
  -e '"'"'s,[@]AWS_VAULT_PROG[@],$(AWS_VAULT_PROG),g'"'"' \
  -e '"'"'s,[@]BASH_SH[@],$(BASH_SH),g'"'"' \
  -e '"'"'s,[@]CAT[@],$(CAT),g'"'"' \
  -e '"'"'s,[@]CHMOD_PROG[@],$(CHMOD_PROG),g'"'"' \
  -e '"'"'s,[@]DATE_PROG[@],$(DATE_PROG),g'"'"' \
  -e '"'"'s,[@]ECHO_PROG[@],$(ECHO_PROG),g'"'"' \
  -e '"'"'s,[@]EXPR_PROG[@],$(EXPR_PROG),g'"'"' \
  -e '"'"'s,[@]MKDIR_PROG[@],$(MKDIR_PROG),g'"'"' \
  -e '"'"'s,[@]SED[@],$(SED),g'"'"' \
  -e '"'"'s,[@]TRUE_PROG[@],$(TRUE_PROG),g'"'"' \
'
]
AC_SUBST([do_subst_command])

# It's common for makefile scripts to need to "sed sanitize" the value of a
# variable in order to use it on the left side of a sed 's' (substitution)
# command. To obtain a "sed sanitized" copy of any variable, echo its value to:
#     $(sed_slsanitize)
#
# Example:
#     vers_sedslsanitized=$$(printf "%s\n" "-$(VERSION)" | $(sed_slsanitize)) ;\
#     nonvers=$$(echo progwithvers | sed -e "s/$${vers_sedslsanitized}$$//")
[
sed_slsanitize_command='sed_slsanitize = sed \
  -e '"'"'s!\([]^\*\$\/&[]\)!\\\1!g'"'"' \
  -e '"'"'s![-]![-]!g'"'"' \
'
]
AC_SUBST([sed_slsanitize_command])

# Similar to 'sed_slsanitize' above, but for the replacement text on the right
# side of a sed 's' (substitution) command. On the right side of a sed 's'
# substitution we only need to escape backslashes and amperdoodle chars.
[
sed_srsanitize_command='sed_srsanitize = sed \
  -e '"'"'s!\([\/&]\)!\\\1!g'"'"' \
'
]
AC_SUBST([sed_srsanitize_command])


dnl AC_CONFIG_FILES requires a list of file paths, but we do not want
dnl to have to maintain the list by hand.
dnl
dnl To obtain a list of of paths to all Makefiles we want to have
dnl generated, based on the existence of Makefile.am files, run the
dnl following shell snippet (for use as the argument to
dnl AC_CONFIG_FILES).
dnl
dnl Note that we take care to m4-quote the output strings. Filenames
dnl with square brackets in them will break this, so don't do that.
dnl
dnl   find . -type f -name 'Makefile.am' -printf "%P\n"| sed -e 's/\.am$//g;s/^/[/g;s/$/]/g' | sort
dnl
dnl Cookbook for emacs users:
dnl   1. Delete all [**/Makefile] entries from args to AC_CONFIG_FILES
dnl
dnl   2. Copy the above 'find' command line (M-w)
dnl
dnl   3. Place point (cursor) on a line by itself after
dnl      AC_CONFIG_FILES's opening paren and set mark (C-SPACE)
dnl
dnl   4. Run the 'find' command, inserting the output at point:
dnl          C-u M-! C-y RET
dnl
dnl   5. Use `indent-region' (C-M-\) to indent the quoted
dnl      strings. Region is already set up because we set the mark above.

dnl Declare our output Makefiles
AC_CONFIG_FILES(
    [Makefile]
    [src/main/bash/bin/Makefile]
)

dnl Actually output the declared files
AC_OUTPUT


# I know that shell functions are not considered portable to old ancient
# *nices, but users of such systems are unlikely users of the aws-as
# (please let me know if this assumption is incorrect!). In most cases, it may
# be that we may need to beef something up to /find/ it, but there should
# definately be a shell installed that understands functions...
#
# Prints on stdout the fully expanded path represented by the value in
# the SHELL_VAR_NAME param. Does /not/ assign the expanded value to
# the named shell var.
#
# __poor_mans_expand_file_name( SHELL_VAR_NAME )
#
# @param SHELL_VAR_NAME should be set to the /name/ of an existing
#                       shell variable whose value contains a
#                       directory path that may or may not need
#                       expansion of the configure shell variables
#                       ($prefix, and the like).
#
__poor_mans_expand_file_name () {
    _tmp_name=$1
    _tmp_last="`eval echo '${'"${_tmp_name}"'}'`"
    if test -z "${_tmp_last}"; then
        printf "__poor_mans_expand_file_name() (WARNING): var \"%s\" is empty\n" \
          "${_tmp_name}" 1>&2
        return 0;  # nothing to do
    fi
    _tmp_last_expanded="`eval echo ${_tmp_last}`"
    while test "${_tmp_last_expanded}" != "${_tmp_last}"; do

        # Set 'last' hold var to most recently expanded version...
        _tmp_last="${_tmp_last_expanded}"

        # ...and try to expand further.
        _tmp_last_expanded="`eval echo ${_tmp_last_expanded}`"
    done
    printf "%s\n" "${_tmp_last_expanded}"
    unset _tmp_last_expanded
    unset _tmp_last
    unset _tmp_name
    return 0;
}

dnl show user a summary of the configured options
printf "\n"
printf "${PACKAGE} has been configured with the following options:\n"
printf "                        Prefix: %s\n" "`__poor_mans_expand_file_name prefix`"
printf "                 User binaries: %s\n" "`__poor_mans_expand_file_name bindir`"
printf "               System binaries: %s\n" "`__poor_mans_expand_file_name sbindir`"
printf "           Configuration files: %s\n" "`__poor_mans_expand_file_name sysconfdir`"
printf "                  Manual pages: %s\n" "`__poor_mans_expand_file_name mandir`"
printf "\n"
dnl printf "                           aws: %s\n" "`__poor_mans_expand_file_name AWS_PROG`"
printf "                     aws-vault: %s\n" "`__poor_mans_expand_file_name AWS_VAULT_PROG`"
printf "                          bash: %s\n" "`__poor_mans_expand_file_name BASH_SH`"
printf "                         chmod: %s\n" "`__poor_mans_expand_file_name CHMOD_PROG`"
printf "                          date: %s\n" "`__poor_mans_expand_file_name DATE_PROG`"
printf "                         mkdir: %s\n" "`__poor_mans_expand_file_name MKDIR_PROG`"
printf "                          perl: %s\n" "`__poor_mans_expand_file_name PERL`"
printf "                       pod2man: %s\n" "`__poor_mans_expand_file_name POD2MAN`"
printf "\n"


## DEBUG: Show all variables set in the shell at the end of 'configure' run
## set | grep -v '^[ {}]' | sort
