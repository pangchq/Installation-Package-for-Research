## LAMMPs
1. 下载LAMMPS相应版本并解压;  
2. 进入src目录, make ps、 make yes-xxx安装必要的package;  
3. 修改MAKE/Makefile.intel_cpu, 下面是适用于天河-2的makefile.intel_cpu;  
```
# intel_cpu_intelmpi = USER-INTEL package, Intel MPI, MKL FFT

SHELL = /bin/sh

# ---------------------------------------------------------------------
# compiler/linker settings
# specify flags and libraries needed for your compiler

CC =        mpicxx -std=c++11 
#OPTFLAGS =      -xHost -O2 -fp-model fast=2 -no-prec-div -qoverride-limits 
CCFLAGS =   -g -O3  -DLAMMPS_MEMALIGN=64 -no-offload \
                -xHost -fno-alias -ansi-alias -restrict -override-limits
SHFLAGS =   -fPIC
DEPFLAGS =  -M

LINK =            mpicxx -std=c++11
LINKFLAGS = -g -O3 -xHost
LIB =           
SIZE =            size

ARCHIVE =   ar
ARFLAGS =   -rc
SHLIBFLAGS =      -shared

# ---------------------------------------------------------------------
# LAMMPS-specific settings, all OPTIONAL
# specify settings for LAMMPS features you will use
# if you change any -D setting, do full re-compile after "make clean"

# LAMMPS ifdef settings
# see possible settings in Section 2.2 (step 4) of manual

LMP_INC =   -DLAMMPS_GZIP -DLAMMPS_JPEG

# MPI library
# see discussion in Section 2.2 (step 5) of manual
# MPI wrapper compiler/linker can provide this info
# can point to dummy MPI library in src/STUBS as in Makefile.serial
# use -D MPICH and OMPI settings in INC to avoid C++ lib conflicts
# INC = path for mpi.h, MPI compiler settings
# PATH = path for MPI library
# LIB = name of MPI library

MPI_INC =       -DMPICH_SKIP_MPICXX -DOMPI_SKIP_MPICXX=1 -I/usr/local/mpi3/include
MPI_PATH =      -L/usr/local/mpi3-dynamic/lib
MPI_LIB =       -lmpich -lmpl -lopa -lfmpich -lmpichcxx

# FFT library
# see discussion in Section 2.2 (step 6) of manaul
# can be left blank to use provided KISS FFT library
# INC = -DFFT setting, e.g. -DFFT_FFTW, FFT compiler settings
# PATH = path for FFT library
# LIB = name of FFT library
MKL_ROOT=/WORK/app/intel/composer_xe_2013_sp1.2.144/mkl
FFT_INC =       -DFFT_MKL -DFFT_SINGLE #-I${MKL_ROOT}/include/fftw
FFT_PATH = 
FFT_LIB =       -L${MKL_ROOT}/lib/intel64 -lpthread -liomp5 -lmkl_intel_ilp64 \
                -lmkl_intel_thread -lmkl_core

# JPEG and/or PNG library
# see discussion in Section 2.2 (step 7) of manual
# only needed if -DLAMMPS_JPEG or -DLAMMPS_PNG listed with LMP_INC
# INC = path(s) for jpeglib.h and/or png.h
# PATH = path(s) for JPEG library and/or PNG library
# LIB = name(s) of JPEG library and/or PNG library

JPG_INC =       
JPG_PATH =  
JPG_LIB =   -ljpeg

# ---------------------------------------------------------------------
# build rules and dependencies
# do not edit this section

include     Makefile.package.settings
include     Makefile.package

EXTRA_INC = $(LMP_INC) $(PKG_INC) $(MPI_INC) $(FFT_INC) $(JPG_INC) $(PKG_SYSINC)
EXTRA_PATH = $(PKG_PATH) $(MPI_PATH) $(FFT_PATH) $(JPG_PATH) $(PKG_SYSPATH)
EXTRA_LIB = $(PKG_LIB) $(MPI_LIB) $(FFT_LIB) $(JPG_LIB) $(PKG_SYSLIB)
EXTRA_CPP_DEPENDS = $(PKG_CPP_DEPENDS)
EXTRA_LINK_DEPENDS = $(PKG_LINK_DEPENDS)

# Path to src files

vpath %.cpp ..
vpath %.h ..

# Link target

$(EXE):     $(OBJ) $(EXTRA_LINK_DEPENDS)
      $(LINK) $(LINKFLAGS) $(EXTRA_PATH) $(OBJ) $(EXTRA_LIB) $(LIB) -o $(EXE)
      $(SIZE) $(EXE)

# Library targets

lib:  $(OBJ) $(EXTRA_LINK_DEPENDS)
      $(ARCHIVE) $(ARFLAGS) $(EXE) $(OBJ)

shlib:      $(OBJ) $(EXTRA_LINK_DEPENDS)
      $(CC) $(CCFLAGS) $(SHFLAGS) $(SHLIBFLAGS) $(EXTRA_PATH) -o $(EXE) \
        $(OBJ) $(EXTRA_LIB) $(LIB)

# Compilation rules

%.o:%.cpp $(EXTRA_CPP_DEPENDS)
      $(CC) $(CCFLAGS) $(SHFLAGS) $(EXTRA_INC) -c $<

%.d:%.cpp $(EXTRA_CPP_DEPENDS)
      $(CC) $(CCFLAGS) $(EXTRA_INC) $(DEPFLAGS) $< > $@

%.o:%.cu $(EXTRA_CPP_DEPENDS)
      $(CC) $(CCFLAGS) $(SHFLAGS) $(EXTRA_INC) -c $<

# Individual dependencies

depend : fastdep.exe $(SRC)
      @./fastdep.exe $(EXTRA_INC) -- $^ > .depend || exit 1

fastdep.exe: ../DEPEND/fastdep.c
      cc -O -o $@ $<

sinclude .depend
```
4. make intel_cpu -j 24, 在src目录下生成lmp_intel_cpu可执行文件, copy到~/bin下.