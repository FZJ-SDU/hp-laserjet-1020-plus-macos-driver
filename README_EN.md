**English** | [中文](./README.md)

# HP LaserJet 1020 Plus Driver Installation Guide

## GitHub Repository

**Download: https://github.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver**

---

## Background

HP LaserJet 1020 Plus is an older printer model that HP no longer supports with official macOS drivers. This printer uses the Zenographics ZJS protocol and requires the open-source foo2zjs driver. Additionally, **firmware must be uploaded each time the printer is powered on** for it to function properly.

This solution implements **automatic firmware upload** - the firmware is automatically sent before each print job, eliminating the need for manual intervention.

---

## Files

| File | Description | Download |
|------|-------------|----------|
| `sihp1020.dl` | HP 1020 printer firmware | [Download](https://raw.githubusercontent.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver/main/sihp1020.dl) |
| `foo2zjs` | Filter that converts PBM format to ZJS format | [Download](https://raw.githubusercontent.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver/main/foo2zjs) |
| `foomatic-rip` | CUPS filter script (auto firmware upload + format conversion) | [Download](https://raw.githubusercontent.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver/main/foomatic-rip) |
| `gs-static` | Statically compiled Ghostscript (converts PDF/PS to PBM) | [Download](https://raw.githubusercontent.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver/main/gs-static) |
| `HP-LaserJet_1020.ppd` | Printer description file | [Download](https://raw.githubusercontent.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver/main/HP-LaserJet_1020.ppd) |

---

## Installation Steps

### Step 1: Download Files

```bash
# Clone the repository
git clone https://github.com/FZJ-SDU/hp-laserjet-1020-plus-macos-driver.git
cd hp-laserjet-1020-plus-macos-driver
```

### Step 2: Install Files to System Directories

Open Terminal and run the following commands:

```bash
# Create directory
sudo mkdir -p /usr/local/share/foo2zjs/firmware

# Copy firmware file
sudo cp sihp1020.dl /usr/local/share/foo2zjs/firmware/

# Copy static Ghostscript
sudo cp gs-static /usr/local/bin/
sudo chmod +x /usr/local/bin/gs-static

# Copy foo2zjs filter
sudo cp foo2zjs /usr/libexec/cups/filter/
sudo chmod +x /usr/libexec/cups/filter/foo2zjs

# Copy foomatic-rip filter
sudo cp foomatic-rip /usr/libexec/cups/filter/
sudo chmod +x /usr/libexec/cups/filter/foomatic-rip

# Copy PPD file
sudo cp HP-LaserJet_1020.ppd /Library/Printers/PPDs/Contents/Resources/
```

### Step 3: Restart CUPS Print Service

```bash
sudo killall -HUP cupsd
```

### Step 4: Connect Printer and Add to System

1. Connect HP LaserJet 1020 Plus to your Mac via USB
2. Open **System Settings → Printers & Scanners**
3. Click **+** to add a printer
4. Select **HP LaserJet 1020**
5. If prompted to select a driver, choose **HP LaserJet 1020 Foomatic/foo2zjs-z1**

### Step 5: Test Print

Print from any application - the firmware will be uploaded automatically.

---

## One-Line Installation

Copy and paste the following into Terminal:

```bash
# Navigate to download directory
cd ~/Downloads/hp-laserjet-1020-plus-macos-driver

# Or use the cloned directory
# cd hp-laserjet-1020-plus-macos-driver

# One-line installation
sudo mkdir -p /usr/local/share/foo2zjs/firmware
sudo cp sihp1020.dl /usr/local/share/foo2zjs/firmware/
sudo cp gs-static /usr/local/bin/ && sudo chmod +x /usr/local/bin/gs-static
sudo cp foo2zjs /usr/libexec/cups/filter/ && sudo chmod +x /usr/libexec/cups/filter/foo2zjs
sudo cp foomatic-rip /usr/libexec/cups/filter/ && sudo chmod +x /usr/libexec/cups/filter/foomatic-rip
sudo cp HP-LaserJet_1020.ppd /Library/Printers/PPDs/Contents/Resources/

sudo killall -HUP cupsd

echo "Installation complete! Please add the printer in System Settings."
```

---

## Uninstallation

To uninstall, run the following commands:

```bash
sudo rm -f /usr/local/share/foo2zjs/firmware/sihp1020.dl
sudo rm -f /usr/local/bin/gs-static
sudo rm -f /usr/libexec/cups/filter/foo2zjs
sudo rm -f /usr/libexec/cups/filter/foomatic-rip
sudo rm -f /Library/Printers/PPDs/Contents/Resources/HP-LaserJet_1020.ppd
sudo killall -HUP cupsd
```

Then remove the printer in System Settings.

---

## Troubleshooting

### Issue: Distorted Print Output

Check the paper size setting in foomatic-rip. The default is A4 (-p9). If using Letter paper, change to `-p1`.

### Issue: Printer Not Responding

1. Check if printer is properly connected: `lpinfo -v | grep 1020`
2. Check print queue: `lpstat -o`
3. View error log: `tail -50 /var/log/cups/error_log`

### Issue: Filter Failed

Check file permissions:
```bash
ls -la /usr/libexec/cups/filter/foomatic-rip
ls -la /usr/libexec/cups/filter/foo2zjs
ls -la /usr/local/bin/gs-static
```

Ensure all files have execute permissions (-rwxr-xr-x).

---

## About HP Official Drivers

The HP official driver package does not include support for HP LaserJet 1020:
- HP LaserJet 1020 uses the special Zenographics ZJS protocol
- HP official drivers attempt to use the CP1022 driver as a substitute, but this causes errors
- The foo2zjs open-source driver is required for proper functionality

**This solution uses entirely open-source drivers and does not depend on the HP official driver package.**

---

## Technical Details

1. **Firmware Upload**: The HP 1020 printer has no internal persistent storage, so firmware must be uploaded via USB each time it's powered on
2. **Print Pipeline**: PDF/PS → Ghostscript(pbmraw) → foo2zjs(ZJS) → Printer
3. **Automation**: The foomatic-rip filter automatically sends firmware before each print job

---

## Compatible Systems

- macOS Sequoia (15.x)
- macOS Sonoma (14.x)
- macOS Ventura (13.x)
- Other versions untested but should theoretically work

---

## License

- `foo2zjs`: GPL v2
- `Ghostscript`: AGPL v3
- `sihp1020.dl`: HP proprietary firmware
- Other files: MIT License

---

## References

- [foo2zjs Project](https://github.com/OpenPrinting/foo2zjs)
- [OpenPrinting](https://openprinting.org/)
- [Apple Discussions Original Thread](https://discussions.apple.com/thread/255831271)