## cmake  
**需要先安装gcc, 并且cmake与gcc版本对应**
1. 下载cmake相应版本并解压;
2. bootstrap;
```
./bootstrap --prefix=/dir/cmake-x.x.x
```
一般来说会遇到 version `GLIBCXX_3.4.14' not found 的问题, 这是因为gcc升级了但是动态链接还是旧的, 解决方法是把新版gcc的lib64/libstdc++.so.6.xx软连接到usr/lib64/libstdc++.so.6
3. make -j n;
4. make install;
5. 加入环境变量, cmake --version.