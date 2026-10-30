# RESTIC

Backing up home folders with restic
 
Result
Documents, notes, system-config, career, learn and books (about 256 MB total) backed up into an encrypted, deduplicated restic repo on the backup drive at `/mnt/backup/restic-repo`. 1 snapshot (`c4015a48`, 1768 files, 248 MiB added, 188 MiB stored). `restic check --read-data` reported no errors. Pictures were imported into Immich and covered by the Immich resync (see the Immich backup note).

Needs the backup drive from the note "Setting up a backup drive and backing up Immich before a wipe".

Fix

1. Check sizes against free space.


```Bash
	> du -sh ~/Documents ~/notes ~/system-config ~/Pictures ~/career ~/learn ~/books
```

2. Get restic (not on root's PATH on NixOS) and point it at the repo. The drive must be mounted at `/mnt/backup`.


```Bash
	> nix-shell -p restic
	> export RESTIC_REPOSITORY=/mnt/backup/restic-repo
```

3. Create the repo. It asks for a password. **Put it in the password manager first. If it is lost, the repo is unrecoverable.**


```Bash
	> restic init
```

4. Back up the folders. Add `--exclude node_modules --exclude .venv --exclude target` if any of them hold build junk.


```Bash
	> restic backup ~/Documents ~/notes ~/system-config ~/career ~/learn ~/books
```

5. Confirm the snapshot exists.


```Bash
	> restic snapshots
```

6. Verify every byte in the repo. Slow on big repos, instant on this one.


```Bash
	> restic check --read-data
```

Expected last line: `no errors were found`.

7. Test a real restore (a backup you have never restored from is only a hope). Restored files keep their full original path under the target.


```Bash
	> restic restore latest --target /tmp/restore-test --include "$HOME/notes"
	> ls /tmp/restore-test/home/*/notes | head
	> rm -rf /tmp/restore-test
```

Syncing later

Re-run the same command. It only stores what changed and adds a new snapshot.


```Bash
	> sudo mount /dev/disk/by-label/backup /mnt/backup
	> nix-shell -p restic
	> export RESTIC_REPOSITORY=/mnt/backup/restic-repo
	> restic backup ~/Documents ~/notes ~/system-config ~/career ~/learn ~/books
	> restic snapshots
	> sudo umount /mnt/backup
```

Restoring on a new machine (Ubuntu)

1. Install restic and mount the drive.


```Bash
	> sudo apt install restic
	> sudo mount /dev/disk/by-label/backup /mnt/backup
```

2. Restore into a staging folder first, then copy what you want. Do not restore straight over `/home` until you have looked.


```Bash
	> export RESTIC_REPOSITORY=/mnt/backup/restic-repo
	> restic snapshots
	> restic restore latest --target /tmp/restore

	> restic -r /mnt/backup/restic-repo snapshots
	> restic -r /mnt/backup/restic-repo restore latest --target /tmp/restore
	> ls /tmp/restore/home/*/
```

3. Copy the folders into place.


```Bash
	> rsync -a /tmp/restore/home/*/ ~/
```

Gotchas

- Password is the only key. Store it in the password manager, not just in my head.
- Snapshots are tied to the absolute path (`/home/ethycs/...`), so restores land under `home/<user>/` in the target.
- The drive is a single SSD. Keep a second copy somewhere else. restic can also back up to Backblaze B2 or similar.
- Unpowered SSDs lose data over long periods. Plug it in and refresh every month or so.
- Old snapshots pile up. Later, trim with `restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune`.

Before the wipe

- Run `restic backup ...` once more so the repo has the final state.
- Push `system-config` (NixOS config) to a git remote, so it does not depend on this drive.
- `git status` and `git log @{u}..` in repos under `learn` and `career` for unpushed work.
- Still to check: `~/.ssh`, `~/.gnupg`, password manager DBs, 2FA recovery codes, `~/.config` and `~/.local/share` if wanted.