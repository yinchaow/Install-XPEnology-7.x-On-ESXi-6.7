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
- [黑群晖系统](<https://global.download.synology.com/download/DSM/release/6.2.3/25426/DSM_DS3615xs_25426.pat>)
- [OSFMount](<https://www.osforensics.com/tools/mount-disk-images.html>)

## 安装黑群晖
### 修改 synoboot.vmdk
1. 在 ESXi 新建虚拟机

## 参考文献
1. [Tutorial: Install DSM 6.2 on ESXi 6.7](<https://xpenology.com/forum/topic/13061-tutorial-install-dsm-62-on-esxi-67/>)
2. [Tutorial/Reference: 6.x Loaders and Platforms](<https://xpenology.com/forum/topic/13333-tutorialreference-6x-loaders-and-platforms/>)
