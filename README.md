## Raspberry Pi (4B, 4GB) Forensic Duplicator
<img src = "thumbnail.png">
### Author
ezloomdev

### Disclaimer

- This Raspberry Pi forensic duplicator is intended for educational and home-lab purposes only.
- It is not certified for professional digital forensics and should not be used as the sole method to handle real evidence.
- Always follow proper chain-of-custody and professional forensic procedures when handling critical or legal data.

### Purpose
A forensic duplicator's job is to protect the integrity of the original evidence drive from any accidental writes while allowing you to:

- Examine the drive
- Create a forensic image
- Verify the image integrity

This Raspberry Pi acts as the middleman:

- Evidence drive → read-only → image drive / analysis

- This setup is built on Raspberry Pi OS (64-bit) running on a Raspberry Pi 4B with 4GB RAM, optimized for lightweight forensic tasks, stability, and fast USB handling.

### Installation & Setup

- Make sure your Raspberry Pi OS is up to date:
```bash
sudo apt update && sudo apt upgrade -y
```
- Install required imaging tools:
```bash
sudo apt install ddrescue dcfldd -y
```
- Ensure `blockdev` is available (part of util-linux):
  - git clone (https://github.com/msuhanov/Linux-write-blocker/tree/master/userspace/udev) and move to /etc/udev/rules.d and the stuff in the 'tools' folder (https://github.com/msuhanov/Linux-write-blocker/tree/master/userspace/tools) to /usr/sbin, and then reboot.
```bash
which blockdev
```
- Copy the udev script and rule files to the correct locations:

- udev script `/usr/local/bin/usb-readonly.sh`

```bash
#!/bin/bash
DEVNAME="/dev/$1"
sudo blockdev --setro $DEVNAME
logger "USB device $DEVNAME set to read-only."
```
```bash
sudo chmod +x /usr/local/bin/usb-readonly.sh
```

- udev rule `/etc/udev/rules.d/99-usb-readonly.rules`
```bash
ACTION=="add", SUBSYSTEM=="block", ENV{ID_BUS}=="usb", RUN+="/usr/local/bin/usb-readonly.sh %k"
```
- System log shows
```bash
USB device /dev/sdb1 set to read-only
```

- `/usr/local/bin/usb-readonly.sh`
- `/etc/udev/rules.d/99-usb-readonly.rules`

- Reload udev rules:
```bash
sudo udevadm control --reload
sudo udevadm trigger
```

## What the Pi Does

### Detect the evidence drive
- Automatically detects when a USB drive is connected using udev
- Example: lsblk will show /dev/sda or /dev/sdb (may vary)
- Any plugged-in USB is immediately recognized as the evidence drive

### Block writes automatically
- udev-triggered script sets the device read-only at the block level
```bash
sudo blockdev --setro /dev/sda
```
- Notification/log confirms the action
```bash
logger "USB device /dev/sda set to read-only."
```
- Optionally, mount read-only for safe browsing
```bash
sudo mount -o ro /dev/sda1 /mnt/evidence
```
- Write attempts fail
```bash
touch testfile.txt
# Output: touch: cannot touch 'testfile.txt': Read-only file system
```

### Forensic imaging tools

> **Important Notes**
> 
> - Always run imaging commands with `sudo` to access devices correctly.
> - Make sure the Raspberry Pi is powered reliably; losing power during imaging can corrupt your copy.
> - After installing or modifying udev rules, rebooting the Pi may be required to ensure rules take effect:
> ```bash
> sudo reboot
> ```

- Two terminal imaging tools are installed
- ddrescue → resilient imaging for error-prone drives
- dcfldd → forensic imaging with hash verification
- Example usage:
```bash
sudo ddrescue -f -n /dev/sda /dev/sdb /home/pi/rescue.log
sudo dcfldd if=/dev/sda of=/dev/sdb hash=sha256 hashlog=/home/pi/hash.log
```

### Verify image integrity

- Compare hashes to confirm the copy matches the original
```bash
sudo sha256sum /dev/sda1
sudo sha256sum /dev/sdb1
```
- `blockdev --getro /dev/sda` returns 1 if read-only

### Browse evidence safely

- Mount read-only to inspect files
```bash
sudo mkdir -p /mnt/evidence
sudo mount -o ro /dev/sda1 /mnt/evidence
ls /mnt/evidence
```
- Never mount the original drive read-write

### Testing the write-blocker tool

- Plug in USB (/dev/sdb1)
- Script sets it read-only automatically
- Mount and inspect
```bash
sudo mkdir -p /mnt/testusb
sudo mount -o ro /dev/sdb1 /mnt/testusb
cd /mnt/testusb
ls
```
- Attempt writing fails
```bash
touch testfile.txt
# Output: touch: cannot touch 'testfile.txt': Read-only file system
```

- Device is now ready for imaging and analysis

### Photos
<center>
<p>
  <img src="images/img1.png" width="300" />
  <img src="images/img2.png" width="300" />
</p>
<p>
  <img src="images/img3.png" width="300" />
  <img src="images/img4.png" width="300" />
</p>
</center>

### Reference

- Idea
  - https://sechaq.medium.com/building-my-own-write-blocker-cbf687eb0fd0
- Hardware
  - https://www.amazon.com/dp/B07XBVF1C9?ref=ppx_yo2ov_dt_b_fed_asin_title
  - https://www.amazon.com/CanaKit-Raspberry-4GB-Basic-Kit/dp/B07TXKY4Z9?crid=2F31ZF0DQQR3H&dib=eyJ2IjoiMSJ9.NbdpkzE9de4BnV1ityMKr9DbEGkml4gNmPur2l4bvUeterViSIylhvpNrT0tJWntL4_s7ujioONXS4ltjD4FX65XcmERmyOmISZBi_7tABI2IjRR86kS976yofFsDp1Pn0ehSlTRH0TbV4N9Qn64vn87RnAAZWJTNxJ3VU0qkC6SWWPwrcXQOpMvV62SOgHsmFbjHgbn6K8sFETNmZaZyew_-22oRBlC31f6-eV1-C3QDUZhSjdzCCLImPX7reBwPnF8KRD54hf043Qzx3IOyySPGx9YNfwSvm8c0EffF9k.qwplLd1LQfg-D4aSfEywpsGhE2kquR53HjKHHoNoduI&dib_tag=se&keywords=raspberry%2Bpi%2B4%2B4b%2B4gb&qid=1761283781&s=electronics&sprefix=raspberry%2Bpi%2B4%2B4b%2B4gb%2Celectronics%2C59&sr=1-1&th=1
