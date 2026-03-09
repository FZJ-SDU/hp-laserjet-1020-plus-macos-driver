# HP LaserJet 1020 Plus 驱动安装说明

## 背景

HP LaserJet 1020 Plus 是一款老旧的打印机，HP 官方已不再提供 macOS 驱动支持。该打印机采用 Zenographics ZJS 协议，需要使用开源的 foo2zjs 驱动，并且**每次打印机开机后需要上传固件**才能正常工作。

本方案已实现**自动上传固件**，每次打印时会自动完成固件上传，无需手动操作。

---

## 文件说明

| 文件名 | 说明 |
|--------|------|
| `sihp1020.dl` | HP 1020 打印机固件（每次开机需要上传到打印机） |
| `foo2zjs` | 将 PBM 格式转换为 ZJS 格式的过滤器 |
| `foomatic-rip` | CUPS 打印过滤器脚本（自动发送固件+转换格式） |
| `gs-static` | 静态编译的 Ghostscript（将 PDF/PS 转换为 PBM） |
| `HP-LaserJet_1020.ppd` | 打印机描述文件 |

---

## 安装步骤

### 步骤 1：安装文件到系统目录

打开终端，执行以下命令：

```bash
# 创建目录
sudo mkdir -p /usr/local/share/foo2zjs/firmware

# 复制固件文件
sudo cp ~/Desktop/HP\ Laser\ Jet\ 1020\ plus驱动安装/sihp1020.dl /usr/local/share/foo2zjs/firmware/

# 复制静态 Ghostscript
sudo cp ~/Desktop/HP\ Laser\ Jet\ 1020\ plus驱动安装/gs-static /usr/local/bin/
sudo chmod +x /usr/local/bin/gs-static

# 复制 foo2zjs 过滤器
sudo cp ~/Desktop/HP\ Laser\ Jet\ 1020\ plus驱动安装/foo2zjs /usr/libexec/cups/filter/
sudo chmod +x /usr/libexec/cups/filter/foo2zjs

# 复制 foomatic-rip 过滤器
sudo cp ~/Desktop/HP\ Laser\ Jet\ 1020\ plus驱动安装/foomatic-rip /usr/libexec/cups/filter/
sudo chmod +x /usr/libexec/cups/filter/foomatic-rip

# 复制 PPD 文件
sudo cp ~/Desktop/HP\ Laser\ Jet\ 1020\ plus驱动安装/HP-LaserJet_1020.ppd /Library/Printers/PPDs/Contents/Resources/
```

### 步骤 2：重启 CUPS 打印服务

```bash
sudo killall -HUP cupsd
```

### 步骤 3：连接打印机并添加

1. 用 USB 线连接 HP LaserJet 1020 Plus 到 Mac
2. 打开 **系统设置 → 打印机与扫描仪**
3. 点击 **+** 添加打印机
4. 选择 **HP LaserJet 1020**
5. 如果提示选择驱动，选择 **HP LaserJet 1020 Foomatic/foo2zjs-z1**

### 步骤 4：测试打印

直接从任意应用程序打印即可，固件会自动上传。

---

## 一键安装命令

将以下内容复制到终端执行：

```bash
# 一键安装脚本
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

如需卸载，执行以下命令：

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

## 关于 Pacifist 安装的 HP 驱动

通过 Pacifist 安装的 HP 官方驱动（CP1020 系列）**在本方案中没有被使用**。原因是：

1. HP 官方驱动不包含 HP LaserJet 1020 的支持
2. HP LaserJet 1020 使用特殊的 ZJS 协议，需要 foo2zjs 开源驱动
3. HP 官方驱动尝试使用 CP1022 驱动替代，但会导致 "Filter failed" 错误

因此，最终方案完全使用 foo2zjs 开源驱动，不依赖 HP 官方驱动包。

---

## 文件来源

| 文件 | 来源 |
|------|------|
| sihp1020.dl | https://github.com/OpenPrinting/foo2zjs |
| foo2zjs | 从 foo2zjs 源码编译 |
| gs-static | 从 Ghostscript 10.06.0 源码静态编译 |
| foomatic-rip | 自定义脚本 |
| HP-LaserJet_1020.ppd | foo2zjs 项目 |

---

## 技术原理

1. **固件上传**：HP 1020 打印机内部没有持久存储，每次开机需要通过 USB 上传固件
2. **打印流程**：PDF/PS → Ghostscript(pbmraw) → foo2zjs(ZJS) → 打印机
3. **自动化**：foomatic-rip 过滤器在每次打印前自动发送固件

