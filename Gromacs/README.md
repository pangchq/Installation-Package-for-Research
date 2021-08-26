## Gromacs  
**版本越高对应的编译器也要越新!**
1. 下载Gromacs相应版本并解压;  
2. mkdir build && cd build;  
3. module  load cmake/3.5.1;  
4. cmake;  
```
cmake .. -DGMX_MPI=ON  -DCMAKE_INSTALL_PREFIX=/Path/to/gromacs-5.1.5 -DCMAKE_PREFIX_PATH=/WORK/app/fftw/3.3.4-single-avx-sse2 -DGMX_GPU=OFF -DGMX_BINARY_SUFFIX="_mpi" -DCMAKE_C_COMPILER=mpicc -DCMAKE_CXX_COMPILER=mpic++ -DGMX_SIMD=AVX_256
```
5. make -j n;
6. make install, source /WORK/sysu_pangchq_1/app/gromacs-5.1.5/bin/GMXRC.