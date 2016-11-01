---
layout: post
title: Bootloader
---

1. UEFI (also named as EFI) firmware on system board is the primary bootloader.
   1. [*sys-boot/efibootmgr*](https://wiki.gentoo.org/wiki/Efibootmgr) application is not a bootloader; it is a tool to interact with the UEFI firmware and update its settings.
   2. Enable *CONFIG_EFI_VARS* so that the system firmware can be manipulated via *efibootmgr*.
2. Grub/LILO etc. are secondary bootloaders with more flexible extensions.
   1. Secondary bootloader is not a requirement, even for multiple boot entries.
3. The kernel must have specific options enabled to be [directly](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Bootloader#Alternative_2:_efibootmgr) bootable by the system's UEFI firmware (*CONFIG_EFI* and *CONFIG_EFI_STUB*).
   1. Be sure to read though the [EFI stub kernel](https://wiki.gentoo.org/wiki/EFI_stub_kernel) article before continuing.
   2. To reiterate, *efibootmgr* is not a requirement to directly boot an UEFI system. The Linux kernel itself can be booted immediately.
   3. Additional kernel command-line options can be built-in to the Linux kernel (*CONFIG_CMDLINE*).
   4. Even an *initramfs* can be 'built-in' to the kernel, which is not recommended. To support *initramfs* directly, passing a parameter to the primary UEFI bootloader with the help of *efibootmgr*.
   5. For additional kernels or Windows, enable *CONFIG_EFI_VARS*. Then create another UEFI entries by *efibootmgr*.
