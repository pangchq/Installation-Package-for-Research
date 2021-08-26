## fftw
1. 下载fftw相应版本并解压;  
2. configure;  
```
./configure --prefix=/Path/to/fftw-x.x.x --enable-sse2 --enable-avx --enable-float --enable-shared
```
3. make -j n;  
4. make install;  
5. 加入环境变量.