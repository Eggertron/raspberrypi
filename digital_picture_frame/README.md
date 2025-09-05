# Digital Picture Frame

Raspberry Pi with X org server running feh. The USB drives are automatically mounted and played on screen.

## Installation

Update Repo and OS. Install X org server

```bash
sudo apt-get update -y && sudo apt-get upgrade -y
sudo apt install -y xserver-xorg xinit
```

Install feh

```bash
sudo apt-get install -y feh
```

## Configuration

Add USB Mount and Run script

Create the file `/usr/local/bin/usb_automount.sh` with

```bash
#!/bin/bash

# A script to automatically mount and unmount USB storage devices.
# The `udev` rule will pass the device name (e.g., "sdb1") and action ("add" or "remove")
# as arguments.

ACTION="$1"
DEVICE="/dev/$2"
DEVICE_NAME="$2"

# Define the base mount point directory
MOUNT_POINT_BASE="/media/usb"

# Define the mount point for this specific device
MOUNT_POINT="${MOUNT_POINT_BASE}/${DEVICE_NAME}"

# Log actions for debugging
log_action() {
    logger -t usb_automount "ACTION=$ACTION, DEVICE=$DEVICE, MOUNT_POINT=$MOUNT_POINT"
}

if [[ "$ACTION" == "add" ]]; then
    log_action
    # Create mount point and mount the device
    if [ ! -d "$MOUNT_POINT" ]; then
        mkdir -p "$MOUNT_POINT"
    fi
    /usr/bin/mount -o nosuid,nodev,nofail "$DEVICE" "$MOUNT_POINT" 2>/dev/null
    exit 0
elif [[ "$ACTION" == "remove" ]]; then
    log_action
    # Unmount the device and remove the directory
    /usr/bin/umount -l "$MOUNT_POINT" 2>/dev/null
    if [ -d "$MOUNT_POINT" ]; then
        rmdir "$MOUNT_POINT"
    fi
    exit 0
fi
```

make it executable

```bash
chmod +x /usr/local/bin/usb_automount.sh
```

Create the file `/etc/udev/rules.d/99-usb-automount.rules` with

```ini
# Rule to automount USB storage devices
ACTION=="add", KERNEL=="sd[a-z][0-9]", SUBSYSTEM=="block", ENV{ID_BUS}=="usb", RUN+="/sbin/blockdev --setro /dev/%k", RUN+="/usr/local/bin/usb_automount.sh add %k"

# Rule to unmount USB storage devices on removal
ACTION=="remove", KERNEL=="sd[a-z][0-9]", SUBSYSTEM=="block", ENV{ID_BUS}=="usb", RUN+="/usr/local/bin/usb_automount.sh remove %k"
```

Reload the Rule

```sh
# Reload udev rules
sudo udevadm control --reload-rules

# Trigger the rules to apply to existing devices
sudo udevadm trigger
```
