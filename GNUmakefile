MAKE=make
SHELL=/bin/sh
OBJDIR=obj
DRIVER_BIN=driver
PROOF=
STRFUNCS=
INSTALL=install -c
INSTALL_DIR=../bin
OPTIMIZE=-O3
CPP=clang++ -m64 -flto -E
CFLAGS=-D__USE_FIXED_PROTOTYPES__
CXX=clang++ -std=c++17 -Wno-everything
YACC=bison -d -y
RANLIB=ranlib
A=a
O=o
A_OUT=a.out
# YOU DO NOT NEED TO CONFIGURE ANYTHING IN THIS FILE.
#
# RUN THE SHELL SCRIPT ./build.MudOS to generate the Makefiles, and follow
# its instructions.
#
############################################################################
#
# **** TARGETS AND THEIR CORRECT USAGE ****
#
# COMPILATION TARGETS:
# 
# all:		compile all the files
#
# install:	make all, then move the files to the correct directories
#
# remake: 	remove the object files and generated files, and recompile
# 		     NO reconfiguration is done, etc.
#
# depend:	automatically create dependency info
#
#
# 'CLEAN' TARGETS:
#
# neat: 	remove object files and generated files (used by remake)
#
# clean:	in addition to neat, also remove .orig and .rej files,
#		cores, lint files, emacs backups, tag files, yacc debug
#		files, generated Makefiles, generated binaries, and
#		generated dependency info
#
# spotless:	make clean, then remove ALL CONFIGURATION AND CUSTOMIZATION
#		     useful for creating distributions
#
# ---- REALLY COMPLEX OPTIONS YOU PROBABLY DON'T WANT TO TOUCH -----
#
# NeXT: link with MallocDebug if you have a NeXT with NeXTOS 2.1 or later and
# you wish to search for memory leaks (see /NextDeveloper/Apps/MallocDebug).
# Note: linking with MallocDebug will cause the virtual size of the
# driver process to reach appoximately 40MB however the amount of real memory
# used will remain close to normal.
#EXTRALIBS=-lMallocDebug -lsys_s
#
# ---- DO NOT EDIT ANYTHING BELOW HERE UNLESS YOU KNOW ALOT ABOUT HOW
#      MUDOS WORKS INTERNALLY ----

OVERRIDES=$(MAKEOVERRIDES)

# **************************************************************************
# **** NOTE: If you add something here, also add it to the OBJ= rule below,
# **** or non-GNU makes will die
# **************************************************************************
SRC=grammar.tab.c lex.c main.c rc.c interpret.c simulate.c file.c object.c \
  backend.c array.c mapping.c comm.c ed.c regexp.c buffer.c crc32.c \
  malloc.c mallocwrapper.c class.c efuns_main.c efuns_port.c \
  call_out.c otable.c dumpstat.c stralloc.c hash.c \
  port.c reclaim.c parse.c simul_efun.c sprintf.c program.c \
  compiler.c avltree.c icode.c trees.c generate.c scratchpad.c \
  socket_efuns.c socket_ctrl.c qsort.c eoperators.c socket_err.c md.c \
  disassembler.c $(STRFUNCS) \
  replace_program.c master.c function.c \
  debug.c crypt.c applies_table.c add_action.c eval.c fliconv.c console.c

all: $(OBJDIR) cc.h main_build
main_build2: $(DRIVER_BIN) addr_server portbind

main_build: files
	$(MAKE) main_build2

VPATH = $(OBJDIR)

OBJ=$(addprefix $(OBJDIR)/,$(subst .c,.o,$(SRC)))

$(OBJDIR)/%.o: %.c 
	$(CXX) $(CFLAGS) $(OPTIMIZE) -o $@ -c $<

$(OBJDIR)/lex.o: lex.c preprocess.c cc.h grammar.tab.c
	$(CXX) $(CFLAGS) $(OPTIMIZE) -o $@ -c $<

$(OBJDIR)/grammar.tab.o: grammar.tab.c opcodes.h
	$(CXX) $(CFLAGS) $(OPTIMIZE) -o $@ -c $<

$(OBJDIR):
	mkdir -p $(OBJDIR)

which_makefile:
	echo MakeIsGNU

grammar.tab.c: grammar.y
	$(YACC) grammar.y
	-rm -f grammar.tab.*
	sed "s/y.tab.c/grammar.tab.c/g" y.tab.c  > grammar.tab.c
	-mv y.tab.h grammar.tab.h

packages/packages.a: packages/*.c
	$(MAKE) -C packages 'CXX=$(CXX)' 'CFLAGS=$(CFLAGS) $(OPTIMIZE)' 'OBJDIR=../$(OBJDIR)' 'RANLIB=$(RANLIB)' 'A=$(A)' 'O=$(O)'

$(DRIVER_BIN): packages/packages.a $(OBJ) dtrace_compile
	-mv -f $(DRIVER_BIN) $(DRIVER_BIN).old
	$(PROOF) $(CXX) $(CFLAGS) $(OPTIMIZE) $(OBJ) `./dtrace_compile` -o $(DRIVER_BIN) packages/packages.a $(EXTRALIBS) `cat system_libs`

dtrace_compile: dtrace_compile.c local_options
	$(CXX) dtrace_compile.c -o dtrace_compile

addr_server: files $(OBJDIR)/addr_server.o $(OBJDIR)/socket_ctrl.o $(OBJDIR)/port.o addr_server.h
	$(CXX) $(CFLAGS) $(OPTIMIZE) $(OBJDIR)/socket_ctrl.o $(OBJDIR)/addr_server.o $(OBJDIR)/port.o \
	-o addr_server `cat system_libs`

portbind: $(OBJDIR)/portbind.o
	$(CXX) $(CFLAGS) $(OPTIMIZE) $(OBJDIR)/portbind.o -o portbind `cat system_libs`


remake: neat all

customize:
	-cp ../local_options .
	-cp ../system_libs .
	-cp ../configure.h .

depend: opcodes.h grammar.tab.c cc.h efunctions.h efun_defs.c configure.h
	-rm tmpdepend
	for i in *.c; do $(CXX) -MM -DDEPEND $$i >>tmpdepend; done
	sed -e "s!^[^ ]!$(OBJDIR)/&!" <tmpdepend >Dependencies
	-rm tmpdepend

cc.h: GNUmakefile
	rm -f cc.h
	echo "/* this file automatically generated by the Makefile */" > cc.h
	echo '#define COMPILER "$(CXX)"' >> cc.h
	echo '#define OPTIMIZE "$(OPTIMIZE)"' >> cc.h
	echo '#define CFLAGS   "$(CFLAGS) $(OPTIMIZE)"' >> cc.h
	echo '#define OBJDIR   "$(OBJDIR)"' >> cc.h

# the touches here are necessary to fix the modification times; link(2) does
# 'modify' a file
files: edit_source sysmalloc.c options.h op_spec.c func_spec.c packages/Makefile.pre packages/GNUmakefile.pre configure.h grammar.y.pre
	./edit_source -options -malloc -build_func_spec '$(CPP) $(CFLAGS)' \
	              -process grammar.y.pre
	./edit_source -process packages/Makefile.pre
	./edit_source -process packages/GNUmakefile.pre
	./edit_source -build_efuns -build_applies
	touch mallocwrapper.c
	touch malloc.c
	touch files

make_func.tab.c: make_func.y cc.h
	$(YACC) $(YFLAGS) make_func.y
	-rm -f make_func.tab.c
	mv y.tab.c make_func.tab.c

configure.h: edit_source build.FluffOS
	-if test \( ! -r configure.h \) -o \( ! -r configuration \); then \
	    rm -f configuration; \
	    touch configuration; \
	fi
	if test "Machine `uname -a` Configure version 5" = "`cat configuration`"; then \
	    echo "Skipping configuration ..."; \
	else \
	    ./edit_source -configure; \
	    echo "Machine `uname -a` Configure version 5" > configuration; \
	fi

$(OBJDIR)/edit_source.o: edit_source.c preprocess.c cc.h

edit_source: $(OBJDIR)/edit_source.o $(OBJDIR)/hash.o $(OBJDIR)/make_func.tab.o
	$(CXX) $(CFLAGS) $(OBJDIR)/edit_source.o $(OBJDIR)/hash.o $(OBJDIR)/make_func.tab.o -o edit_source

# don't optimize these two
$(OBJDIR)/edit_source.o: edit_source.c
	$(CXX) $(CFLAGS) -o $@ -c $<

$(OBJDIR)/make_func.tab.o: make_func.tab.c
	$(CXX) $(CFLAGS) -o $@ -c $<

tags: $(SRC)
	ctags $(SRC)

TAGS: $(SRC)
	etags $(SRC)

install: all
	-mkdir $(INSTALL_DIR)
	$(INSTALL) $(DRIVER_BIN) $(INSTALL_DIR)
	$(INSTALL) addr_server $(INSTALL_DIR)
	$(INSTALL) portbind $(INSTALL_DIR)

Makefiles: Makefile.in NMakefile.in

Makefile.in: edit_source Makefile.in.pre Makefile.master
	./edit_source -process Makefile.in.pre

NMakefile.in: edit_source NMakefile.in.pre Makefile.master
	./edit_source -process NMakefile.in.pre

nothing:

# remove local configuration
spotless: clean
	-rm -f configure.h local_options system_libs configuration
	-rm -f options_incl.h
	-rm -f *.diffs
	-find . -name "*~" -print | xargs rm
	-rm -f 1.out 2.out

# get ready for recompile
neat:
	-(cd packages; $(MAKE) "A=$(A)" "O=$(O)" clean)
	-rm -rf $(OBJDIR) *.$(O) *.tab.c *.tab.h
	-rm -f efun_defs.c option_defs.c
	-rm -f opcodes.h efunctions.h opc.h efun_protos.h
	-rm -f malloc.c mallocwrapper.c
	-rm -f func_spec.cpp applies.h applies_table.c files
	-rm -f grammar.y comptest* a.out *.exe
	-rm -f packages/Makefile packages/GNUmakefile packages/packages

# remove everything except configuration
clean: neat
	-rm -f Makefile.MudOS GNUmakefile.MudOS
	-rm -f cc.h edit_source
	-rm -f core y.output testsuite/core testsuite/tmp/*
	-rm -f testsuite/OPCPROF.* testsuite/opc.*
	-rm -rf testsuite/binaries testsuite/single/swapfile.*
	-rm -f testsuite/OBJ_DUMP* testsuite/test_file testsuite/testfile
	-rm -f testsuite/tmp_eval_file.c testsuite/sf.o testsuite/ed_test
	-rm -f testsuite/log/log testsuite/log/debug.log testsuite/log/compile
	-find . -name "*~" -print | xargs rm
	-find . -name "*.orig" -print | xargs rm
	-find . -name "*.rej" -print | xargs rm
	-rm -f *.ln tags TAGS
	-rm -f $(DRIVER_BIN) $(DRIVER_BIN).old addr_server portbind *.exe
	-rm -f Dependencies tmpdepend
	-touch Dependencies

# remove everything
distclean: neat
	-rm -f Makefile.MudOS GNUmakefile.MudOS
	-rm -f cc.h edit_source
	-rm -f core y.output testsuite/core testsuite/tmp/*
	-rm -f testsuite/OPCPROF.* testsuite/opc.*
	-rm -rf testsuite/binaries testsuite/single/swapfile.*
	-rm -f testsuite/OBJ_DUMP* testsuite/test_file testsuite/testfile
	-rm -f testsuite/tmp_eval_file.c testsuite/sf.o testsuite/ed_test
	-rm -f testsuite/log/log testsuite/log/debug.log testsuite/log/compile
	-find . -name "*~" -print | xargs rm
	-find . -name "*.orig" -print | xargs rm
	-find . -name "*.rej" -print | xargs rm
	-rm -f *.ln tags TAGS
	-rm -f $(DRIVER_BIN) $(DRIVER_BIN).old addr_server portbind
	-rm -f *.exe
	-rm -f configure.h system_libs options_incl.h insttest configuration
	-rm -f 1.out 2.out
	-rm -f local_options
	-rm -f dtrace_compile
	-rm -f trash_me.bat trash_me.err
	-cp -f GNUmakefile.in GNUmakefile
	-rm -f Dependencies tmpdepend
	-rm -f semantic* packages/semantic*
	-touch Dependencies

include Dependencies
