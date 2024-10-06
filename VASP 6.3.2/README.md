## VASP6.3.2, 包含vtst、VASPsol、fix lattice vector components这三个常用功能，如果想进行恒电势模拟还要加上CP-VASP插件  
1. 下载vasp相应版本并解压;  
2. 固定基矢：\
https://github.com/Chengcheng-Xiao/VASP_OPT_AXIS 下载对应版本的cell_relax.patch, copy到vasp根目录, patch -p0 < cell_relax.patch;
3. 隐式溶剂：\
https://github.com/henniggroup/VASPsol 下载patch, copy solvation.F覆盖vasp的src文件夹下的同名文件，copy对应版本的patch到src并patch -p0 < xxx.patch;  
4. VTST：\
下载vtstcode和vtstscripts, 把vtstcode解压并copy vtstcode/src/*到vasp的src文件夹下, 注意对应版本；如果是vtst6.3，还需要修改vasp/src/makefile如下
```
LIB= lib parser pyamff_fortran
```
此外要修改vasp/src/main.F和vasp/src/.objects:
*main.F
```
CALL CHAIN_FORCE(T_INFO%NIONS,DYN%POSION,TOTEN,TIFOR, &
     LATT_CUR%A,LATT_CUR%B,IO%IU6)
```
修改为
```
CALL CHAIN_FORCE(T_INFO%NIONS,DYN%POSION,TOTEN,TIFOR, &
      TSIF,LATT_CUR%A,LATT_CUR%B,IO%IU6)
```
```
IF (LCHAIN) CALL chain_init( T_INFO, IO)
```
修改为
```
CALL chain_init( T_INFO, IO)
```
*./objects, 在chain.o前添加如下内容:  
```
bfgs.o dynmat.o instanton.o lbfgs.o sd.o cg.o dimer.o bbm.o \
fire.o lanczos.o neb.o qm.o \
pyamff_fortran/*.o ml_pyamff.o \
opt.o \
```
5. 恒电势：\
https://github.com/yuanyue-liu-group/CP-VASP?tab=readme-ov-file，填表单问作者要patch，然后把patch放在vasp/src内，patch -p0 < xxx.patch;
6. module load 相应的mpi;  
7. 选择合适的makefile, 例如cp arch/makefile.include.linux_intel makefile.include; 
8. 修改makefile.include,有一些插件需要再CPP_OPTIONS添加参数；
9. make std或者make all, 可执行文件在vasp/bin文件夹下, 若修改过makefile需要make veryclean;  
10. 把vtstscripts路径加入环境变量.

