[English](./README_EN.md) | **中文**

# HP LaserJet 1020 Plus 驱动安装说明

## GitHub 仓库

**下载地址：https://github.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver**

---

## 背景

HP LaserJet 1020 Plus 是一款老旧的打印机，HP 官方已不再提供 macOS 驱动支持。该打印机采用 Zenographics ZJS 协议，需要使用开源的 foo2zjs 驱动，并且**每次打印机开机后需要上传固件**才能正常工作。

本方案已实现**自动上传固件**，每次打印时会自动完成固件上传，无需手动操作。

---

## 文件说明

| 文件名 | 说明 | 下载链接 |
|--------|------|----------|
| `sihp1020.dl` | HP 1020 打印机固件 | [下载](https://raw.githubusercontent.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver/main/sihp1020.dl) |
| `foo2zjs` | 将 PBM 格式转换为 ZJS 格式的过滤器 | [下载](https://raw.githubusercontent.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver/main/foo2zjs) |
| `foomatic-rip` | CUPS 打印过滤器脚本（自动发送固件+转换格式） | [下载](https://raw.githubusercontent.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver/main/foomatic-rip) |
| `gs-static` | 静态编译的 Ghostscript（将 PDF/PS 转换为 PBM） | [下载](https://raw.githubusercontent.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver/main/gs-static) |
| `HP-LaserJet_1020.ppd` | 打印机描述文件 | [下载](https://raw.githubusercontent.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver/main/HP-LaserJet_1020.ppd) |

---

## 安装步骤

### 步骤 1：下载文件

```bash
# 克隆仓库
git clone https://github.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver.git
cd hp-laserjet-1020-plus-macos-driver
```

### 步骤 2：安装文件到系统目录

```bash
# 创建目录
sudo mkdir -p /usr/local/share/foo2zjs/firmware

# 复制固件文件
sudo cp sihp1020.dl /usr/local/share/foo2zjs/firmware/

# 复制静态 Ghostscript
sudo cp gs-static /usr/local/bin/
sudo chmod +x /usr/local/bin/gs-static

# 复制 foo2zjs 过滤器
sudo cp foo2zjs /usr/libexec/cups/filter/
sudo chmod +x /usr/libexec/cups/filter/foo2zjs

# 复制 foomatic-rip 过滤器
sudo cp foomatic-rip /usr/libexec/cups/filter/
sudo chmod +x /usr/libexec/cups/filter/foomatic-rip

# 复制 PPD 文件
sudo cp HP-LaserJet_1020.ppd /Library/Printers/PPDs/Contents/Resources/
```

### 步骤 3：重启 CUPS 打印服务

```bash
sudo killall -HUP cupsd
```

### 步骤 4：连接打印机并添加

1. 用 USB 线连接 HP LaserJet 1020 Plus 到 Mac
2. 打开 **系统设置 → 打印机与扫描仪**
3. 点击 **+** 添加打印机
4. 选择 **HP LaserJet 1020**
5. 如果提示选择驱动，选择 **HP LaserJet 1020 Foomatic/foo2zjs-z1**

### 步骤 5：测试打印

直接从任意应用程序打印即可，固件会自动上传。

---

## 一键安装命令

```bash
cd ~/Desktop/HP\ Laser\ Jet\ 1020\ plus驱动安装

sudo mkdir -p /usr/local/share/foo2zjs/firmware
sudo cp sihp1020.dl /usr/local/share/foo2zjs/firmware/
sudo cp gs-static /usr/local/bin/ && sudo chmod +x /usr/local/bin/gs-static
sudo cp foo2zjs /usr/libexec/cups/filter/ && sudo chmod +x /usr/libexec/cups/filter/foo2zjs
sudo cp foomatic-rip /usr/libexec/cups/filter/ && sudo chmod +x /usr/libexec/cups/filter/foomatic-rip
sudo cp HP-LaserJet_1020.ppd /Library/Printers/PPDs/Contents/Resources/

sudo killall -HUP cupsd

echo "安装完成！请在系统设置中添加打印机。"
```

---

## 卸载方法

```bash
sudo rm -f /usr/local/share/foo2zjs/firmware/sihp1020.dl
sudo rm -f /usr/local/bin/gs-static
sudo rm -f /usr/libexec/cups/filter/foo2zjs
sudo rm -f /usr/libexec/cups/filter/foomatic-rip
sudo rm -f /Library/Printers/PPDs/Contents/Resources/HP-LaserJet_1020.ppd
sudo killall -HUP cupsd
```

然后在系统设置中删除打印机。

---

## 故障排除

### 问题：打印出来格式变形

检查 foomatic-rip 中的纸张设置。默认为 A4（-p9），如使用 Letter 纸张，改为 `-p1`。

### 问题：打印机无反应

1. 检查打印机是否正确连接：`lpinfo -v | grep 1020`
2. 检查打印队列：`lpstat -o`
3. 查看错误日志：`tail -50 /var/log/cups/error_log`

### 问题：Filter failed

检查文件权限：
```bash
ls -la /usr/libexec/cups/filter/foomatic-rip
ls -la /usr/libexec/cups/filter/foo2zjs
ls -la /usr/local/bin/gs-static
```

确保所有文件都有执行权限（-rwxr-xr-x）。

---

## 关于 HP 官方驱动

HP 官方驱动包不包含 HP LaserJet 1020 的支持：
- HP LaserJet 1020 使用特殊的 Zenographics ZJS 协议
- HP 官方驱动尝试使用 CP1022 驱动替代，但会导致错误
- 必须使用 foo2zjs 开源驱动才能正常工作

---

## 技术原理

1. **固件上传**：HP 1020 打印机内部没有持久存储，每次开机需要通过 USB 上传固件
2. **打印流程**：PDF/PS → Ghostscript(pbmraw) → foo2zjs(ZJS) → 打印机
3. **自动化**：foomatic-rip 过滤器在每次打印前自动发送固件

---

## 适用系统

- macOS Sequoia (15.x)
- macOS Sonoma (14.x)
- macOS Ventura (13.x)

---

## 参考资料

- [foo2zjs 项目](https://github.com/OpenPrinting/foo2zjs)
- [OpenPrinting](https://openprinting.org/)
