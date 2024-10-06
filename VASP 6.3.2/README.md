## VASP6.3.2, 包含vtst、VASPsol、fix lattice vector components这三个常用功能，如果想进行恒电势模拟还要加上CP-VASP插件  
1. 下载vasp相应版本并解压;  
2. 固定基矢插件：\
https://github.com/Chengcheng-Xiao/VASP_OPT_AXIS 下载对应版本的cell_relax.patch, copy到vasp根目录, patch -p0 < cell_relax.patch;
3. VTST插件：\
https://theory.cm.utexas.edu/vtsttools/installation.html, 下载vtstcode和vtstscripts, 把vtstcode解压并copy vtstcode/src/*到vasp的src文件夹下, 注意对应版本；如果是vtstcode6.3，还需要修改vasp/src/makefile如下
```
LIB= lib parser pyamff_fortran
```
此外要修改vasp/src/main.F和vasp/src/.objects:\
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
vtstcode6.3有个小问题(参考http://bbs.keinsci.com/thread-46112-1-1.html), 少了一个ENDIF，需要补上，否则编译出错；在vasp/src/chain.F大概202行把
```
 199         IF (newcar_exists) THEN
 200           CALL RD_POSCAR_HEAD(LATT_CUR, T_I, NIOND, NIONPD, NTYPD, NTYPPD, IO%IU0, IO%IU6)
 201           CALL RD_POSCAR(LATT_CUR, T_I, DYN, NIOND, NIONPD, NTYPD, NTYPPD, IO%IU0, IO%IU6)
 202           posion = DYN%POSION
```
修改为
```
 199         IF (newcar_exists) THEN
 200           CALL RD_POSCAR_HEAD(LATT_CUR, T_I, NIOND, NIONPD, NTYPD, NTYPPD, IO%IU0, IO%IU6)
 201           CALL RD_POSCAR(LATT_CUR, T_I, DYN, NIOND, NIONPD, NTYPD, NTYPPD, IO%IU0, IO%IU6)
 202           posion = DYN%POSION
 203         ENDIF
```
4. 隐式溶剂插件：\
vaspsol https://github.com/henniggroup/VASPsol 下载patch, copy solvation.F覆盖vasp的src文件夹下的同名文件，copy对应版本的patch到src并patch -p0 < xxx.patch;\
vaspsol++ https://github.com/VASPsol/VASPsol 问作者要patch, 选择合适的patch, 然后在vasp根目录patch -p1 < xxx.patch； 
5. 恒电势插件：\
https://github.com/yuanyue-liu-group/CP-VASP?tab=readme-ov-file, 填表单问作者要patch, 然后把patch放在vasp/src内, patch -p0 < xxx.patch;
6. module load 相应的mpi;  
7. 选择合适的makefile, 例如cp arch/makefile.include.linux_intel makefile.include; 
8. 修改makefile.include, 有一些插件需要再CPP_OPTIONS添加参数；
9. make std或者make all, 可执行文件在vasp/bin文件夹下, 若修改过makefile需要make veryclean;  
10. 把vtstscripts路径加入环境变量.

