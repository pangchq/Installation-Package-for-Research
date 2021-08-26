## VASP, 包含vtst、VASPsol、fix lattice vector components这三个常用功能
1. 下载vasp相应版本并解压;  
2. 下载vtstcode和vtstscripts, 把vtstcode解压并copy到vasp的src文件夹下, 注意对应版本  
修改src/main.F源码:  
CALL CHAIN_FORCE(T_INFO%NIONS,DYN%POSION,TOTEN,TIFOR, &
     LATT_CUR%A,LATT_CUR%B,IO%IU6)  
改为  
CALL CHAIN_FORCE(T_INFO%NIONS,DYN%POSION,TOTEN,TIFOR, &
      TSIF,LATT_CUR%A,LATT_CUR%B,IO%IU6)  
修改src/.objects源码, 在chain.o前（大概第72行）添加如下内容:  
bfgs.o dynmat.o instanton.o lbfgs.o sd.o cg.o dimer.o bbm.o \  
fire.o lanczos.o neb.o qm.o opt.o \
3. https://github.com/henniggroup/VASPsol 下载patch, copy solvation.F覆盖vasp的src文件夹下的同名文件;  
4. https://github.com/Chengcheng-Xiao/VASP_OPT_AXIS 下载cell_relax.patch, copy到vasp根目录, patch -p0 < cell_relax.patch;  
5. module load intel-compilers/mkl-14, echo $MKLROOT;  
6. 选择合适的makefile, cp arch/makefile.include.linux_intel makefile.include;  
7. 修改makefile.include, 下面是适用于天河-2的makefile.include;  
```
# Precompiler options
CPP_OPTIONS= -DHOST=\"LinuxIFC\"\
             -DMPI -DMPI_BLOCK=8000 \
             -Duse_collective \
             -DscaLAPACK \
             -DCACHE_SIZE=4000 \
             -Davoidalloc \
             -Duse_bse_te \
             -Dsol_compat
 
CPP        = fpp -f_com=no -free -w0  $*$(FUFFIX) $*$(SUFFIX) $(CPP_OPTIONS)
 
FC         = mpif90
FCL        = mpif90 -mkl=sequential -lstdc++
 
#FREE       = -free -names lowercase
FREE        = -free -heap-arrays 
 
FFLAGS     = -assume byterecl -w
OFLAG      = -O2
OFLAG_IN   = $(OFLAG)
DEBUG      = -O0
 
MKLROOT    =/WORK/app/intel/composer_xe_2013_sp1.2.144/mkl
MKL_PATH   = $(MKLROOT)/lib/intel64
BLAS       =-L$(MKL_PATH) -lmkl_intel_lp64 -lmkl_sequential -lmkl_core -lpthread
LAPACK     =-L$(MKL_PATH) -lmkl_intel_lp64 -lmkl_sequential -lmkl_core -lpthread
#BLACS      = -lmkl_blacs_intelmpi_lp64
BLACS      =-L$(MKL_PATH) -lmkl_blacs_intelmpi_lp64 -lmkl_blacs_openmpi_lp64
SCALAPACK  = $(MKL_PATH)/libmkl_scalapack_lp64.a $(MKL_PATH)/libmkl_scalapack_ilp64.a $(BLACS)
OBJECTS    = fftmpiw.o fftmpi_map.o fftw3d.o fft3dlib.o /WORK/app/fftw/3.3.4/lib/libfftw3_mpi.a
 
INCS       =-I/WORK/app/fftw/3.3.4-double/include
 
LLIBS      = $(SCALAPACK) $(LAPACK) $(BLAS)
 
OBJECTS_O1 += fftw3d.o fftmpi.o fftmpiw.o
OBJECTS_O2 += fft3dlib.o
 
# For what used to be vasp.5.lib
CPP_LIB    = $(CPP)
FC_LIB     = $(FC)
CC_LIB     = icc
CFLAGS_LIB = -O
FFLAGS_LIB = -O1
OBJECTS_LIB= linpack_double.o getshmem.o
 
# For the parser library
CXX_PARS   = icpc
 
LIBS       += parser
LLIBS      += -Lparser -lparser -lstdc++
 
# Normally no need to change this
SRCDIR     = ../../src
BINDIR     = ../../bin
 
#================================================
# GPU Stuff
 
CPP_GPU    = -DCUDA_GPU -DRPROMU_CPROJ_OVERLAP -DUSE_PINNED_MEMORY -DCUFFT_MIN=28 -UscaLAPACK
 
OBJECTS_GPU = fftmpiw.o fftmpi_map.o fft3dlib.o fftw3d_gpu.o fftmpiw_gpu.o
 
CC         = icc
CXX        = icpc
CFLAGS     = -fPIC -DADD_ -Wall -openmp -DMAGMA_WITH_MKL -DMAGMA_SETAFFINITY -DGPUSHMEM=300 -DHAVE_CUBLAS
 
CUDA_ROOT  ?= /usr/local/cuda/
NVCC       := $(CUDA_ROOT)/bin/nvcc -ccbin=icc
CUDA_LIB   := -L$(CUDA_ROOT)/lib64 -lnvToolsExt -lcudart -lcuda -lcufft -lcublas
 
GENCODE_ARCH    := -gencode=arch=compute_30,code=\"sm_30,compute_30\" \
                   -gencode=arch=compute_35,code=\"sm_35,compute_35\" \
                   -gencode=arch=compute_60,code=\"sm_60,compute_60\"
 
MPI_INC    = $(I_MPI_ROOT)/include64/
```
8. make std或者make all, 可执行文件在vasp/bin文件夹下, 若修改过makefile需要make veryclean.