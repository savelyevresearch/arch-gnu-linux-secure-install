- # Basic information  
	- This guide describes security-enhanced Arch GNU/Linux installation, which can be useful for Cybersecurity Engineers, Cybersecurity Analysts, Pentesters.  
- # Basic requirements  
	- reEFInd bootloader with minimalistic theme
	- Hardened Linux Kernel  
	- AppArmor  
	- BTRFS  
	- AES encryption of Root Filesystem  
	- ZRAM  
	- Wayland  
	- XWayland  
	- Sway WM  
	- Waybar  
	- Wofi  
	- Alacritty  
	- PipeWire  
- # Installation  
	- ## Step 1 $\implies$ Download Arch GNU/Linux image  
		- The official site of Arch GNU/Linux: https://archlinux.org  
	- ## Step 2 $\implies$ Check integrity and digital signature of image  
		- For this, use sha256sum command and GPG  
	- ## Step 3 $\implies$ Burn distribution image to USB drive  
		- For this, use dd command:  
			-  
			  ```zsh
				dd if=/path/to/image/file of=/path/to/usb/drive status=progress
			  ```
	- ## Step 4 $\implies$ Boot environment from USB drive  
		- Just select your USB drive as a booting device and launch your system  
	- ## Step 5 $\implies$ Connect to the Internet  
		- Use iwctl for connecting to the Internet:  
			-  
			  ```zsh
			  	iwctl
			  ```
			-  
			  ```prompt
			  	# iwctl list
			  ```
			-  
			  ```prompt
			  	# iwctl station <device> scan
			  ```
			-  
			  ```prompt
			  	# iwctl station <device> connect <SSID>
			  ```
	- ## Step 6 $\implies$ Disk partitioning  
		- Make the EFI system partition or use the existing one with `300MB`:  
			-  
			  ```zsh
			  	cfdisk /dev/{nvmeXXX, sXX}
			  ```
		- Make the boot partition with`800MB`:  
			-  
			  ```zsh
			  	fdisk /dev/{nvmeXXX, sXX}
			  ```
		- Make the root partition with all the rest space:  
			-  
			  ```zsh
			  	cfdisk /dev/{nvmeXXX, sdX}
			  ```
	- ## Step 7 $\implies$ Create and mount the filesystems of the partitions  
		- Make the VFAT filesystem for the EFI system partition (or use the existing one):  
			-  
			  ```zsh
			  	mkfs.vfat /dev/{nvmeXXXXX, sdXX}
			  ```
		- Create the LUKS partition:  
			-  
			  ```zsh
			  	cryptsetup --type luks2 --cipher aes-xts-plain64 --hash sha512 --key-size 512 --iter-time 2000 --pbkdf argon2id --use-random --verify-passphrase luksFormat /dev/{nvmeXXXXX, sdXX}
			  ```
		- Then, enter the passphrase  
		- Add key file to the LUKS partition:  
			-  
			  ```zsh
			  	cryptsetup luksAddKey /dev/{nvmeXXXXX, sdXX} /path/to/keyfile
			  ```
		- Then, check the number of the key slots by viewing the dump:  
			-  
			  ```zsh
			  	cryptsetup luksDump /dev/{nvmeXXXXX, sdXX}
			  ```
		- Open the LUKS partition using passphrase or key file:  
			-  
			  ```zsh
			  	cryptsetup open /dev/{nvmeXXXXX, sdXX} <LUKS-devicename>
			  ```
		- Make the header backup file for LUKS partition:  
			-  
			  ```zsh
			  	cryptsetup luksHeaderBackup /dev/{nvmeXXXXX, sdXX} --header-backup-file /path/to/backup/file
			  ```
		- Make the BTRFS filesystem for LUKS partition:  
			-  
			  ```zsh
			  	mkfs.btrfs -L root /dev/mapper/<LUKS-devicename>
			  ```
		- Mount the LUKS partition:  
			-  
			  ```zsh
			  	mount /dev/mapper/<LUKS-devicename> /mnt
			  ```
		- Make BTRFS subvolumes:  
			-  
			  ```zsh
			  	btrfs subvolume create /mnt/@
			  ```
			-  
			  ```zsh
			  	btrfs subvolume create /mnt/@home
			  ```
			-  
			  ```zsh
			  	btrfs subvolume create /mnt/@snapshots
			  ```
		- Create the necessary directories:  
			-  
			  ```zsh
			  	mkdir /mnt/{home,.snapshots,boot}
			  ```
		- Unmount the root filesystem:  
			-  
			  ```zsh
			  	umount /mnt
			  ```
		- Mount the BTRFS subvolumes:  
			-  
			  ```zsh
			  	mount -t btrfs -o noatime,relatime,compress=zstd,commit=120,space_cache=v2,ssd,discard=async,autodefrag,subvol=@ /dev/mapper/<LUKS-devicename> /mnt
			  ```
			-  
			  ```zsh
			  	mount -t btrfs -o noatime,relatime,compress=zstd,commit=120,space_cache=v2,ssd,discard=async,autodefrag,subvol=@home /dev/mapper/<LUKS-devicename> /mnt/home
			  ```
			-  
			  ```zsh
			  	mount -t btrfs -o noatime,relatime,compress=zstd,commit=120,space_cache=v2,ssd,discard=async,autodefrag,subvol=@snapshots /dev/mapper/<LUKS-devicename> /mnt/.snapshots
			  ```
		- Mount the EFI system partition:  
			-  
			  ```zsh
			  	mount /dev/{nvmeXXXXX, sdXX} /mnt/boot
			  ```
	- ## Step 8 $\implies$ Install the system packages  
		-  
		  ```zsh
		  	pacstrap /mnt linux-hardened linux-headers-hardened linux-firmware base \
		  			 	  base-devel intel-ucode btrfs-progs git go python iwd \
		  			      wpa_supplicant wireless_tools iw dhcpcd networkmanager \
		  			      mesa lib32-mesa xf86-video-intel vulkan-intel docker qemu \
		  			      refind wl-clipboard zsh npm xdg-user-dirs snapper sway waybar \
		  			      wofi wayland xorg-xwayland alacritty xdg-desktop-portal-wlr \
		  			      unrar tar unzip zip apparmor pipewire ttf-roboto \
		  			      ttf-roboto-mono ttf-dejavu ttf-liberation ttf-fira-code \
		  			      ttf-hanazono ttf-fira-mono ttf-opensans ttf-hack noto-fonts \
		  			      noto-fonts-emoji ttf-font-awesome ttf-droid \ 
		  			      adobe-source-code-pro-fonts man bluez bluez-utils yarn code \
		  			      swayidle swaylock e2fsprogs dosfstools wf-recorder reflector \
		  			      neovim sudo neofetch htop strace cryptsetup pactree
		  ```
	- ## Step 9 $\implies$ Generate fstab file  
		-  
		  ```zsh
		  	genfstab -U /mnt > /mnt/etc/fstab
		  ```
	- ## Step 10 $\implies$ Change the working environment  
		-  
		  ```zsh
		  	arch-chroot /mnt
		  ```
	- ## Step 11 $\implies$ Configure the system  
		- Set date and time:  
			-  
			  ```zsh
			  	timedatectl set-time "year-month-day hours:minutes:seconds"
			  ```
		- Set the time zone:  
			-  
			  ```zsh
			  	ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
			  ```
		- Run hwclock to generate  `/etc/adjtime`:  
			-  
			  ```zsh
			  	hwclock --systohc
			  ```
		- Edit the `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed locales. Generate the locales by running:  
			-  
			  ```zsh
			  	locale-gen
			  ```
		- Create the `locale.conf` file, and set the `LANG` variable accordingly:  
			-  
			  ```conf
			  	LANG=en_US.UTF-8
			  ```
		- Create the hostname file:  
			-  
			  ```zsh
			  	touch /etc/hostname
			  ```
		- Edit the `/etc/hostname` and enter your hostname 
		- Edit the `/etc/hosts` file and enter the following:  
			-  
			  ```text
			  	127.0.0.1	localhost
			  	::1			localhost
			  	127.0.1.1	<hostname>.localdomain	<hostname>
			  ```
	- ## Step 12 $\implies$ Edit the mkinitcpio.conf file and create initramfs  
		- Add the `intel_agp`, `i915`, `btrfs` to the `MODULES=(...)` $\implies$ `MODULES=(intel_agp i915 btrfs)` (use strictly this order of modules)  
		- Add the `systemd`, `sd-encrypt` to the `HOOKS=(...)` $\implies$ `HOOKS=(base systemd autodetect modconf block sd-encrypt filesystems keyboard fsck)` (use strictly this order of hooks)  
		- Uncomment the `COMPRESSION="<CPOMRESSION_ALGORITHM>"` line with the desired compression
		- Edit other options if you wish  
		- Finally, generate initramfs:  
			-  
			  ```zsh
			  	mkinitcpio -p linux-hardened
			  ```
	- ## Step 13 $\implies$ rEFInd bootloader installation  
		-  
		  ```zsh
		  	refind-install
		  ```
	- ## Step 14 $\implies$ Edit the refind.conf file  
		- Add and/or change the following lines:  
			-  
			  ```conf
			  	menuentry "Arch Linux" {
			  		icon     /EFI/Boot/themes/rEFInd-minimal/icons/os_arch.png # or use a different path to os_arch.png
			  		volume   "Arch Linux"
			  		loader   /vmlinuz-linux-hardened # or use a different path to vmlinuz-linux-hardened
			  		initrd   /initramfs-linux-hardened.img # or use a different path to initramfs-linux-hardened.img
			  		options  "rd.luks.name=<UUID of the root filesystem>=<LUKS-devicename> root=/dev/mapper/<LUKS-devicename> rootfstab=btrfs rootflags=subvol=@ root_trim=yes rw quiet nmi_watchdog=0 kernel.unprivileged_userns_clone=0 net.core.bpf_jit_harden=2 apparmor=1 lsm=lockdown,yama,apparmor systemd.unified_cgroup_hierarchy=1 add_efi_memmap initrd=/intel-ucode.img"
			  		submenuentry "Boot using fallback initramfs" {
			  			initrd /initramfs-linux-hardened-fallback.img # or use a different path to initramfs-linux-hardened-fallback.img
			  		}
			  		submenuentry "Boot to terminal" {
			  			add_options "systemd.unit=multi-user.target"
			  		}
			  	}
			  ```
		- If you want to use dualboot with Windows, then add the following:  
			-  
			  ```conf
			  	menuentry "Windows 10" {
			  		loader \EFI\Microsoft\Boot\bootmgfw.efi # or use a different path to bootmgfw.efi
			  	}
			  ```
	- ## Step 15 $\implies$ Create the ZRAM service for swap space as a Systemd unit  
		- First of all, create bash scripts `zram.start` and `zram.stop` in the `~./zram` directory  
			- `zram.start`:  
				-  
				  ```bash
				  	#!/bin/bash
				  					  
				  	modprobe zram
				  					  
				  	SIZE=<RAM_GB / 4>
				  	MAX_COMP_STREAMS=<# of CPU cores>
				  	COMP_ALGORITHM=<desired compression algorithm>
				  					  
				  	echo $COMP_ALGORITHM > /sys/block/zram0/comp_algorithm
				  	echo $MAX_COMP_STREAMS > /sys/block/zram0/max_comp_streams
				  	echo $SIZE > /sys/block/zram0/disksize
				  					  
				  	mkswap /dev/zram0
				  					  
				  	swapon /dev/zram0 -p 10
				  ```
			- `zram.stop`:  
				-  
				  ```bash
				  	#!/bin/bash
				  					  
				  	swapoff /dev/zram0
				  					  
				  	echo 1 > /sys/block/zram0/reset
				  					  
				  	modprobe -r zram
				  ```
		- Then, create the systemd unit file in the `/etc/systemd/system` directory:  
			- `zram.service`:  
				-  
				  ```service
				  	[Unit]
				  	Description=ZRAM-based swap (compressed RAM block devices)
				  					  
				  	[Service]
				  	Type=oneshot
				  	ExecStart=/home/<username>/zram/zram.start
				  	ExecStop=/home/<username>/zram/zram.stop
				  	RemainAfterExit=yes
				  					  
				  	[Install]
				  	WantedBy=multi-user.target
				  ```
		- Enable the newly created service:  
			-  
			  ``` zsh
			  	systemctl enable zram.service
			  ```
	- ## Step 16 $\implies$ Miscellaneous  
		- Optimize and customize `makepkg` by editing the `/etc/makepkg.conf` file:  
			-  
			  ``` conf
			  	...
			  				  
			  	#-- Compiler and Linker Flags
			  	#CPPFLAGS=""
			  	CFLAGS="-march=native -mtune=native -O2 -pipe -fno-plt -fexceptions \
			  			-Wp,-D_FORTIFY_SOURCE=2 -Wformat -Werror=format-security \
			  			-fstack-clash-protection -fcf-protection"
			  	CXXFLAGS="$CFLAGS -Wp,-D_GLIBCXX_ASSERTIONS"
			  	LDFLAGS="-Wl,-O1,--sort-common,--as-needed,-z,relro,-z,now"
			  	LTOFLAGS="-flto=auto"
			  	RUSTFLAGS="-C opt-level=2 -C target-cpu=native"
			  	#-- Make Flags: change this for DistCC/SMP systems
			  	MAKEFLAGS="-j<# of CPU cores> --quiet"
			  	#-- Debugging flags
			  	DEBUG_CFLAGS="-g"
			  	DEBUG_CXXFLAGS="$DEBUG_CFLAGS"
			  	DEBUG_RUSTFLAGS="-C debuginfo=2"
			  				  
			  	...
			  				  
			  	#-- Specify a directory for package building.
			  	BUILDDIR=/home/<username>/makepkg
			  				  
			  	...
			  ```
	- ## Step 17 $\implies$ Create new user and set password for new user and root user
		- Add user:  
			-  
			  ```zsh
			  	useradd -m -G  docker,audio,disk,floppy,optical,scanner,input,kvm,storage,video,wheel -s /bin/zsh <username>
			  ```
		- Set password for user:  
			-  
			  ```zsh
			  	passwd <username>
			  ```
		- Edit the `/etc/sudoers` file and uncomment `%wheel ALL=(ALL:ALL) ALL`  
		- Set password for root user:  
			-  
			  ```zsh
			  	passwd root
			  ```
	- ## Step 18 $\implies$ Reboot and log into the newly installed system  
		-  
		  ```zsh
		  	exit
		  ```
		-  
		  ```zsh
		  	reboot
		  ```
