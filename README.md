# 在 ESXi 6.7 上安装黑群晖 DSM 7

目前 Redpill 支持的黑群晖型号：

|型号|CPU|微架构|盘位|
| -------- | -------- | -------- | -------- |
|DS3615xs|Intel Core i3-4130|Haswell (2013-9-1)|36|
|DS3617xs|Intel Xeon D-1527 (2015-11-1)|Broadwell|36|
|DS916+|Intel Pentium N3710 (2015-3)|Braswell|9|
|DS918+|Intel Celeron J3455 (2016-8-30)|Apollo Lake|9|
|DS920+|Intel Celeron J4125 (2019-11)|Gemini Lake Refresh|9|
|DS3622xs+|Intel Xeon D-1531 (2015-11-1)|Broadwell|36|
|FS6400|Intel Xeon Silver 4110 (2017-7-11)|Skylake|48/72|
|DVA3219|Intel Atom C3538 (2017-8-15)|Denverton|14|
|DVA3221|Intel Atom C3538 (2017-8-15)|Denverton|14|
|DS1621+|AMD Ryzen V1500B (2018-12)|Zen|16|

本文以在 ESXi 6.7 上安装 DS3622xs+ 为例。共有 7 块物理硬盘，其中 3 块连接在主板 SATA 接口，作 RDM 供黑群晖使用，4 块连接在 PCI-E 转 SATA 扩展卡，直通给黑群晖。另有一块 PCI-E 网卡直通给黑群晖。

方法参照 [tmyers07](<https://github.com/tmyers07>) 的[教程](<https://www.tsunati.com/blog/xpenology-7-0-1-on-esxi-7-x>)，稍作改动。

## 下载
- [tinycore-redpill 虚拟硬盘文件](<https://drive.google.com/drive/folders/1nRoggLEVLRbKagIaP3aE28m73agiEGpQ>)（tmyers07 提供，目前版本是 0.4.6）
- [DSM v7.0.1-42218](<https://global.download.synology.com/download/DSM/release/7.0.1/42218/DSM_DS3622xs%2B_42218.pat>)（来自 [群晖官网](<https://archive.synology.com/download/Os/DSM>)）
- [Offline bundle for ESXi 6.x - esxui-offline-bundle-6.x-10692217.zip](<https://flings.vmware.com/esxi-embedded-host-client>)（可能会用到）

## 新建虚拟机
1. 在 ESXi 新建虚拟机，此处假定虚拟机名为『XPEnology』。
2. 虚拟机操作系统类型选『Linux』，版本选『其他 4.x 或更高版本的 Linux (64位)』。
3. 内存勾选『预选所有客户机内存』。
4. 删除原有硬盘、SCSI 控制器、USB 控制器、光驱。
5. 添加一个 SATA 控制器，此时应共有两个，编号分别是『SATA 控制器 0』和『SATA 控制器 1』。
6. 虚拟机选项 - 引导选项 - 固件，改为『BIOS』。
7. ESXi - 存储 - datastore1 - 数据存储浏览器，在『XPEnology』目录内上传已下载的 tinycore-redpill.v0.4.6-flat.vmdk 和 tinycore-redpill.v0.4.6.vmdk。

## 第一次修改虚拟机配置
1. 虚拟机添加一块现有硬盘，选 tinycore-redpill.v0.4.6.vmdk，控制器选为『SATA 控制器 0:0』。
2. 虚拟机添加一块标准硬盘，大小可设为 50GB，厚置备延迟置零，控制器选为『SATA 控制器 1:0』。

## 虚拟机第一次开机
1. 待进入 Tinycore 的图形界面，在其桌面鼠标右击，弹出菜单中用键盘方向键依次选 Applications 和 Terminal，打开终端。
2. 终端中，用 `ifconfig` 命令查看 IP 地址。
3. 在本地计算机使用 SSH 登录 Tinycore，用户名为 `tc`，密码为 `P@ssw0rd`。
4. 依次执行以下命令：

```sh
sudo su
./rploader.sh update now                            #更新 rploader.sh 至最新
./rploader.sh serialgen DS3622xs+                   #生成 DS3622xs+ 的序列号和 MAC 地址，并写入 user_config.json
```

5. 记下上一步生成的 MAC 地址。
6. 用 vi 修改 user_config.json，设置 DiskIdxMap 和 SataPortMap 参数。本文，DiskIdxMap=**310000**，SataPortMap=**144**。
7. 依次执行以下命令：

```sh
./rploader.sh backup now
./rploader.sh build broadwellnk-7.0.1-42218 auto
poweroff                                            #虚拟机关机
```

## 第二次修改虚拟机配置
网卡 MAC 地址改为上一步记下的 MAC 地址。

## 虚拟机第二次开机
1. 约 1 分钟后，本地计算机浏览器访问 <http://find.synology.com>，寻找本地网络中的黑群晖。
2. 找到黑群晖后，按提示上传已下载的 DSM_DS3622xs+\_42218.pat，安装 DSM v7.0.1-42218。
3. 按页面提示等待几分钟后，登录 DSM，按提示进行初始化设置，此处不赘述。
4. 虚拟机关机。

## 第三次修改虚拟机配置
1. 在本地计算机用 SSH 登录到 ESXi，将连接在主板 SATA 接口的三块硬盘分别设置 RDM。使用 `ls -l /vmfs/devices/disks/` 查看硬盘文件名，然后使用如下格式的命令设置 RDM：

```sh
vmkfstools -z /vmfs/devices/disks/[t...] /vmfs/volumes/datastore1/XPEnology/[...]_RDM.vmdk
```

2. 虚拟机添加三块现有硬盘，依次使用上面设置过 RDM 的三个 vmdk 文件，控制器选『SATA 控制器 1:x』，x 从 1 至 3。
3. 虚拟机添加两个 PCI-E 设备：PCI-E 转 SATA 扩展卡、PCI-E 网卡。
4. 如果添加 RDM 硬盘和 PCI-E 设备后，ESXi 报错『Possibly unhandled rejection: {}』，则将已下载的 esxui-offline-bundle-6.x-10692217.zip 上传到 ESXi，以保存在 datastore1 目录为例，执行以下命令安装，安装后重启 ESXi：

```sh
esxcli software vib install -d /vmfs/volumes/datastore1/esxui-offline-bundle-6.x-10692217.zip
```

## 虚拟机第三次开机
1. 开机后，在黑群晖中添加上一步加入的物理硬盘，此处不赘述。
2. 黑群晖安装 Docker 套件。
3. 在黑群晖控制面板中开启 SSH。
4. 在本地计算机用 SSH 登录黑群晖，执行以下命令安装 Open VM Tools：

```
sudo mkdir /root/.ssh
sudo docker run -d --restart=always --net=host -v /root/.ssh/:/root/.ssh/ --name open-vm-tools yalewp/xpenology-open-vm-tools
```

## 参考文献
1. [tinycore-redpill](<https://github.com/pocopico/tinycore-redpill>)
2. [**Xpenology 7.0.1 on ESXi 7.x**](<https://www.tsunati.com/blog/xpenology-7-0-1-on-esxi-7-x>)**（特别重要）**
3. [How to passthrough SATA drives directly on VMWare EXSI 6.5 as RDMs](<https://gist.github.com/Hengjie/1520114890bebe8f805d337af4b3a064>)
4. [docker-xpenology-open-vm-tools](<https://github.com/yale-wp/docker-xpenology-open-vm-tools>)
5. [Experiment on sata_args in grub.cfg](<https://gugucomputing.wordpress.com/2018/11/11/experiment-on-sata_args-in-grub-cfg>)
6. [ESXi 6.7 client GUI broken - cnMaestro OVA upload fails at times](<https://community.cambiumnetworks.com/t/esxi-6-7-client-gui-broken-cnmaestro-ova-upload-fails-at-times/61731>)
