# HOW TO - ZFS replace disk

## Set variables

```bash
HEALTHY_DISK=/dev/sda
BROKEN_DISK=/dev/sdb
```

Make sure that the variables are correctly populated by executing this:

```bash
echo "HEALTHY_DISK: ${HEALTHY_DISK}"
echo "BROKEN_DISK: ${BROKEN_DISK}"
```

This should output

```
HEALTHY_DISK: /dev/sda
BROKEN_DISK: /dev/sdb
```

## Steps

1. Check status

    ```bash
    zpool status
    ```

2. List disks

    ```bash
    lsblk | grep -v zd
    ```

3. Rescan partitions (if the disk was replaced while the system was online):

    ```bash
    blockdev --rereadpt /dev/${BROKEN_DISK}
    ```

4. Copy partition layout from working disk to replaced disk:

    ```bash
    sgdisk --backup=/tmp/sda.layout ${HEALTHY_DISK}
    sgdisk --load-backup=/tmp/sda.layout ${BROKEN_DISK}
    ```

5. Resilver disk:

    - `FAILED_DISK_ID` can be an ID or just the partition number (when online replaced)
    - `NEW_DISK_ID` from the new disk or just the partition path (e.g. `/dev/sdb3`)

    Recommended is to use the IDs. They can be found with:

    ```bash
    ls -l /dev/disk/by-id/
    ```

    ```bash
    zpool replace rpool <FAILED_DISK_ID> ${BROKEN_DISK}3
    ```

6. Copy data from BIOS boot partition to new one (if replaced disk was the boot disk):

    ```bash
    dd if=${HEALTHY_DISK}1 of=${BROKEN_DISK}1 bs=512
    ```

7. Reinstall bootloader on replaced disk (if replaced disk was the boot disk):

    **GRUB**

    ```bash
    proxmox-boot-tool format ${BROKEN_DISK}2 --force
    proxmox-boot-tool init ${BROKEN_DISK}2 grub
    ```

    **systemd-boot**

    ```bash
    proxmox-boot-tool format ${BROKEN_DISK}2 --force
    proxmox-boot-tool init ${BROKEN_DISK}2
    ```

8. Update initramfs information (if replaced disk was the boot disk):

    ```bash
    update-initramfs -u
    proxmox-boot-tool status
    ```

9. List ZFS status again:

    ```bash
    zpool status
    ```
