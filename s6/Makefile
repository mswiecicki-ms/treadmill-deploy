# Makefile for Skarnet suite rpm.

PWD=$(shell pwd)

SRC=$(PWD)/../submodules
RPMDIR=$(PWD)/build/rpms
BLD=$(RPMDIR)/BUILD/opt/s6
S6_VERSION=$(shell git --git-dir ../submodules/s6/.git describe --tags --match "v*" 2>/dev/null | cut -b 2-)
REPO_COMMITS=$(shell git log --oneline | wc -l)

all: skalibs execline s6 s6-rc s6-dns s6-networking

clean:
	rm -rf $(BLD)
	cd $(SRC)/execline && git clean -d -f -x
	cd $(SRC)/skalibs && git clean -d -f -x
	cd $(SRC)/s6 && git clean -d -f -x
	cd $(SRC)/s6-rc && git clean -d -f -x
	cd $(SRC)/s6-dns && git clean -d -f -x
	cd $(SRC)/s6-networking && git clean -d -f -x

skalibs:
	cd $(SRC)/skalibs \
		&& ./configure --prefix=$(BLD) \
		&& make \
		&& make install

execline: skalibs
	cd $(SRC)/execline \
		&& ./configure --prefix=$(BLD) \
			--with-lib=$(BLD)/lib/skalibs \
			--with-include=$(BLD)/include \
		&& make \
		&& make install

s6: execline
	cd $(SRC)/s6 \
		&& ./configure --prefix=$(BLD) \
			--with-lib=$(BLD)/lib/skalibs \
			--with-lib=$(BLD)/lib/execline \
			--with-include=$(BLD)/include \
		&& make \
		&& make install

s6-rc: s6
	cd $(SRC)/s6-rc \
		&& ./configure --prefix=$(BLD) \
			--with-lib=$(BLD)/lib/skalibs \
			--with-lib=$(BLD)/lib/execline \
			--with-lib=$(BLD)/lib/s6 \
			--with-include=$(BLD)/include \
		&& make \
		&& make install

s6-dns: skalibs
	cd $(SRC)/s6-dns \
		&& ./configure --prefix=$(BLD) \
			--with-lib=$(BLD)/lib/skalibs \
			--with-include=$(BLD)/include \
		&& make \
		&& make install

s6-networking: s6-dns
	cd $(SRC)/s6-networking \
		&& ./configure --prefix=$(BLD) \
			--with-lib=$(BLD)/lib/skalibs \
			--with-lib=$(BLD)/lib/s6 \
			--with-lib=$(BLD)/lib/s6-dns \
			--with-include=$(BLD)/include \
		&& make \
		&& make install

filelist: _bin_list _lib_list _libexec_list _include_list

_bin_list:
	cd $(BLD) && find ./bin -type f \
		| sed "s/^\.\/bin/%{_bindir}/g" > bin.lst

_lib_list:
	cd $(BLD) && find ./lib -type f \
		| sed "s/^\.\/lib/%{_libdir}/g" > lib.lst

_libexec_list:
	cd $(BLD) && find ./libexec -type f \
		| sed "s/^\.\/libexec/%{_libexecdir}/g" > libexec.lst

_include_list:
	cd $(BLD) && find ./include -type f \
		| sed "s/^\.\/include/%{_includedir}/g" > include.lst

rpm:
	mkdir -p $(RPMDIR)/SOURCES
	mkdir -p $(RPMDIR)/SRPMS
	mkdir -p $(RPMDIR)/RPMS
	mkdir -p $(RPMDIR)/BUILD
	mkdir -p $(RPMDIR)/BUILDROOT
	rpmbuild -bb \
		-D "_topdir $(RPMDIR)" \
		-D "_version $(S6_VERSION)" \
		-D "_release $(REPO_COMMITS)" \
		-D "_builddir $(RPMDIR)/BUILD" skarnet.spec

install_rpm:
	[ ! -z $(rpm_install_dir) ] || echo must run make rpm_install_dir=...
	mkdir -p $(rpm_install_dir)
	rsync -r $(RPMDIR)/RPMS $(rpm_install_dir)
	rsync -r $(RPMDIR)/SRPMS $(rpm_install_dir)
