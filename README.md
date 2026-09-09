# pve9-dg1-passthrough

PVE9.1 DG1 直通补丁

内核版本`7.0.0-3-pve`

DG1 PCIE reset 命令存在问题，需要在内核层补丁拦截发给它的reset

DG1 固件中cfg区存在越界问题，硬编码 `cfg_size = PCI_CFG_SPACE_SIZE`

确认在[pve-kernel](https://git.proxmox.com/git/pve-kernel.git) commit `6da668c59515366142ee321c75ceb0504120d0e8` 可用

将patch放到`pve-kernel/patches/kernel`目录，然后重新`make`即可

Fork自[YK-Samgo/pve9-dg1-passthrough](https://github.com/YK-Samgo/pve9-dg1-passthrough)，修改了补丁以支持Linux Kernel 7.x。

## 已知问题

1. 启动后似乎会导致对应虚拟机控制台无法启动，报错 `VM 100 qmp command 'set_password' failed - Could not set password TASK ERROR: Failed to run vncproxy`，不影响别的虚拟机

1. 如果遇到DG1卡住，需要重启PVE

## 编译建议

给内核版本添加一个后缀用于和官方版本做区分

1. 编辑debian/changelog，添加发布信息

2. 编辑`pve-kernel/Makefile:13 KREL_EXTRA=-dg1`

## GitHub Actions 自动编译

仓库提供了基于 Docker Debian Trixie 环境的 GitHub Actions 流水线，适用于 PVE 9
对应的 7.x 内核。进入仓库的
`Actions`，选择 `Build DG1 PVE kernel`，点击 `Run workflow`，然后输入要编译的
PVE 内核包版本，例如：

```text
7.0.14-16-pve
```

工作流会在 Proxmox `pve-kernel` 的提交历史中定位对应的 changelog 版本，检出匹配的
源码和子模块，应用本仓库补丁，并编译 amd64 的内核、headers 和 modules Debian 包。
成功后会创建类似下面的 GitHub Release：

```text
pve-7.0.14-16-pve-dg1
```

Release 中包含所有生成的 `.deb`、`SHA256SUMS` 和 `build-info.txt`。安装前可以使用：

```bash
sha256sum -c SHA256SUMS
```

输入版本应使用 PVE 节点显示的完整版本号，例如 `7.0.14-16-pve`。构建失败时不会创建
Release；已经存在同名 Release 时，工作流也会拒绝覆盖。
