
# Recovering from force shutdown while nixos-rebuild switch was running
## Result

BRUB got corrupted. GRUB looked like terminal. I could `exit` and fallback to systemd but it did not allow choice of generation. Instead I had to decrypt my drive and system would fail to boot in NixOS boot stage 2.

## Fix

1. Boot into NixOS live USB
2. Decrypt my drive
```Bash
	> sudo mount -o subvol=@ /dev/mapper/cryptroot /mnt
```
3. Mount all Partitions, BTRFS for me.
```Bash
	> sudo mount -o subvol=@ /dev/mapper/cryptroot /mnt
	> sudo mount -o subvol=@home /dev/mapper/cryptroot /mnt/home
	> sudo mount /dev/nvme0n1p1
```
4. Enter mounted NixOS 
```Bash
	> sudo nixos-enter
```
5. Check available generations
```Bash
	> nix-env --list-generations -p /nix/var/nix/profiles/system
```
6. Try various rollback method (Did not work for me)
```Bash
	> /nix/var/nix/profiles/system-XXX-link/bin/switch-to-configuration switch
	> /nix/var/nix/profiles/system-XXX-link/bin/switch-to-configuration boot
	> nixos-rebuild switch --rollback
```
7. Use stable generation to reinstall bootloader
```Bash
	> NIXOS_INSTALL_BOOTLOADER=1 /nix/var/nix/profiles/system-68-link/bin/switch-to-configuration boot
```
8. Exit, Unmount and Reboot
```Bash
	> exit
	> sudo umount -R /mnt
	> sudo reboot
```

This should fix the issue.