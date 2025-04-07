# HOW TO - Enable swap with ZFS for memory exhaustion

To avoid a system crash when RAM fills up unexpectedly, you can enable swap in combination with ZFS. Swap acts as an overflow for memory, storing less frequently used data when RAM is completely full. In this example, a `16G` swap volume is created, but you can adjust the size as needed.

**Important:** Swap is only used when the RAM is completely full. It is recommended to monitor swap usage and swap I/O to act accordingly if swap is being used frequently, as excessive swap usage can degrade system performance.

## Steps to enable swap on ZFS

### Create a ZFS Volume for swap

Execute the following command to create a ZFS volume:

```bash
zfs create -V 16G -o compression=off -o dedup=off -o sync=always rpool/swapvol
```

### Verify ZFS volume properties

Check if compression and deduplication are disabled:

```bash
zfs get compression,dedup rpool/swapvol
```

Expected output:

```
NAME           PROPERTY     VALUE           SOURCE
rpool/swapvol  compression  off             local
rpool/swapvol  dedup        off             local
```

### Format the ZFS volume as swap

Format the ZFS volume to make it usable as swap space:

```bash
mkswap /dev/zvol/rpool/swapvol
```

Expected output:

```
Setting up swapspace version 1, size = 16 GiB (17179865088 bytes)
no label, UUID=dc95f19e-f084-4777-a31f-c5d39a2b07b8
```

### Adjust swappiness

Set the system to use swap only when memory is exhausted:

- **For runtime (temporary):**

  ```bash
  sysctl vm.swappiness=1
  ```
- **For permanent settings:**

  ```bash
  echo "vm.swappiness=1" > /etc/sysctl.d/99-swappiness.conf
  sysctl -p /etc/sysctl.d/99-swappiness.conf
  ```

### Enable swap

Activate the swap volume:

```bash
swapon /dev/zvol/rpool/swapvol
```

### Verify swap activation

Check if the swap is enabled:

```bash
swapon --show
```

Expected output:

```
NAME       TYPE      SIZE USED PRIO
/dev/zd304 partition  16G   0B   -2
```

### Enable swap on boot

To ensure the swap is mounted on every boot, add it to `/etc/fstab`:

```bash
echo "/dev/zvol/rpool/swapvol none swap defaults 0 0" >> /etc/fstab
```

### Verify `/etc/fstab`

Check the contents of `/etc/fstab` to confirm the entry:

```bash
cat /etc/fstab
```

Expected output:

```
# <file system> <mount point> <type> <options> <dump> <pass>
proc /proc proc defaults 0 0
/dev/zvol/rpool/swapvol none swap defaults 0 0
```

Now you have an emergency swap volume configured using ZFS. Monitor its usage and adjust as needed.
