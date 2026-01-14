<div align="center">
  <img src="https://github.com/CachyOS/calamares-config/blob/grub-3.2/etc/calamares/branding/cachyos/logo.png" width="64" alt="CachyOS logo"></img>
  <br/>
  <h1 align="center">CachyOS Linux 6.19 RC - Personal Testing Fork</h1>
  <p align="center">Custom build of CachyOS Linux 6.19-rc5 for testing and integration with <a href="https://github.com/hello-dan-codes/bazzite">Bazzite</a>.</p>
</div>

This repository is a personal testing fork for building and integrating CachyOS Linux 6.19-rc5 with [Bazzite](https://github.com/hello-dan-codes/bazzite).

**Note:** This uses Linux kernel 6.19-rc5 built from git, which is a release candidate version and may be unstable. Use at your own risk for testing purposes only.

## 📋 Patches Applied

This kernel includes the following patch sources:

1. **CachyOS Base Patches** (`Patch0`) - CachyOS kernel optimizations and tweaks
2. **BORE Scheduler** (`Patch1`) - BORE CPU scheduler with sched-ext support
3. **MSI WMI Platform Patches** (`Patch2`) - 12 patches from Bazzite for MSI Claw 8 AI+ support:
   - MSI Claw device detection and quirk system
   - Platform profile via Shift Mode (Sport/Comfort/Green/Eco/User modes)
   - Fan control and sensor access
   - TDP/power tunable support
   - Battery charge threshold control
4. **Additional MSI Patches** (`Patch3`) - 3 supplementary patches:
   - MSI Center button support
   - Keyboard platform profile support
   - MSI platform brightness control

- [Kernels](#-kernels)
  - [Features](#-features)
  - [Patches Applied](#-patches-applied)
  - [Installation Instructions](#%EF%B8%8F-installation-instructions)
    - [Default Kernel](#default-kernel)
- [Addons](#-addons)
  - [CachyOS-Settings](#cachyos-settings)
  - [scx-scheds](#scx-scheds)
  - [scx-manager](#scx-manager)
  - [ananicy-cpp](#ananicy-cpp)

# 🐧 Kernel

This repository builds a single CachyOS kernel variant:
- `kernel-cachyos` - 1000 Hz kernel with BORE scheduler, built from Linux 6.19-rc5 git snapshot

## 🌟 Features
- Choose between `GCC` and `LLVM-ThinLTO`
- Optimized for `x86-64v3` CPUs for `kernel-cachyos` and `x86-64v2` for `kernel-cachyos-lts` and `kernel-cachyos-server`
- **Intel Lunar Lake optimizations** (x86-64-v4 ISA support, P-State, TPM 2.0, Thunderbolt)
- **MSI Claw 8 AI+ platform support** - Full WMI platform driver integration with:
  - Shift Mode (Platform Profile) support for power management
  - Fan control and sensor access
  - Firmware-level power tunables
  - Battery charging threshold control
  - TDP/power mode switching (Sport/Comfort/Green/Eco/User modes)
- BORE scheduler with sched-ext support (excl. `kernel-cachyos-server`, sched-ext support only for `kernel-cachyos`)
- AMD P-State Preferred Core, AMD CPB Switch and upstream `amd-pstate` enchancements (exclusive to `kernel-cachyos`)
- Cachy Sauce - Provides tweaks for the scheduler and other settings
- Latest & improved ZSTD patchset
- Improved BFQ Scheduler
- BBRv3 tcp_congestion_control
- v4l2loopback modules as default included
- Cherry picked patches from Clear Linux
- Backported patches from `linux-next`
- OpenRGB and ACS Override support
- NTSync patched and integrated into the kernel (exclusive to `kernel-cachyos`)

## ⬇️ Installation Instructions
Make sure your CPU supports the higher target `x86-64` architectures. You need minimum `x86-64-v3` for all kernels, except `kernel-cachyos-lts` and `kernel-cachyos-server` that only require `x86-64-v2`.
```bash
/lib64/ld-linux-x86-64.so.2 --help | grep "(supported, searched)"
```

Next, enable the COPR repository hosting the kernels.
```bash
sudo dnf copr enable bieszczaders/kernel-cachyos # For GCC built kernels
# or
sudo dnf copr enable bieszczaders/kernel-cachyos-lto # For LLVM-ThinLTO build kernels
```

Now you can install the kernels
```bash
sudo dnf install kernel-cachyos kernel-cachyos-devel-matched # For GCC built kernels
# or
sudo dnf install kernel-cachyos-lto kernel-cachyos-lto-devel-matched # For LLVM-ThinLTO built kernels

## LTS Kernel
sudo dnf install kernel-cachyos-lts kernel-cachyos-lts-devel-matched
# or
sudo dnf install kernel-cachyos-lts-lto kernel-cachyos-lts-lto-devel-matched

## Real-time Kernel
sudo dnf install kernel-cachyos-rt kernel-cachyos-rt-devel-matched

## Server Kernel
sudo dnf install kernel-cachyos-server kernel-cachyos-server-devel-matched
```

🚨 Lastly if you use SELinux, you need to enable the necessary policy to be able to load kernel modules.
```bash
sudo setsebool -P domain_kernel_load_modules on
```

### Fedora Silverblue
```bash
cd /etc/yum.repos.d/
sudo wget https://copr.fedorainfracloud.org/coprs/bieszczaders/kernel-cachyos/repo/fedora-$(rpm -E %fedora)/bieszczaders-kernel-cachyos-fedora-$(rpm -E %fedora).repo
sudo rpm-ostree override remove kernel kernel-core kernel-modules kernel-modules-core kernel-modules-extra --install kernel-cachyos
sudo systemctl reboot
```

### Default Kernel
By default Fedora will use the kernel that was most recently updated by `dnf` which will lead to inconsistent behaviour if you have multiple kernels installed, but we can tell Fedora to always boot with the latest CachyOS kernel by running a script after kernel updates.

Create a file in `/etc/kernel/postinst.d`:
```bash
sudo nano /etc/kernel/postinst.d/99-default
```

Enter the following content that will set the latest CachyOS kernel as the default kernel:
```bash
#!/bin/sh

set -e

grubby --set-default=/boot/$(ls /boot | grep vmlinuz.*cachy | sort -V | tail -1)
```

Make `root` the owner and make the script executable:
```bash
sudo chown root:root /etc/kernel/postinst.d/99-default ; sudo chmod u+rx /etc/kernel/postinst.d/99-default
```

The next time any installed kernel (e.g. the official Fedora kernel) gets an update, the system will change default kernel back to the latest CachyOS kernel. This way you can keep the official kernel as a backup in case an update goes wrong and you need to temporarily switch to the official kernel.

# 🧩 Addons
We provide a few addons that supplement the kernel packages and system.
- [CachyOS-Settings](https://github.com/CachyOS/CachyOS-Settings) - Settings used in CachyOS (includes modprobe config, udev rules, etc) packaged for Fedora.
- [scx-scheds](https://github.com/sched-ext/scx) - sched-ext schedulers. Provides both `scx-scheds` releases and `scx-scheds-git` package.
- [scx-manager](https://github.com/CachyOS/scx-manager/) - Simple GUI for managing sched-ext schedulers via scx_loader.
- [ananicy-cpp](https://gitlab.com/ananicy-cpp/ananicy-cpp/) & [cachyos-ananicy-rules](https://github.com/CachyOS/ananicy-rules) - Auto nice daemon with rules support.

## ⬇️ Installation instructions
First, enable the COPR repository hosting addon packages.
```bash
sudo dnf copr enable bieszczaders/kernel-cachyos-addons
```

Now you can install the addon packages.

### CachyOS-Settings
```bash
sudo dnf swap zram-generator-defaults cachyos-settings
sudo dracut -f
```

### scx-scheds and scx-tools
```bash
sudo dnf install scx-scheds scx-tools
```
or -git packages
```bash
sudo dnf install scx-scheds-git scx-tools-git

```
**For Fedora Silverblue / Kinoite:**

```bash
sudo rpm-ostree install scx-scheds scx-tools
sudo systemctl reboot
```
or -git packages
```bash
sudo rpm-ostree install scx-scheds-git scx-tools-git
sudo systemctl reboot
```

You can use [scxctl](https://github.com/sched-ext/scx-loader/blob/main/crates/scxctl/README.md) to start/change the scheduler with profiles/custom flags.

📖 Usage guide available in the [CachyOS wiki](https://wiki.cachyos.org/configuration/sched-ext/).

Starting with version 1.0.18, scx_loader and scxctl have been moved to a separate repository. Remember to install `scx-tools` if you plan to continue using these tools! 

### scx-manager

```
sudo dnf install scx-manager
```

### ananicy-cpp

> [It is advised against running `ananicy-cpp` and a scheduler from the `sched-ext` framework *simultaneously*. Use one but not the other.](https://wiki.cachyos.org/configuration/sched-ext/#disable-ananicy-cpp)

```bash
sudo dnf install ananicy-cpp
sudo systemctl enable --now ananicy-cpp
```
