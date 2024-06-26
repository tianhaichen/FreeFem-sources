# ======================================================================
# Laboratoire Jacques-Louis Lions
# Université Pierre et Marie Curie-Paris6, UMR 7598, Paris, F-75005 France
# http://www.ljll.math.upmc.fr/lehyaric
# ======================================================================
# This file is part of Freefem++
#
# Freefem++ is free software; you can redistribute it and/or modify
# it under the terms of the GNU Lesser General Public License as
# published by the Free Software Foundation; either version 2.1 of
# the License, or (at your option) any later version.
#
# Freefem++ is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public
# License along with Freefem++; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA
# ======================================================================
# headeralh default=0 freefem make multipleauthors start=19/03/10 upmc

include Makefile.inc

PYTHON ?=

DIR_INSTALL_REAL:=$(FF_prefix_petsc)/r
DIR_INSTALL_COMPLEX:=$(FF_prefix_petsc)/c
VERSION=$(shell grep "'petsc-" ../getall | perl -pe '($$_)=/([0-9]+([.][0-9]+)+)/')
PACKAGE=../pkg/petsc-$(VERSION).tar.gz
SRCDIR=petsc-$(VERSION)
PETSC_DIR:=$(shell pwd)/$(SRCDIR)

W_SUDO:=$(SHELL mkdir -p "$(DESTDIR)$(FF_prefix_petsc)" && test -w "$(DESTDIR)$(FF_prefix_petsc) || echo sudo)
ifeq ($(COMPILE_OPENBLAS),openblas)
PETSC_OPENBLAS := --download-f2cblaslapack
PETSC_OPENBLAS_C := --with-blaslapack-dir=$(DIR_INSTALL_REAL)
else
PETSC_OPENBLAS = --with-blaslapack-include='$(shell echo $(BLASINC) | sed "s/-I//g")' \
		--with-blaslapack-lib='$(BLAS_LAPACK_LIBS)'
PETSC_OPENBLAS_C = $(PETSC_OPENBLAS)
endif
PETSC_DOWNLOAD_C := --with-metis-dir=$(DIR_INSTALL_REAL) \
		--with-ptscotch-dir=$(DIR_INSTALL_REAL) \
		--with-superlu-dir=$(DIR_INSTALL_REAL) \
		--with-suitesparse-dir=$(DIR_INSTALL_REAL) \
		--with-parmetis-dir=$(DIR_INSTALL_REAL) \
		--with-tetgen-dir=$(DIR_INSTALL_REAL)
PETSC_DOWNLOAD := --download-metis --download-ptscotch \
		--download-hypre --download-parmetis \
		--download-superlu --download-suitesparse --download-tetgen
ifneq ($(TOOL_DYLIB_mmg),)
PETSC_DOWNLOAD += --download-mmg
PETSC_DOWNLOAD_C += --with-mmg-dir=$(DIR_INSTALL_REAL)
ifneq ($(TOOL_DYLIB_parmmg),)
PETSC_DOWNLOAD += --download-parmmg
PETSC_DOWNLOAD_C += --with-parmmg-dir=$(DIR_INSTALL_REAL)
endif
endif
ifeq ($(FFCMAKE),no)
PETSC_DOWNLOAD := $(PETSC_DOWNLOAD) --download-cmake
endif
PETSC_DOWNLOAD_C := $(PETSC_DOWNLOAD_C) --download-htool --download-htool-commit=946875d79d0036afb4dc2c0c13c165a607d830df
ifeq ($(FF_generic_petsc), yes)
FLAGS_MTUNE := -mtune=generic
else
FLAGS_MTUNE := -mtune=native
endif
unexport MAKEFLAGS
COMMON_FLAGS := --with-debugging=0 COPTFLAGS='-O3 $(FLAGS_MTUNE)' CXXOPTFLAGS='-O3 $(FLAGS_MTUNE)' FOPTFLAGS='-O3 $(FLAGS_MTUNE)' --with-ssl=0 --with-x=0 --with-fortran-bindings=0 --with-cudac=0 --with-tau-perfstubs=0 \
		--download-slepc --download-hpddm

ifneq ($(wildcard ~/.petsc_pkg),)
COMMON_FLAGS += --with-packages-download-dir=~/.petsc_pkg
endif

ifeq ($(FC),)
COMMON_FLAGS += --with-fc=0
PETSC_DOWNLOAD += --download-superlu_dist
PETSC_DOWNLOAD_C += --with-superlu_dist-dir=$(DIR_INSTALL_REAL)
endif

ifeq ($(MPICC)$(MPICXX),)
ifeq ($(WIN32DLLTARGET),)
PETSC_DOWNLOAD += --with-cc='$(CC)' --with-cxx='$(CXX)' --download-mpich
PETSC_DOWNLOAD_C += --with-mpi-dir=$(DIR_INSTALL_REAL)
ifneq ($(FC),)
PETSC_DOWNLOAD += --with-fc='$(FC)'
endif
endif
else
COMMON_FLAGS += --with-cc='$(MPICC)' --with-cxx='$(MPICXX)'
ifneq ($(MPIFC),)
COMMON_FLAGS += --with-fc='$(MPIFC)'
endif
endif

ifeq ($(WIN32DLLTARGET),)
ifneq ($(FC),)
ifneq ($(wildcard ~/.petsc_pkg),)
PETSC_DOWNLOAD += --download-slepc-configure-arguments="--download-arpack --with-packages-download-dir=~/.petsc_pkg"
else
PETSC_DOWNLOAD += --download-slepc-configure-arguments="--download-arpack"
endif
endif
endif

ifneq ($(FC),)
ifneq ($(TOOL_DYLIB_scalapack)$(TOOL_DYLIB_mumps),)
PETSC_DOWNLOAD += --download-scalapack --download-mumps
PETSC_DOWNLOAD_C += --with-scalapack-dir=$(DIR_INSTALL_REAL) --with-mumps-dir=$(DIR_INSTALL_REAL)
else
PETSC_DOWNLOAD += --download-superlu_dist
PETSC_DOWNLOAD_C += --with-superlu_dist-dir=$(DIR_INSTALL_REAL)
endif
endif

ifneq ($(WIN32DLLTARGET),)
COMMON_FLAGS += --with-shared-libraries=0 \
	CXXFLAGS='-fno-stack-protector -Wno-stringop-overflow' \
	CFLAGS='-fno-stack-protector -Wno-stringop-overflow' \
	FFLAGS='$(FCFLAGS)' \
	--with-mpiexec='/C/Program\ Files/Microsoft\ MPI/Bin/mpiexec'
ifneq ($(FC),)
ifneq ($(wildcard ~/.petsc_pkg),)
PETSC_DOWNLOAD += '--download-slepc-configure-arguments=--with-packages-build-dir=$(PETSC_DIR)/fr/externalpackages --download-arpack --with-packages-download-dir=~/.petsc_pkg'
else
PETSC_DOWNLOAD += '--download-slepc-configure-arguments=--with-packages-build-dir=$(PETSC_DIR)/fr/externalpackages --download-arpack'
endif
endif
endif

FLAGS_CONF_PETSC_REAL = $(COMMON_FLAGS) --with-scalar-type=real $(PETSC_OPENBLAS) $(PETSC_DOWNLOAD)
FLAGS_CONF_PETSC_COMPLEX = $(COMMON_FLAGS) --with-scalar-type=complex $(PETSC_OPENBLAS_C) $(PETSC_DOWNLOAD_C)

all-local:
	@echo make ..
	@echo " # To build petsc and slepc do $ make petsc-slepc"
	@echo " # warning you have no write acces in $(DESTDIR)$(FF_prefix_petsc)"
	@echo " # need sudo if '$(W_SUDO)' == 'sudo' .."
	@echo " # run under root "
	@echo " # or create / add access to $(DESTDIR)$(FF_prefix_petsc) "
	@echo " # do for examples:"
	@echo " # PETSC_DOWNLOAD = $(PETSC_DOWNLOAD)"
	@echo "	sudo mkdir $(DESTDIR)$(FF_prefix_petsc)"
	@echo "	sudo chown -R $(LOGNAME) $(DESTDIR)$(FF_prefix_petsc)"
	@echo "	make petsc-slepc "

MPI_DIR:=$(shell dirname "$(MPI_INC_DIR)")
W_MPICC:=$(strip $(shell which mpicc))
SUDO_WITH_ENV=$(if $(SUDO), $(SUDO) -E)
ifeq ($(LAPACKLIBS), $(BLASLIBS))
  BLAS_LAPACK_LIBS=$(LAPACKLIBS)
else
  BLAS_LAPACK_LIBS=$(LAPACKLIBS) $(BLASLIBS)
endif

petsc-slepc: Makefile.inc dir-install WHERE-all
recompile: recompile-real recompile-complex
recompile-real:
	-rm */tag-make-real
	-rm */tag-install-real
	make WHERE
recompile-complex:
	-rm */tag-make-complex
	-rm */tag-install-complex
	@if [ -n "$(MPI_INCLUDE)" ]; then $(MAKE) -f Makefile.complex WHERE-complex ;\
	else echo " -- No PETSc, no MPI"; fi

dir-install: Makefile.inc
	-$(SUDO) mkdir -p $(DESTDIR)$(DIR_INSTALL_REAL)
	-$(SUDO) chown ${USER} $(DESTDIR)$(DIR_INSTALL_REAL)
	-$(SUDO) mkdir -p $(DESTDIR)$(DIR_INSTALL_COMPLEX)
	-$(SUDO) chown ${USER} $(DESTDIR)$(DIR_INSTALL_COMPLEX)
	@if test -w 	"$(DESTDIR)$(DIR_INSTALL_REAL)" -a -w 	"$(DESTDIR)$(DIR_INSTALL_COMPLEX)" ; then echo "no need of sudo"; echo > do-sudo; \
	else \
	 echo no write access in $(DESTDIR)$(DIR_INSTALL_REAL) and $(DESTDIR)$(DIR_INSTALL_COMPLEX); \
	 echo " do: make petsc-slepc SUDO=sudo "; \
	 echo " or add write access to 2 directory "; \
	 echo sudo >do-sudo; \
	fi
	test -e "$(DESTDIR)$(DIR_INSTALL_REAL)" -a -e "$(DESTDIR)$(DIR_INSTALL_COMPLEX)"
PETSc-real:
	@if [ -n "$(MPI_INCLUDE)" ]; then $(MAKE) WHERE ;\
	else echo " -- No PETSc, no MPI"; fi

PETSc-complex:
	@if [ -n "$(MPI_INCLUDE)" ]; then $(MAKE) -f Makefile.complex WHERE-complex ;\
	else echo " -- No PETSc, no MPI, no PETSc real"; fi

WHERE:../lib/WHERE.PETSc ../lib/WHERE.SLEPc

../lib/WHERE.PETSc:$(SRCDIR)/tag-install-real
	-mkdir ../lib
	echo 'petsc LD -Wl,-rpath,"$(DIR_INSTALL_REAL)/lib" -L"$(DIR_INSTALL_REAL)/lib" -lpetsc' > $@
	echo 'petsc INCLUDE -I"$(DIR_INSTALL_REAL)/include"' >> $@

$(SRCDIR)/tag-make-real:$(SRCDIR)/tag-conf-real
	cd $(SRCDIR) && $(MAKE) PETSC_DIR=$(PETSC_DIR) PETSC_ARCH=fr all
	touch $@


$(SRCDIR)/tag-install-real :$(SRCDIR)/tag-make-real
	cd $(SRCDIR) && $(MAKE) PETSC_DIR=$(PETSC_DIR) PETSC_ARCH=fr install-lib
	cd $(SRCDIR) && export HWLOC_HIDE_ERRORS=2 && $(MAKE) PETSC_DIR=$(PETSC_DIR) PETSC_ARCH=fr check
	test -e $(DIR_INSTALL_REAL)/include/petsc.h
	test -e $(DIR_INSTALL_REAL)/lib/petsc/conf/petscvariables
	touch $@

Makefile.inc:../../config.status Makefile Makefile-PETSc.inc
	../../config.status --file="Makefile.inc:Makefile-PETSc.inc"

$(SRCDIR)/tag-conf-real:$(SRCDIR)/tag-tar
	cd $(SRCDIR) && $(PYTHON) ./configure --prefix=$(DIR_INSTALL_REAL) \
		$(FLAGS_CONF_PETSC_REAL) PETSC_ARCH=fr
	test -f $(SRCDIR)/fr/lib/petsc/conf/petscvariables
	mkdir -p $(SRCDIR)/fc/externalpackages || true
	cp -r $(SRCDIR)/fr/externalpackages/git.slepc $(SRCDIR)/fr/externalpackages/git.hpddm $(SRCDIR)/fc/externalpackages || true
	touch $@
Make-petsc-download.mk:$(SRCDIR)/tag-install-real
	test -e $(DIR_INSTALL_REAL)/lib/petsc/conf/petscvariables
	egrep 'PETSC_LIB_BASIC|SCALAPACK_|METIS_|MUMPS_|HPDDM_|HTOOL_|TETGEN_|SUPERLU_|MMG_|PTSCOTCH_|SUITESPARSE_' $(DIR_INSTALL_REAL)/lib/petsc/conf/petscvariables | sed 's/-I/ /g'|sort >$@
ifdef 	COMPLEX_CASE
#  version COMPLEX.....
$(SRCDIR)/tag-conf-complex:$(SRCDIR)/tag-tar Make-petsc-download.mk
	cd $(SRCDIR) && $(PYTHON) ./configure --prefix=$(DIR_INSTALL_COMPLEX) \
		$(FLAGS_CONF_PETSC_COMPLEX) PETSC_ARCH=fc
	test -f $(SRCDIR)/fc/lib/petsc/conf/petscvariables
	touch $@

$(SRCDIR)/tag-make-complex:$(SRCDIR)/tag-conf-complex Make-petsc-download.mk
	cd $(SRCDIR) && $(MAKE) PETSC_DIR=$(PETSC_DIR) PETSC_ARCH=fc all
	touch $@

$(SRCDIR)/tag-install-complex :$(SRCDIR)/tag-make-complex
	cd $(SRCDIR) && $(MAKE) PETSC_DIR=$(PETSC_DIR) PETSC_ARCH=fc install-lib
	test -e $(DIR_INSTALL_COMPLEX)/include/petsc.h
	touch $@
WHERE-complex:../lib/WHERE.PETSc-complex ../lib/WHERE.SLEPc-complex

../lib/WHERE.PETSc-complex:$(SRCDIR)/tag-install-complex
	test -e $(DIR_INSTALL_COMPLEX)/include/petsc.h
	-mkdir ../lib
	echo 'petsccomplex LD -Wl,-rpath,"$(DIR_INSTALL_COMPLEX)/lib" -L"$(DIR_INSTALL_COMPLEX)/lib" -lpetsc' > $@
	echo 'petsccomplex INCLUDE -I"$(DIR_INSTALL_COMPLEX)/include"' >> $@
	echo done $@
../lib/WHERE.SLEPc-complex:
	test -e $(DIR_INSTALL_COMPLEX)/include/slepc.h
	-mkdir ../lib
	echo 'slepccomplex LD -Wl,-rpath,"$(DIR_INSTALL_COMPLEX)/lib" -L"$(DIR_INSTALL_COMPLEX)/lib" -lslepc' > $@
	echo 'slepccomplex INCLUDE -I"$(DIR_INSTALL_COMPLEX)/include"' >> $@
	echo done $@

WHERE-all:Makefile WHERE WHERE-complex
	echo >$@
	test -z '$(SCALAPACK_LIB)' || echo scalapack LD $(SCALAPACK_LIB) >>$@
	test -z '$(SCALAPACK_INCLUDE)' || echo scalapack INCLUDE -I$(SCALAPACK_INCLUDE) >>$@
	test -z '$(METIS_LIB)' || echo metis LD $(METIS_LIB) >>$@
	test -z '$(METIS_INCLUDE)' || echo metis INCLUDE -I$(METIS_INCLUDE) >>$@
	test -z '$(MUMPS_LIB)' || echo mumps LD $(MUMPS_LIB) >>$@
	test -z '$(MUMPS_INCLUDE)' || echo mumps INCLUDE -I$(MUMPS_INCLUDE) >>$@
	test -z '$(SUPERLU_LIB)' || echo superlu LD $(SUPERLU_LIB) >>$@
	test -z '$(SUPERLU_INCLUDE)' || echo superlu INCLUDE -I$(SUPERLU_INCLUDE) >>$@
	test -z '$(MMG_LIB)' || echo mmg LD $(MMG_LIB) >>$@
	test -z '$(MMG_INCLUDE)' || echo mmg INCLUDE -I$(MMG_INCLUDE) >>$@
	test -z '$(PARMMG_LIB)' || echo parmmg LD $(PARMMG_LIB) >>$@
	test -z '$(PARMMG_INCLUDE)' || echo parmmg INCLUDE -I$(PARMMG_INCLUDE) >>$@
	test -z '$(PTSCOTCH_LIB)' || echo ptscotch LD $(PTSCOTCH_LIB) >>$@
	test -z '$(PTSCOTCH_INCLUDE)' || echo ptscotch INCLUDE -I$(PTSCOTCH_INCLUDE) >>$@
	test -z '$(TETGEN_LIB)' || echo tetgen LD $(TETGEN_LIB) >>$@
	test -z '$(TETGEN_INCLUDE)' || echo tetgen INCLUDE -I$(TETGEN_INCLUDE) >>$@
	test -z '$(PARMETIS_LIB)' || echo parmetis LD $(PARMETIS_LIB) >>$@
	test -z '$(PARMETIS_INCLUDE)' || echo parmetis INCLUDE -I$(PARMETIS_INCLUDE) >>$@
	test -z '$(HPDDM_LIB)' || echo hpddm LD $(HPDDM_LIB) >>$@
	test -z '$(HPDDM_INCLUDE)' || echo hpddm INCLUDE -I$(HPDDM_INCLUDE) >>$@
	test -z '$(HTOOL_LIB)' || echo htool LD $(HTOOL_LIB) >>$@
	test -z '$(HTOOL_INCLUDE)' || echo htool INCLUDE -I$(HTOOL_INCLUDE) >>$@
	echo 'petsc LD -Wl,-rpath,"$(DIR_INSTALL_REAL)/lib" -L"$(DIR_INSTALL_REAL)/lib" -lpetsc' >>$@
	echo 'petsc INCLUDE -I"$(DIR_INSTALL_REAL)/include"' >>$@
	echo 'petsccomplex LD -Wl,-rpath,"$(DIR_INSTALL_COMPLEX)/lib" -L"$(DIR_INSTALL_COMPLEX)/lib" -lpetsc' >>$@
	echo 'petsccomplex INCLUDE -I"$(DIR_INSTALL_COMPLEX)/include"' >> $@
	echo 'slepc LD -Wl,-rpath,"$(DIR_INSTALL_REAL)/lib" -L"$(DIR_INSTALL_REAL)/lib" -lslepc' >>$@
	echo 'slepc INCLUDE -I"$(DIR_INSTALL_REAL)/include"' >>$@
	echo 'slepccomplex LD -Wl,-rpath,"$(DIR_INSTALL_COMPLEX)/lib" -L"$(DIR_INSTALL_COMPLEX)/lib" -lslepc' >>$@
	echo 'slepccomplex INCLUDE -I"$(DIR_INSTALL_COMPLEX)/include"' >>$@

#
else
#
WHERE-complex:
	@if [ -n "$(MPI_INCLUDE)" ]; then $(MAKE) -f Makefile.complex WHERE-complex ;\
	else echo " -- No PETSc, no MPI"; fi

WHERE-all:Makefile Make-petsc-download.mk
	 $(MAKE) -f Makefile.complex $@
endif
$(SRCDIR)/tag-tar:$(PACKAGE)
	-mkdir $(SRCDIR) 2>/dev/null
	-tar xzf $(PACKAGE) -C $(SRCDIR) --strip-components 1
	cd $(SRCDIR) && patch -p1 < ../petsc-cmake-seq.patch && cd -
	touch $@
$(PACKAGE):
	../getall -o PETSc -a

../lib/WHERE.SLEPc:
	-mkdir ../lib
	test -e $(DIR_INSTALL_REAL)/include/slepc.h
	echo 'slepc LD -Wl,-rpath,"$(DIR_INSTALL_REAL)/lib" -L"$(DIR_INSTALL_REAL)/lib" -lslepc' > $@
	echo 'slepc INCLUDE -I"$(DIR_INSTALL_REAL)/include"' >> $@

clean:clean-local
clean-local:
	-cd $(SRCDIR) && $(MAKE) clean -C $(SRCDIR)
	-rm Makefile.inc FAIRE* ../lib/WHERE.PETSc* ../lib/WHERE.SLEPc*
	-rm -rf ../include/*PETSc*
	-rm -rf ../lib/lib*PETSc*
	-rm -rf $(SRCDIR)
	-rm -rf $(FF_prefix_petsc)
	-rm WHERE-all config.log *.done
	-if test -d $(FF_prefix_petsc) ; then echo " try of remove of $(FF_prefix_petsc) under sudo .."; sudo rm -rf $(FF_prefix_petsc) ; fi
	-rm do-sudo
echo:
	@echo SUDO: $(SUDO)
	@echo MTUNE: $(FLAGS_MTUNE)
	@echo "sudo using user env: $(SUDO_WITH_ENV)"
	@echo MPI_DIR: $(MPI_DIR)
	@echo " dir install real :" $(DIR_INSTALL_REAL)
	@echo " dir install complex :" $(DIR_INSTALL_COMPLEX)
	@echo " do-sudo auto ???:" $(SHELL cat do-sudo)
	@echo " BLAS_LAPACK_LIBS: $(BLAS_LAPACK_LIBS)"
	@echo " # need sudo if '$(W_SUDO)' == 'sudo' .."

install-destdir:
	cd $(SRCDIR) && $(MAKE) PETSC_DIR=$(PETSC_DIR) PETSC_ARCH=fr install-lib DESTDIR=$(DESTDIR)
	cd $(SRCDIR) && $(MAKE) PETSC_DIR=$(PETSC_DIR) PETSC_ARCH=fc install-lib DESTDIR=$(DESTDIR)

.NOTPARALLEL:
# Local Variables:
# mode:makefile
# ispell-local-dictionary:"british"
# coding:utf-8
# End:
