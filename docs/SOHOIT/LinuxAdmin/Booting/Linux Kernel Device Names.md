# Linux Kernel Device Names

It is one of the most common points of confusion in Linux storage. Just like GRUB assigns its own `(hdX)` numbers, the **Linux kernel** assigns its own `/dev/sdX` or `/dev/nvmeX` identifiers when it boots—and these are **not guaranteed to be the same** between different OS installations.

Here is why your "Drive B" might show up as `/dev/sdb` in one OS and `/dev/sdc` in another.

---

## 1. Kernel Discovery Order

When a Linux OS boots, the kernel scans the hardware controllers (SATA, NVMe, USB) to find drives. It assigns letters (`sda`, `sbb`, `sdc`) based on the order it "settles" the hardware.

* **Driver Load Speed:** If OS #1 loads the NVMe driver faster than the SATA driver, the NVMe drive becomes `sda`.
* **Kernel Version:** A newer kernel in OS #2 might initialize a PCIe controller in a different order than an older kernel in OS #1, swapping `sdb` and `sdc`.
* **Hardware Ports:** Physical port 0 is usually `sda`, but if you have a failing cable or a slow-spinning mechanical drive, the kernel might skip it temporarily and assign that letter to the next available drive it finds.

---

## 2. The Danger of `/dev/sdX`

Because these names are "volatile" (they can change), modern Linux systems **never** use `/dev/sdb` to mount your filesystems in `/etc/fstab`.

If OS #1 thinks the drive is `/dev/sdb` but OS #2 thinks it is `/dev/sdc`, using the letter would cause the boot to fail or, worse, mount the wrong drive.

### How Linux handles this (The Solution)

Linux uses **UUIDs** (Universally Unique Identifiers) inside the `/etc/fstab` file.

| Method | Example | Stability |
| --- | --- | --- |
| **Kernel Name** | `/dev/sdb1` | **Unstable**: Can change every reboot. |
| **UUID** | `UUID=550e8400-e29b...` | **Permanent**: Tied to the filesystem metadata. |
| **Label** | `LABEL=DATA_DRIVE` | **Stable**: Tied to the name you gave the partition. |

---

## 3. Visualizing the Conflict

Imagine you have two SSDs. Here is how they might appear to two different Linux menus:

**Booting into OS #1 (Ubuntu):**

* SSD A (System): `/dev/sda`
* SSD B (Storage): `/dev/sdb`

**Booting into OS #2 (Fedora):**

* SSD B (System): `/dev/sda`  <-- *Because it booted from this drive, it saw it first.*
* SSD A (Storage): `/dev/sdb`

Even though the **letters** swapped, the **UUIDs** remain identical. Both operating systems will successfully find their files as long as they look for the UUID.

---

## Summary

* **GRUB** sees: `(hd0)`, `(hd1)` — Based on BIOS boot order.
* **Linux Kernel** sees: `/dev/sda`, `/dev/sdb` — Based on driver initialization order.
* **The Reality:** Neither is permanent. Always rely on **UUIDs** for mounting disks and configuring boot parameters.

---
@end
