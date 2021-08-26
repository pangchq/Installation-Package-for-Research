## MS  
**如果在天河-2安装, 先解决权限问题, 否则会被killed**
1. 下载MS相应版本并解压;  
2. ./install;  
```
指定安装目录和License目录
根据提示进行一些选择
99) Finished with license configuration, 至此主程序安装完成
```
3. 安装License;  
```
根据hostname结果在msi.lic文件相应位置进行修改
cd ~/Path/to/Accelrys/LicensePack/linux/bin
./lp_install ~/Path/to/msi.lic
Checkout succeeded, 看到这个说明License安装成功, MS安装完毕
```
**如果在天河-2安装, 需要进行一些额外修改**
vi /Path/to/BIOVIA_LicensePack/etc/lp_echovars
```
line1 #! /bin/csh -f
```
改为
```
line1 #! /WORK/app/osenv/ln1/bin/csh -f
```
vi /Path/to/MaterialsStudio19.1/etc/Gateway/root_default/dsd/commands/DSD_serverutils.pm
```

```
改为
```

```
提交脚本为
```

```