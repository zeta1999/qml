include ../mk/pkg.mk

nlopt-2.4.2.tar.gz.url = http://ab-initio.mit.edu/nlopt/nlopt-2.4.2.tar.gz
nlopt-2.4.2.tar.gz.sha256 = \
    8099633de9d71cbc06cd435da993eb424bbcdbded8f803cdaa9fb8c6e09c8e89

do_extract: ../download/nlopt-2.4.2.tar.gz
	tar xzf $<
	ln -sf nlopt-2.4.2 nlopt
	mkdir -p work

# C++ code is not enabled, but configure fails without CXXCPP, so fake it.
do_configure:
	cd nlopt && ./configure \
	    CC='$(CC)' \
	    CXXCPP='$(CC) -E -x c' \
	    CFLAGS='$(FLAGS) $(CFLAGS)'

# Don't need most of the routines from general.c but they pull in other files.
do_patch:
	$(SEDI) '/nlopt_version/,$$d' nlopt/api/general.c
	cp timer.c nlopt/util/
	cp nlopt/util/nlopt-util.h work/
	patch -p0 -d work <nlopt-util.patch
	$(SEDI) 's/i\(\*:MSYS\)/\1/' nlopt/config.guess

# Explicit is better than implicit. Only build the object files we need.
# All are MIT-licensed.
OBJS = neldermead/nldrmd.o neldermead/sbplx.o \
       cobyla/cobyla.o \
       util/redblack.o util/qsort_r.o util/rescale.o util/stop.o util/timer.o \
       api/general.o
HEADERS = neldermead/neldermead.h \
          cobyla/cobyla.h \
          api/nlopt.h

define rule_make
$(1): $(patsubst %.o,%.c,$(1))
	make -C $(dir $(1)) $(notdir $(1))
endef
$(foreach obj,$(OBJS),$(eval $(call rule_make,nlopt/$(obj))))

work/libnlopt.a: $(addprefix nlopt/,$(OBJS))
	mkdir -p work
	$(AR) r $@ $^
	$(RANLIB) $@

do_build: work/libnlopt.a

do_install:
	cp work/libnlopt.a ../lib/
	cp $(addprefix nlopt/,$(HEADERS)) work/nlopt-util.h ../include/

do_clean:
	rm -rf nlopt nlopt-[0-9]* work
