# Linux Disk Info UUIDs and Labels

The `df` command is great for a quick health check on disk space, but by default, it’s pretty anonymous—it just shows you device paths like `/dev/sda1`.

If one need to get more level of detail, one want to pair `df`  with `lsblk` or `blkid`.

Here is how to get the information you’re looking for:

### 1. The "All-in-One" Method (`lsblk`)

This is usually the cleanest way to see size, mount point, label, and UUID in one view.

```bash
lsblk -o NAME,SIZE,USED,AVAIL,USE%,MOUNTPOINT,LABEL,UUID

```

* **Why use this:** it gives you the human-readable sizes (like `df -h`) but attaches the metadata `df` misses.

---

### 2. Using `findmnt` (The Modern Alternative)

Most modern Linux distributions include `findmnt`, which is arguably more powerful than `df`.

```bash
findmnt -lo source,target,fstype,label,uuid,size,used,avail

```

* **Why use this:** It presents the data in a clean table and specifically targets currently mounted filesystems.

---

### 3. The Classic `blkid` (For IDs only)

If you already ran `df -h` and just need to know which UUID belongs to which partition:

```bash
sudo blkid

```

* **Note:** This requires root privileges. It will list every partition, its Label, and its UUID, but it won't show you how much space is left.

---

### Summary of Differences

| Command | Shows Size/Usage? | Shows UUID? | Shows Labels? | Best For... |
| --- | --- | --- | --- | --- |
| `df -h` | **Yes** | No | No | Quick capacity check. |
| `lsblk -o` | **Yes** | **Yes** | **Yes** | Comprehensive mapping. |
| `blkid` | No | **Yes** | **Yes** | Finding IDs for `/etc/fstab`. |
| `findmnt` | **Yes** | **Yes** | **Yes** | Detailed mount analysis. |

---
@end