<p align="center">
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/b46c5224-681e-45ec-9d07-feab2c8acfb6" />
lappy!
</p>

# Configuration
>[!NOTE]
> Due to the AOSC installer not supporting [Full Disk Encryption](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system), the installation process has to be done manually!

## Partitions
```
sda             8:0    0 119.2G  0 disk
├─sda1          8:1    0   512M  0 part  /boot/efi
└─sda2          8:2    0 118.7G  0 part
  └─cryptroot 251:0    0 118.7G  0 crypt /home
                                         /boot
                                         /.snapshots
                                         /
```
## Things related to LUKS
```bash
# /etc/dracut.conf.d/10-luks.conf
add_dracutmodules+=" crypt dm btrfs "
filesystems+=" btrfs "
```
```bash
# /etc/default/grub
# ...
# Linux boot parameters for all boot modes.
GRUB_CMDLINE_LINUX="cryptdevice=UUID=<sda2>:cryptroot root=/dev/mapper/cryptroot rootflags=subvol=@root"
GRUB_ENABLE_CRYPTODISK=y
```
```bash
# /etc/crypttab
cryptroot	UUID=<sda2>	none	luks
```
### Needed for LUKS2 (may update in the future

```bash
$ sudo oma topics --opt-in grub-2.14
```

## Secureboot
`sbctl` isn't packaged. Build it.
```bash
$ git clone https://github.com/Foxboron/sbctl && cd sbctl
$ sudo oma install go asciidoc devel-base
$ make install
```

### GRUB without a shim
GRUB has to be compact in order to run.
```bash
$ sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi \
    --modules="all_video boot btrfs cat echo ext2 fat gfxterm gfxmenu gfxterm_background gzio halt help linux ls part_gpt play cpuid tpm cryptodisk luks" \
    --bootloader-id=aosc \
    --disable-shim-lock
```

### No such cryptodisk
By default, Argon2i(d)(?) has a 1GB memory, which GRUB *cannot* handle

Set the memory size to 65536KB (or 64MB).
> [!CAUTION]
> This reduces security! Use a seperated header file if possible.

```bash
$ sudo cryptsetup luksConvertKey -h /dev/sda2 --pbkdf argon2i --pbkdf-memory 65536
```

<sub>thanks bai!</sub>

# Ricing, i guess

## Remove DE(s)
```bash
$ sudo apt remove kde-base
$ sudo oma install util-base
```

## Build a fork of Niri!
```bash
$ sudo oma install devel-base llvm clang gcc rust
$ git clone --branch feat/blur --single-branch https://github.com/visualglitch91/niri.git
$ cd niri
$ cargo build --release --vv
$ mkdir -p ~/.local/bin && cp target/release/niri
```
```toml
# /usr/share/wayland-sessions/niri.desktop
[Desktop Entry]
Name=niri
Comment=niri
Exec=/home/gahxke/.local/bin/niri
TryExec=/home/gahxke/.local/bin/niri
Type=Application
```

## Other goodies...
```bash
$ oma install waybar swaybg swaylock mako-notification-daemon waybar wofi jetbrains-mono rsms-inter-fonts symbols-font-nerd aria2 fastfetch
$ aria2c 'https://github.com/raphamorim/rio/releases/download/v0.2.37/rioterm-0.2.37-1.amd64_wayland.deb'
$ sudo dpkg -i rioterm-0.2.37-1.amd64_wayland.deb
$ curl -o rio.terminfo https://raw.githubusercontent.com/raphamorim/rio/main/misc/rio.terminfo
$ sudo tic -xe xterm-rio,rio rio.terminfo
$ rm rio.terminfo
```
