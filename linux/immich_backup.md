Setting up a backup drive and backing up Immich before a wipe
 
Result
Old LUKS-encrypted SSD in a USB-C enclosure turned into an unencrypted ext4 backup drive (469G usable) mounted at `/mnt/backup`. Immich (NixOS module, `mediaLocation = /var/lib/immich`, 67G) backed up to it as plain files plus a Postgres dump, then verified byte for byte. Done before switching from NixOS to Ubuntu with a full wipe.

Layout on the drive:

- `/mnt/backup/photos/immich/` rsync copy of `/var/lib/immich`
- `/mnt/backup/photos/immich-db.sql.gz` Postgres dump
- `/mnt/backup/restic-repo/` for the rest of `/home` (not done yet)

Fix

1. Identify the drive. **Wrong device name wipes the wrong disk.** Plug it in, run it again, see what appears (mine was `sda`, `TRAN=usb`, 476.9G). Internal NVMe was `nvme0n1`.


```Bash
	> lsblk -o NAME,SIZE,MODEL,TRAN,FSTYPE,MOUNTPOINT
```

2. Wipe old signatures and make a fresh GPT partition. NixOS does not ship `parted` or `mkfs.ext4` on root's PATH, so use nix-shell.


```Bash
	> sudo umount /dev/sda1 2>/dev/null
	> sudo wipefs -a /dev/sda
	> nix-shell -p parted e2fsprogs
	> sudo parted /dev/sda -- mklabel gpt mkpart backup 1MiB 100%
```

3. `lsblk` still showed `crypto_LUKS` on `sda1`. The new partition started at the same offset as the old one, so the old LUKS header was still inside it. Wipe the partition itself too.


```Bash
	> sudo wipefs -a /dev/sda1
	> sudo blkid -p /dev/sda1
```

`blkid -p` reads the disk directly. If it shows only `PART_ENTRY_*` and no `TYPE=`, the signature is gone. `lsblk` was just showing a stale udev cache (fix with `sudo udevadm trigger /dev/sda1 && sudo udevadm settle` or `partprobe`).

4. Format as ext4. `-m 0` removes the 5% root reserve (pointless on a data drive). Use the full path because sudo resets PATH inside nix-shell.


```Bash
	> sudo "$(which mkfs.ext4)" -L backup -m 0 /dev/sda1
```

The "Creating journal" step looks frozen but is writing ~1 GB of zeros over USB. **Do not Ctrl+C**, I did once and had to redo it. Give it a couple of minutes. Skip the wait with `-E lazy_journal_init=1`.

5. Mount it, take ownership, make folders.


```Bash
	> sudo mkdir -p /mnt/backup
	> sudo mount /dev/sda1 /mnt/backup
	> sudo chown "$USER": /mnt/backup
	> mkdir -p /mnt/backup/photos /mnt/backup/restic-repo
	> df -h /mnt/backup
```

6. Sanity check writes and the label (stable path: `/dev/disk/by-label/backup`).


```Bash
	> touch /mnt/backup/photos/test && rm /mnt/backup/photos/test && echo writable
	> ls -l /dev/disk/by-label/backup
```

7. Check Immich size against free space, then stop Immich (leave Postgres running).


```Bash
	> sudo du -sh /var/lib/immich
	> sudo systemctl stop immich-server immich-machine-learning
```

8. Dump the database. The `collation version mismatch` warnings (glibc 2.42 vs 2.44) are harmless for a logical dump. The redirect runs as my user, so no permission issues.


```Bash
	> sudo -u postgres pg_dumpall --clean --if-exists | gzip > /mnt/backup/photos/immich-db.sql.gz
```

9. Verify the dump. Mine was 82M, 72 `COPY` statements, and had `CREATE DATABASE immich`.


```Bash
	> ls -lh /mnt/backup/photos/immich-db.sql.gz
	> gzip -t /mnt/backup/photos/immich-db.sql.gz && echo "gzip OK"
	> zcat /mnt/backup/photos/immich-db.sql.gz | grep -c '^COPY'
	> zcat /mnt/backup/photos/immich-db.sql.gz | grep -E '^CREATE EXTENSION'
```

10. Copy the library. 71 GB took ~33 min (~34 MB/s). Safe to re-run if interrupted. Trailing slashes matter.


```Bash
	> sudo rsync -aHAX --info=progress2 /var/lib/immich/ /mnt/backup/photos/immich/
```

11. Verify by checksum. **No output means the copy matches.** This reads everything on both sides, so it is slow and silent.


```Bash
	> sudo rsync -rcn --itemize-changes /var/lib/immich/ /mnt/backup/photos/immich/
	> sudo du -sh /var/lib/immich /mnt/backup/photos/immich
```

Lines starting with `>f` mean a file differs or is missing on the backup.

12. Start Immich again if still using it before the wipe (re-run step 10 afterwards to pick up new uploads).


```Bash
	> sudo systemctl start immich-server immich-machine-learning
```

13. Unmount cleanly when done.


```Bash
	> sudo umount /mnt/backup
```

Gotchas

- `parted` / `mkfs.ext4` not found on NixOS: use `nix-shell -p parted e2fsprogs`, and call tools via `"$(which ...)"` under sudo.
- `lsblk` can show stale FSTYPE after `wipefs`. Trust `blkid -p`.
- USB SSD speed was only ~34 MB/s. Check the link with `lsusb -t` (`5000M`/`10000M` = USB 3, `480M` = USB 2). Try a different cable or port.
- This drive is a **single copy** after the wipe. Keep a second copy of the photos elsewhere (another drive, Backblaze B2). Unpowered SSDs lose data over long periods, so plug it in and refresh every month or so.

Restore notes (Ubuntu, not done yet)

- Run Immich with the official Docker Compose setup, not the NixOS module.
- The dump has `LOCALE = 'en_IN'` in the `CREATE DATABASE` lines. The Postgres Docker image may not have that locale, so `sed` it to `en_US.utf8` or `C` before restoring.
- Check the `CREATE EXTENSION` lines (cube, earthdistance, and the vector one) and use a Postgres image that provides them.
- Match the Immich version to the backup, or restore first and upgrade after.
- Even if the DB restore fails, the originals in `immich/` (`library/` or `upload/`) are normal image files.

Still to back up before the wipe

- Rest of `/home` via restic to `/mnt/backup/restic-repo` (store the restic password in a password manager)
- `~/.ssh`, `~/.gnupg`, password manager DBs, 2FA recovery codes
- Git repos with unpushed or uncommitted work
- NixOS config pushed to a git remote


Resync

```bash
# 1. mount the drive (if not already) 
sudo mount /dev/disk/by-label/backup /mnt/backup 
# 2. stop Immich so files and DB match (optional, see below) 
sudo systemctl stop immich-server immich-machine-learning 
# 3. fresh DB dump (overwrites the old one) 
sudo -u postgres pg_dumpall --clean --if-exists | gzip > /mnt/backup/photos/immich-db.sql.gz 
# 4. sync the files 
sudo rsync -aHAX --info=progress2 /var/lib/immich/ /mnt/backup/photos/immich/ 
# 5. start Immich again and unmount 
sudo systemctl start immich-server immich-machine-learning 
sudo umount /mnt/backup
```