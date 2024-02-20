# bazzite-win3

### windows 10 dual boot, bazzite os installation needs to create these partition by manually to install the os.

```
/boot/efi           efi partition 300MiB
/boot               ext4 partition 1GiB
/                   btrfs partition all space
```
### Win3 Selinux set up to permissive:

```
$ sudo vim /etc/selinux/config
```
```
SELINUX=permissive
```
```
$ sudo systemctl reboot
```

### win3 rotation:
```
$ mkdir -p ~/.config/environment.d/
```

```
$ vim ~/.config/environment.d/1000-win3.conf 
```
```
export STEAM_DISPLAY_REFRESH_LIMITS=30,60
export GAMESCOPECMD="/usr/bin/gamescope \
  -e \
  --filter fsr \
  --fsr-sharpness 5 \
  --generate-drm-mode fixed \
  --xwayland-count 2 \
  -O *,DSI-1 \
  --default-touch-mode 4 \
  --hide-cursor-delay $CURSOR_DELAY \
  --fade-out-duration 200 \
  --cursor-scale-height 720 \
  --force-orientation right "
```
### win3 windows ntfs disk for sharing to bazzite os:
#### `/dev/nvme0n1p5` is D drive steam libary, insert the last line to `/etc/fstab`

Create `~/local` directory for mounting ntfs D drive;
```
$ mkdir ~/local
```

Create `~/share` directory for using the new steam libary;
```
$ mkdir ~/share
```

Insert the last line to `/etc/fstab`;
```
$ sudo vim /etc/fstab
```
```
/dev/nvme0n1p5 /var/home/deck/local lowntfs-3g windows_names,uid=1000,gid=1000,rw,user,exec,umask=000 0 0
```

```
#
# /etc/fstab
# Created by anaconda on Fri Feb  9 20:21:02 2024
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=b8abf785-83dc-411e-8840-4235ae941518 /                       btrfs   subvol=root,noatime,lazytime,commit=120,discard=async,compress-force=zstd:1,space_cache=v2 0 0
UUID=ad111963-95b6-456a-9ce0-a86f30277a9c /boot                   ext4    defaults        1 2
UUID=8904-5F91          /boot/efi               vfat    umask=0077,shortname=winnt 0 2
/dev/nvme0n1p5 /var/home/deck/local lowntfs-3g windows_names,uid=1000,gid=1000,rw,user,exec,umask=000 0 0
```

```
$ sudo mount -av
```

Link from `/home/deck/local/Steam/steamapps/common` to `/home/deck/share/steamapps/`

```
$ ln -s /home/deck/local/Steam/steamapps/common /home/deck/share/steamapps/
```

### rpm installation for hibernate instead of sleep( needs reboot twice and it will create 32GiB swapfile for hibernate automatically ) :
```
$ sudo rpm-ostree install ostree-s4swap-20231112-1.fc39.x86_64.rpm
$ sudo systemctl reboot
```

### rpm installation for intel cpu cpu_max_perf:
```
$ sudo rpm-ostree install intel_set_prefs-20231112-1.fc39.x86_64.rpm
$ sudo systemctl reboot
```

```
$ intel_set_prefs -read-all
```

```
$ intel_set_prefs -write-sensor cpu_max_perf 70
```

```
{"cpu_min_perf":"8","cpu_max_perf":"70","cpu_turbo":"true","gpu_min_freq":"100","gpu_max_freq":"1300","gpu_min_limit":"350","gpu_max_limit":"1300","gpu_boost_freq":"1300","gpu_cur_freq":"350","cpu_governor":"powersave","intel_rapl_short":"40","intel_rapl_long":"162"}
```

### Use systemd daemon to lock cpu_max_perf to 70%:
```
$ sudo vim /etc/systemd/system/set_prefs.service
```
```
[Unit]
Description=Set Performance Script
Requires=local-fs.target
After=local-fs.target


[Service]
Type=oneshot
ExecStart=/usr/bin/intel_set_prefs -write-sensor cpu_max_perf 70 || true
RemainAfterExit=no

[Install]
WantedBy=default.target
```

```
sudo systemctl status systemctl daemon-reload
sudo systemctl enable set_prefs.service
sudo systemctl start set_prefs.service
```

### Use `decky terminal` plugin to adjust `cpu max perf`
Change Default shell to `/bin/bash`

```
echo "alias i='/usr/bin/intel_set_prefs -write-sensor cpu_max_perf'" >> ~/.bashrc
source ~/.bashrc
```

```
$ i 58
{"cpu_max_perf":"58"}
```

### Use steam-patch-intel (latest client doesn't work), steam support `3~10` tdp to change cpu_max_perf `30%~100%` (need 2 rpms packages intel_set_prefs & steam-patch-intel):
```
$ sudo rpm-ostree install intel_set_prefs-20231112-1.fc39.x86_64.rpm steam-patch-intel-20240216-1.fc39.x86_64.rpm
$ sudo systemctl reboot
```

```
$ sudo systemctl enable steam-patch-intel@deck
$ sudo systemctl start steam-patch-intel@deck
```
