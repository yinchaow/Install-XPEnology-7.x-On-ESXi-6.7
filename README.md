# 在 ESXi 6.7 上安装黑群晖 DSM 7

本文以在 ESXi 6.7 上安装 DS3622xs+ 为例。共有 7 块物理硬盘，其中 3 块连接在主板 SATA 接口，4 块连接在 PCI-E 转 SATA 扩展卡。

|群晖 DS3622xs+ 主要配置|规格|
| ----------- | ----------- |
|CPU|Intel Xeon D-1531 (Broadwell)|
|盘位|12，扩展可至 36|

方法参照 [tmyers07](<https://github.com/tmyers07>) 的[教程](<https://www.tsunati.com/blog/xpenology-7-0-1-on-esxi-7-x>)，稍作改动。

## 下载
- [tinycore-redpill 虚拟硬盘文件](<https://drive.google.com/drive/folders/1nRoggLEVLRbKagIaP3aE28m73agiEGpQ>)（tmyers07 提供，目前版本是 0.4.6）
- [DSM v7.0.1-42218](<https://global.download.synology.com/download/DSM/release/7.0.1/42218/DSM_DS3622xs%2B_42218.pat>)（来自 [群晖官网](<https://archive.synology.com/download/Os/DSM>)）

## 新建虚拟机
1. 在 ESXi 新建虚拟机，此处假定虚拟机名为『XPEnology』。
2. 虚拟机操作系统类型选『Linux』，版本选『其他 4.x 或更高版本 Linux (64位)』。
3. 内存勾选『预选所有客户机内存』。
4. 删除原有硬盘、SCSI 控制器、USB 控制器、光驱。
5. 添加一个 SATA 控制器，此时应共有两个，编号分别是『SATA 控制器 0』和『SATA 控制器 1』。
6. 虚拟机选项 - 引导选项 - 固件，改为『BIOS』。
7. ESXi - 存储 - datastore1 - 数据存储浏览器，在『XPEnology』目录内上传 tinycore-redpill.v0.4.6-flat.vmdk 和 tinycore-redpill.v0.4.6.vmdk。

## 修改虚拟机配置
1. 添加一块现有硬盘，选 tinycore-redpill.v0.4.6.vmdk，控制器选为『SATA 控制器 0:0』。
2. 添加一块标准硬盘，大小可设为 50GB，厚置备延迟置零，控制器选为『SATA 控制器 1:0』。

## 设置 Tinycore Redpill
1. 虚拟机开机，待进入图形界面，在其桌面鼠标右击，在弹出菜单中用键盘方向键依次选 Applications 和 Terminal，打开终端。
2. 在终端用 `ifconfig` 命令查看 IP 地址。
3. 在本地计算机使用 SSH 访问 Tinycore Redpill，用户名为 `tc`，密码为 `P@ssw0rd`。
4. 依次执行以下命令：

```sh
sudo su
./rploader.sh update now                            #更新 rploader.sh 至最新
./rploader.sh serialgen DS3622xs+                   #生成 DS3622xs+ 的序列号和 MAC 地址，并写入 user_config.json
```

5. 记下上一步生成的 MAC 地址。
6. 用 vi 修改 user_config.json，设置 DiskIdxMap 和 SataPortMap 参数。以 4 个 SATA 接口主板加一块 PCI-E 转 4 SATA 接口扩展卡为例，DiskIdxMap=310000，SataPortMap=144。
7. 依次执行以下命令：

```sh
./rploader.sh backup now
./rploader.sh build broadwellnk-7.0.1-42218 auto
poweroff                                            #虚拟机关机
```

## 再次修改虚拟机配置
网卡 MAC 地址改为上一步记下的 MAC 地址。

## 虚拟机再次开机
1. 约 1 分钟后，本地计算机浏览器访问 <http://find.synology.com>，寻找本地网络中的黑群晖。
2. 找到黑群晖后，按提示上传已下载的 DSM_DS3622xs+\_42218.pat，安装 DSM v7.0.1-42218。
3. 按页面提示等待几分钟后，登录 DSM，按提示进行初始化设置，此处不赘述。
4. 虚拟机关机。

## 第三次修改虚拟机配置
1. SSH 登录到 ESXi，将连接在主板 SATA 接口的硬盘设置 RDM，使用如下格式的命令：

`vmkfstools -z /vmfs/devices/disks/t... /vmfs/volumes/datastore1/XPEnology/..._RDM.vmdk`

2. 虚拟机添加现有硬盘，使用上面设置过 RDM 的 vmdk 文件，控制器均选『SATA 控制器 1:x』，x 从 1 递增。
3. 添加 PCI-E 设备，包括 PCI-E 转 SATA 扩展卡和第二块网卡。

## 虚拟机第三次开机
1. 开机后，在黑群晖中添加上一步加入的物理硬盘，此处不赘述。
2. 安装 Docker 套件。
3. 在控制面板中开启 SSH，用 SSH 登录黑群晖，执行以下命令安装 Open VM Tools：

```
sudo mkdir /root/.ssh
sudo docker run -d --restart=always --net=host -v /root/.ssh/:/root/.ssh/ --name open-vm-tools yalewp/xpenology-open-vm-tools
```

## 参考文献
1. [tinycore-redpill](<https://github.com/pocopico/tinycore-redpill>)
2. [**Xpenology 7.0.1 on ESXi 7.x**](<https://www.tsunati.com/blog/xpenology-7-0-1-on-esxi-7-x>)（特别重要）
3. [How to passthrough SATA drives directly on VMWare EXSI 6.5 as RDMs](<https://gist.github.com/Hengjie/1520114890bebe8f805d337af4b3a064>)
4. [docker-xpenology-open-vm-tools](https://github.com/yale-wp/docker-xpenology-open-vm-tools>)
