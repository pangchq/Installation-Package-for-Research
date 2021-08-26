## gcc
**gcc版本并不是越高越好!**
1. 下载gcc相应版本并解压;
2. http://gcc.gnu.org/pub/gcc/infrastructure/ 下载对应gcc版本的gmp、mpfr、mpc、isl, copy到gcc目录下;
3. ./contrib/download_prerequisites;
4. configure;
```
./configure --prefix=/Path/to/gcc-x.x.x --enable-bootstrap --enable-languages=c,c++ --enable-threads=posix --enable-checking=release --disable-multilib --with-system-zlib
```
5. make -j n, 编译过程很耗时, 注意需先安装gcc-c++;
6. make install;
7. 加入环境变量, gcc --version.
