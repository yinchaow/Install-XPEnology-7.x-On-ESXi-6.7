# 在 ESXi 6.7 上安装黑群晖
## 确定黑群晖版本
黑群晖常见版本有三个，根据 [flyride](<https://xpenology.com/forum/profile/39776-flyride/>) 整理的引导程序与硬件平台对照表格（截至 2020 年 12 月 29 日）选择最合适的方案。

| 选项排名 | DSM 平台 | DSM 版本 | [引导程序](<https://xpenology.com/forum/topic/12952-dsm-62-loader/>) | 启动方式 | 硬件转码支持 | NVMe 缓存支持 | RAIDF1 支持 | **支持的 CPU** | 备注 |
| ------ | -------- | ------- | ------- | ------- | ---------- | ----------- | ----------- | ------------- | --- |
| 1，3a | DS918+ | 6.2.0 至 6.2.3 | 1.04b | UEFI，BIOS/CSM | 是 | 是 | 否 | [Haswell](<https://www.intel.cn/content/www/cn/zh/ark/products/codename/42174/haswell.html>)及后续 | 推荐 6.2.0 和 6.2.3，不推荐将 6.2.1/6.2.2 用于全新安装 |
| 2，3b | DS3617xs | 6.2.0 至 6.2.3 | 1.03b | 仅 BIOS/CSM | 否 | 否 | 是 | 任何 x86-64 | 推荐 6.2.0 和 6.2.3，不推荐将 6.2.1/6.2.2 用于全新安装 |
| | DS3615xs | 6.2.0 至 6.2.3 | 1.03b | BIOS/CSM | 否 | 否 | 是 | 任何 x86-64 | 推荐 6.2.0 和 6.2.3，不推荐将 6.2.1/6.2.2 用于全新安装 |

本文以 DS3617xs 为例。

## 下载
- [synoboot.vmdk](<./Files/synoboot_3615.zip>)
- [juns loader for DSM 6.2](<./Files/DS3615xs 6.0.2 Jun's Mod V1.01.zip>)
- [黑群晖系统 v6.2.3-25426](<https://global.download.synology.com/download/DSM/release/6.2.3/25426/DSM_DS3617xs_25426.pat>)
- [黑群晖系统 v6.2.3-25426 Update 3](<https://global.download.synology.com/download/DSM/criticalupdate/update_pack/25426-3/synology_broadwell_3617xs.pat>)
- [OSFMount](<https://www.osforensics.com/tools/mount-disk-images.html>)
- [FixSynoboot.sh](<https://xpenology.com/forum/topic/28183-running-623-on-esxi-synoboot-is-broken-fix-available/>)，需登录论坛

## 安装黑群晖
### 修改 synoboot.img
1. 用 OSFMount 加载 synoboot.img，不勾选*Read-only*
2. 修改 grub/grub.cfg，包括以下内容：
  - mac1
  - sata_args 参数修改 `DiskIdxMap=310000 SataPortMap=144`
3. 注释无关启动项
```
#menuentry "DS3617xs 6.2 Baremetal $VERSION" --class os {
#        set img=
#        savedefault
#        loadlinux 3615 usb
#        loadinitrd
#        showtips
#}
#
#menuentry "DS3617xs 6.2 Baremetal $VERSION Reinstall" --class os {
#        set img=
#        loadlinux 3615 usb mfg
#        loadinitrd
#        showtips
#}
#
#menuentry "DS3617xs 6.2 Baremetal AMD $VERSION" --class os {
#        set img=
#        set zImage=bzImage
#        savedefault
#        loadlinux 3615 usb
#        loadinitrd
#        showtips
#}

menuentry "DS3617xs 6.2 VMWare/ESXI $VERSION" --class os {
        set img=
        savedefault
        loadlinux 3615 sata
        loadinitrd
        showtips
}
```

### 新建虚拟机
1. 客户端操作系统选 *Linux*，版本选*其他 3.x Linux (64位)*
2. 

### DSM 升级至 v6.2.3-25426 Update 3
1. 下载 FixSynoboot.sh，上传至黑群晖
2. `sudo cp FixSynoboot.sh /usr/local/etc/rc.d`
3. `sudo chmod 755 /usr/local/etc/rc.d/FixSynoboot.sh`
4. 重启黑群晖
5. 控制面板 - 更新和还原 - 手动更新 DSM - 选择已下载的 synology_broadwell_3617xs.pat
6. 升级成功

## 参考文献
1. [Tutorial: Install DSM 6.2 on ESXi 6.7](<https://xpenology.com/forum/topic/13061-tutorial-install-dsm-62-on-esxi-67/>)
2. [Tutorial/Reference: 6.x Loaders and Platforms](<https://xpenology.com/forum/topic/13333-tutorialreference-6x-loaders-and-platforms/>)
3. [Running 6.2.3 on ESXi? Synoboot is BROKEN, fix available](<https://xpenology.com/forum/topic/28183-running-623-on-esxi-synoboot-is-broken-fix-available/>)
