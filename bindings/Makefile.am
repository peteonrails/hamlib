
# Hamlib bindings using SWIG
# more information on swig at http://www.swig.org
#

AM_CPPFLAGS = @AM_CPPFLAGS@ -I$(top_srcdir)/bindings @TCL_INCLUDE_SPEC@ @PYTHON_CPPFLAGS@

AM_CFLAGS = -fno-strict-aliasing

SWGFILES = hamlib.swg ignore.swg rig.swg rotator.swg

SWGDEP=$(top_srcdir)/include/hamlib/rig.h $(top_srcdir)/include/hamlib/riglist.h \
	$(top_srcdir)/include/hamlib/rotator.h $(top_srcdir)/include/hamlib/rotlist.h \
	$(SWGFILES)

EXTRA_DIST = $(SWGFILES) \
	Makefile.PL perltest.pl tcltest.tcl pytest.py

noinst_SCRIPTS = perltest.pl tcltest.tcl pytest.py hamlibvb.bas

BUILT_SOURCES=
MOSTLYCLEANFILES=


if ENABLE_PERL
##########################################
# Perl binding

BUILT_SOURCES += hamlibperl_wrap.c
MOSTLYCLEANFILES += hamlibperl_wrap.c Hamlib.pm Hamlib.bs Hamlib-pl.mk.old

hamlibperl_wrap.c: hamlib.swg $(SWIGDEP)
	$(SWIG) -perl5 -shadow @AM_CPPFLAGS@ -I$(top_srcdir)/bindings -o $@ \
			`test -f hamlib.swg || echo '$(srcdir)/'`hamlib.swg

Hamlib-pl.mk: $(srcdir)/Makefile.PL
	test -f Makefile.PL || $(LN_S) $(srcdir)/Makefile.PL Makefile.PL
	perl `test -f Makefile.PL || echo '$(srcdir)/'`Makefile.PL \
			MAKEFILE="Hamlib-pl.mk" \
			PREFIX="$(prefix)" \
			INC="$(AM_CPPFLAGS)"  \
			CC="$(CC)"  \
			OBJECT="hamlibperl_wrap.o" \
			VERSION="$(PACKAGE_VERSION)" \
			LIBS="-L$(top_builddir)/src/.libs -lhamlib"
	sed -e 's/^LD_RUN_PATH.*/LD_RUN_PATH =/' Hamlib-pl.mk > Hamlib-pl-norpath.mk
	mv Hamlib-pl-norpath.mk Hamlib-pl.mk

all-perl: Hamlib-pl.mk hamlibperl_wrap.c
	$(MAKE) $(AM_MAKEFLAGS) -f Hamlib-pl.mk all

check-perl: all-perl
	$(MAKE) $(AM_MAKEFLAGS) -f Hamlib-pl.mk test

clean-perl: Hamlib-pl.mk
	$(MAKE) $(AM_MAKEFLAGS) -f Hamlib-pl.mk clean

## 'distclean' target is a NOOP in Hamlib-pl.mk
distclean-perl: ## Hamlib-pl.mk
##	$(MAKE) $(AM_MAKEFLAGS) -f Hamlib-pl.mk distclean

distcheck-perl: Hamlib-pl.mk
	$(MAKE) $(AM_MAKEFLAGS) -f Hamlib-pl.mk distcheck

install-perl: Hamlib-pl.mk
	$(MAKE) $(AM_MAKEFLAGS) -f Hamlib-pl.mk install_site

## ExtUtils::MakeMaker says uninstall is deprecated and won't remove the files
## anyway, so we'll have to figure out another way to remove the installed files.
uninstall-perl: ## Hamlib-pl.mk
##	$(MAKE) $(AM_MAKEFLAGS) -f Hamlib-pl.mk uninstall

endif
# Perl


if ENABLE_PYTHON
##########################################
# Python binding

pyexec_ltlib = _Hamlib.la

MOSTLYCLEANFILES+= hamlibpy_wrap.c Hamlib.py *.pyc
BUILT_SOURCES += hamlibpy_wrap.c Hamlib.py

nodist_pyexec_PYTHON = Hamlib.py
nodist__Hamlib_la_SOURCES = hamlibpy_wrap.c

_Hamlib_la_LDFLAGS = -no-undefined -module -avoid-version
_Hamlib_la_LIBADD = $(top_builddir)/src/libhamlib.la
_Hamlib_ladir = $(pyexecdir)
_Hamlib_la_LTLIBRARIES = $(pyexec_ltlib)

all-py: $(pyexec_ltlib)

check-py: all-py
	$(srcdir)/pytest.py || echo "Python test failed" 1>&2

Hamlib.py: hamlibpy_wrap.c

hamlibpy_wrap.c: hamlib.swg $(SWGDEP)
	$(SWIG) -python @AM_CPPFLAGS@ -I$(top_srcdir)/bindings -o $@ \
		`test -f hamlib.swg || echo '$(srcdir)/'`hamlib.swg

install-py:
clean-py:
distclean-py:
uninstall-py:

endif
# Python


if ENABLE_TCL
##########################################
# Tcl binding

BUILT_SOURCES+= hamlibtcl_wrap.c
MOSTLYCLEANFILES+= hamlibtcl_wrap.c pkgIndex.tcl

tcl_ltlib = hamlibtcl.la

tcldir = $(libdir)/tcl/Hamlib

PKG_VER=@ABI_VERSION@.@ABI_REVISION@
DLL=hamlibtcl-$(PKG_VER)@TCL_SHLIB_SUFFIX@

nodist_hamlibtcl_la_SOURCES = hamlibtcl_wrap.c
hamlibtcl_la_LDFLAGS = -no-undefined -module -release $(PKG_VER) -avoid-version @TCL_LIB_SPEC@
hamlibtcl_la_LIBADD = $(top_builddir)/src/libhamlib.la

hamlibtcl_ladir = $(tcldir)
hamlibtcl_la_DATA = pkgIndex.tcl

# Install hamlibtcl.la into the tcltk/Hamlib directory set by $hamlibtcl_ladir
hamlibtcl_la_LTLIBRARIES = $(tcl_ltlib)

# Having the first occurance of 'hamlib' capitalized seemed to cause confusion
# for the TCL interpreter loading the Hamlib module.
pkgIndex.tcl: Makefile
	echo 'package ifneeded hamlib $(PKG_VER) [list load [file join $$dir $(DLL)] Hamlib]' > pkgIndex.tcl

hamlibtcl_wrap.c: hamlib.swg $(SWGDEP)
	$(SWIG) -tcl -pkgversion $(PKG_VER) @AM_CPPFLAGS@ -I$(top_srcdir)/bindings -o $@ \
		`test -f hamlib.swg || echo '$(srcdir)/'`hamlib.swg

all-tcl: pkgIndex.tcl $(tcl_ltlib)

check-tcl: all-tcl
	TCLLIBPATH=$(builddir)/.libs $(srcdir)/tcltest.tcl || echo "Tcl test failed" 1>&2

install-tcl:
clean-tcl:
distclean-tcl: clean-tcl
uninstall-tcl:

endif
# TCL


all-local: @BINDING_ALL@

check-local: @BINDING_CHECK@
clean-local: @BINDING_CLEAN@
distclean-local: @BINDING_DISTCLEAN@
distcheck-local: @BINDING_DISTCHECK@
install-exec-local: @BINDING_INSTALL_EXEC@
uninstall-local: @BINDING_UNINSTALL@
