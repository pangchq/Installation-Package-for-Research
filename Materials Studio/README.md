## MS  
**如果在天河-2安装, 先解决权限问题, 否则会被killed**
1. 下载MS相应版本并解压;  
2. ./install;  
```
指定安装目录和License目录
根据提示进行一些选择
...
...
...
99) Finished with license configuration, 至此主程序安装完成
```
3. 安装License;  
```
根据hostname结果在msi.lic文件相应位置进行修改
cd ~/Path/to/Accelrys/LicensePack/linux/bin
./lp_install ~/Path/to/msi.lic
...
...
...
Checkout succeeded, 看到这个说明License安装成功, MS安装完毕
```
**如果在天河-2安装, 需要进行一些额外修改**  
* vi /Path/to/BIOVIA_LicensePack/etc/lp_echovars  
```
line1 #! /bin/csh -f
```
改为
```
line1 #! /WORK/app/osenv/ln1/bin/csh -f
```
* vi /Path/to/MaterialsStudio19.1/etc/Gateway/root_default/dsd/commands/DSD_defaults.pm  
```
line 157 $DSD_defaults::dsd_MpiAppFile = "mpd.hosts";
```
改为
```
line 157 $DSD_defaults::dsd_MpiAppFile = ".mpd.hosts";
```
* 更改通信协议  
```
tar zxvf MS_lib.tar.gz
cp lib/* /Path/to/MaterialsStudio19.1/lib/
```
* 提交脚本为  
CASTEP, yhbatch -N 2 run_CASTEP BASENAME  
run_CASTEP内容为  
```
#!/WORK/app/osenv/ln1/bin/bash

source /WORK/app/toolshs/unsetfunc
export LC_ALL=C
export I_MPI_FABRICS=shm:tcp

NUM_NODES=2
PROCS_PER_NODE=24
BASENAME=$1
MS_PATH=/WORK/sysu_pangchq_1/app/MS2019/MaterialsStudio19.1
NUM_PROCS=`expr $NUM_NODES \* $PROCS_PER_NODE`

yhrun -N $NUM_NODES -n $NUM_NODES hostname > .names.log

iNodeNameTmp=$(cat .names.log)
iNodeName=($iNodeNameTmp)

if [ -e mpd.hosts ]
then
  rm mpd.hosts
fi

for iName in ${iNodeName[@]}
do
  for ((iCore=1 ; iCore <= $PROCS_PER_NODE ; iCore++))
  do
    echo $iName >> mpd.hosts
  done
done

$MS_PATH/etc/CASTEP/bin/RunCASTEP.sh -np $NUM_PROCS $BASENAME > output.log
```
Dmol3, yhbatch -N 2 run_Dmol3 BASENAME  
run_Dmol3内容为  
```
#!/WORK/app/osenv/ln1/bin/bash

source /WORK/app/toolshs/unsetfunc
export LC_ALL=C
export I_MPI_FABRICS=shm:tcp

NUM_NODES=2
PROCS_PER_NODE=24
BASENAME=$1
MS_PATH=/WORK/sysu_pangchq_1/app/MS2019/MaterialsStudio19.1
NUM_PROCS=`expr $NUM_NODES \* $PROCS_PER_NODE`

yhrun -N $NUM_NODES -n $NUM_NODES hostname > .names.log

iNodeNameTmp=$(cat .names.log)
iNodeName=($iNodeNameTmp)

if [ -e mpd.hosts ]
then
  rm mpd.hosts
fi

for iName in ${iNodeName[@]}
do
  for ((iCore=1 ; iCore <= $PROCS_PER_NODE ; iCore++))
  do
    echo $iName >> mpd.hosts
  done
done

$MS_PATH/etc/DMol3/bin/RunDMol3.sh -np $NUM_PROCS $BASENAME > output.log
```
MSpl, yhbatch -N 2 run_MSpl BASENAME  
run_MSpl内容为  
```
#!/WORK/app/osenv/ln1/bin/bash

source /WORK/app/toolshs/unsetfunc
export LC_ALL=C
export I_MPI_FABRICS=shm:tcp

NUM_NODES=2
PROCS_PER_NODE=24
BASENAME=$1
MS_PATH=/WORK/sysu_pangchq_1/app/MS2019/MaterialsStudio19.1
NUM_PROCS=`expr $NUM_NODES \* $PROCS_PER_NODE`

yhrun -N $NUM_NODES -n $NUM_NODES hostname > .names.log

iNodeNameTmp=$(cat .names.log)
iNodeName=($iNodeNameTmp)

if [ -e mpd.hosts ]
then
  rm mpd.hosts
fi

for iName in ${iNodeName[@]}
do
  for ((iCore=1 ; iCore <= $PROCS_PER_NODE ; iCore++))
  do
    echo $iName >> mpd.hosts
  done
done

mkdir -p ${BASENAME}_Files/Documents
cp ${BASENAME}.xsd ${BASENAME}_Files/Documents/
$MS_PATH/etc/Scripting/bin/RunMatScript.sh -np $NUM_PROCS $BASENAME > output.log
```