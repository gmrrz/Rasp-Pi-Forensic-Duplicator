# Raspberry Pi (4B, 4GB) Forensic Write-Blocker

### Author
ezloomdev

---

## Purpose

A forensic write-blocker's job is to protect the integrity of the **original evidence drive** from any accidental writes while allowing you to:

- Examine the drive  
- Create a forensic image  
- Verify the image integrity  

This Raspberry Pi acts as the **middleman**:

evidence drive → read-only → image drive / analysis

This setup is built on a **DietPi v9.18.1 (aarch64)** system running on a **Raspberry Pi 4B with 4GB RAM**, optimized for lightweight forensic tasks, stability, and fast USB handling.

---

## What the Pi Does

### 1. Detect the evidence drive
- Automatically detects when a USB drive is connected using **udev**.  
- Example: `lsblk` will show `/dev/sda` or `/dev/sdb`.  
- Any plugged-in USB is immediately recognized as the evidence drive.

### 2. Block writes automatically
- udev-triggered script sets the device **read-only at the block level**:

```bash
sudo blockdev --setro /dev/sda
```

- Notification/log confirms the action:

```bash
logger "USB device /dev/sda set to read-only."
```

- Optionally, mount read-only for safe browsing:

```bash
sudo mount -o ro /dev/sda1 /mnt/evidence
```

- Write attempts fail:

```bash
touch: cannot touch 'testfile.txt': Read-only file system
```

---

### 3. Forensic imaging tools

- Two terminal imaging tools are installed:
    - ddrescue → resilient imaging for error-prone drives
    - dcfldd → forensic imaging with hash verification

- Example usage:

```bash
sudo ddrescue -f -n /dev/sda /dev/sdb /home/pi/rescue.log
sudo dcfldd if=/dev/sda of=/dev/sdb hash=sha256 hashlog=/home/pi/hash.log
```

### 4. Verify image integrity

- Compare hashes to confirm the copy matches the original:

```bash
sudo sha256sum /dev/sda1
sudo sha256sum /dev/sdb1
```

- `blockdev --getro /dev/sda` returns `1` if read-only.

### 5. Browse evidence safely

- Mount read-only to inspect files:

```bash
sudo mkdir -p /mnt/evidence
sudo mount -o ro /dev/sda1 /mnt/evidence
ls /mnt/evidence
```

- Never mount the original drive read-write.

### 6. Automatic logging and notification

- udev script `/usr/local/bin/usb-readonly.sh`:

```bash
#!/bin/bash
DEVNAME="/dev/$1"
sudo blockdev --setro $DEVNAME
logger "USB device $DEVNAME set to read-only."
```

- udev rule `/etc/udev/rules.d/99-usb-readonly.rules`:

```bash
ACTION=="add", SUBSYSTEM=="block", ENV{ID_BUS}=="usb", RUN+="/usr/local/bin/usb-readonly.sh %k"
```

- System log shows:

```bash
USB device /dev/sdb1 set to read-only.
```

### 7. Testing the write-blocker

- Plug in USB (/dev/sdb1)  

- Script sets it read-only automatically  

- Mount and inspect:

```bash
sudo mkdir -p /mnt/testusb
sudo mount -o ro /dev/sdb1 /mnt/testusb
cd /mnt/testusb
ls
```

- Attempt writing fails:

```bash
touch testfile.txt
# Output: touch: cannot touch 'testfile.txt': Read-only file system
```

- Device is now ready for imaging and analysis.

### 8. Optional: Terminal Dashboard

- Pi auto-launches **btop** on boot for monitoring system resources (CPU, RAM, network, disk I/O).  
- SSH is still password-protected for remote access, while the local terminal displays btop automatically.

### 9. Important Notes

- Never mount the evidence drive read-write.  
- Always verify hash values after imaging.  
- Logs can be monitored live:

```bash
journalctl -f
```

- DietPi ensures minimal overhead, low power usage, and maximum USB throughput.  

- This setup transforms a **4GB Raspberry Pi 4B** into a dedicated, plug-and-play forensic write-blocker with **automatic read-only enforcement**, hashing, and imaging readiness.

---

### Reference

- https://sechaq.medium.com/building-my-own-write-blocker-cbf687eb0fd0