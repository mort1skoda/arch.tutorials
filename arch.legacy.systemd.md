#### archlinux tutorial
#### legacy bios systemd
#### asus k50



#### prep ####################################################{{{

#### download iso

   [download iso](https://archlinux.org/download)

#### wget iso file

    wget https://mirrors.dotsrc.org/archlinux/iso/2023.02.01/archlinux-x86_64.iso

#### wget sig file

    wget https://mirros.dotsrc.org/archlinux/iso/2023.02.01/archlinux-x86_64.iso.sig

#### wget b2sums.txt

    wget https://mirror.rackspace.com/archlinux/iso/2023.02.01/b2sums.txt

#### wget sha256sums.txt

    wget https://mirror.rackspace.com/archlinux/iso/2023.02.01/sha256sums.txt

<pre>
Open b2sums.txt and sha256sums.txt,
open the files and remove remove all lines,
exept for the entry with the iso file you downloaded.
</pre>

#### verify signature

    b2sum -c b2sums.txt | grep --color OK
    sha256sum -c sha256sums.txt | grep --color OK
    sudo pacman -S squoia-sq
    sq wkd get pierre@archlinux.org > release-key.pgp
    sq verify --signer-cert release-key.pgp --detached archlinux-x86_64.iso.sig archlinux-x86_64.iso
    gpg --auto-key-locate clear,wkd -v --locate-external-key pierre@archlinux.org
    gpg --keyserver-options auto-key-retrieve --verify archlinux-x86_64.iso.sig

    The below instruction will only work on an existing arch linux installation: 
    pacman-key -v archlinux-x86_64.iso.sig

#### make bootable usb

    sudo dd bs=1M if=archlinux-x86_64.iso of=/dev/sdx conv=fsync oflag=direct status=progress
#### ..........................................................}}}



#### boot the live environment ------------------{{{
    asus k50 spam ESC, select boot device

---> provide image of archlinux live boot screen <---

#### --------------------------------------------}}}



#### -- initial settings -----------------------{{{

    loadkeys no
    set -o vi
    alias l='ls -la --color --group-directories-first'
    passwd

    cd ~
    mkdir sda7
    mount /dev/sda7 sda7
    cd sda7/dat.mnt/dotfiles
    cp -r .* ~
    cd ~
    tm          #start tmux 
#### -- ------- -------- -----------------------}}}



#### -- connect to internet --------------------{{{

    in tmux:
    one pane for: 
    wip             = watch ip

    Option1 ---> using iwctl: 
    iwctl
    [iwd]# device list
    [iwd]# station wlan0 get-networks
    [iwd]# station wlan0 connect '103B 2.4'
    Passphrase: ********
    [iwd]# ctrl-d
    ip a
    ping -c4 archlinux.org

    Option2 ---> using wpa_supplicant:
    wpa_passphrase "103B 2.4" "sdbyorgufjuad" >> /etc/wpa_supplicant/wlan0.conf
    wpa_supplicant -B -iwlan0 -c/etc/wpa_supplicant/wlan0.conf
    ip a
    ping -c4 archlinux.org

    Option3 ???? can we use networkmanager here? 


#### -- ------- -- -------- --------------------}}}



#### -- work on the host -----------------------{{{

    Switch between tty1-6: Alt+arrow
    You can use lynx to read instructions.
    lynx archlinux.org
    o=option to set vi keys
    Or use tmux and read this document. 
#### -- ---- -- --- ---- -----------------------}}}



#### -- work via ssh client --------------------{{{

    rm .ssh
    ssh root@10.0.0.56
    set -o vi
    alias l='ls -la --color --group-directories-first'
#### -- ---- --- --- ------ --------------------}}}



#### -- partiton ----------------------------{{{

    fdisk -l
    lsblk
    blkid

    fdisk /dev/<the_disk_to_be_partitioned>

<pre>
At this point I went into Windows 10 and run MiniTool Partition Wizard
To make a swap partition and 3 linux installation partitons.
</pre>
#### -- -------- ----------------------------}}}



#### -- format ------------------------------{{{

    lsblk
    mkswap /dev/sda5
    mkfs.ext4 /dev/sda8

---

Provide an image here to see the layout of the ssd on asus.k50

---

#### -- ------- -----------------------------}}}



#### -- mount the file system ----------------{{{

    lsblk
    swapon /dev/sda5
    mount /dev/sda8 /mnt
    lslbk
#### -- ----- --- ---- ------ ----------------}}}



#### -- pacstrap -------------------------------{{{

    lscpu | gdat.mnt -i Vendor
    pacstrap -K /mnt intel-ucode
    pacstrap -K /mnt base base-devel
    pacstrap -K /mnt linux linux-firware
    pacstrap -K /mnt vim sudo openssh 
    pacstrap -K wpa_supplicant dhcpcd


    #pacstrap -K /mnt networkmanager
        
#### -- -------- ------------------------------}}} 



#### -- fstab --------------------------------{{{

    cat /mnt/etc/fstab
    genfstab -U /mnt >> /mnt/etc/fstab
    cat /mnt/etc/fstab
#### -- ----- --------------------------------}}}



#### -- chroot ---------------------------------{{{

    arch-chroot /mnt
    set -o vi
    alias l='ls -la --color --group-directories-first'
#### -- ------ ---------------------------------}}}



#### -- data directory -----------{{{
" create a directory where I can mount common data,
" ntfs to be shared with linux and windows.

    cd /
    mkdir dat.mnt
    pacman -S ntfs-3g
#### .. .... ........ ............}}}



#### -- time zone ------------------------------{{{

    ln -svf /usr/share/zoneinfo/Europa/Oslo /etc/localtime
    hwclock --systohc
#### -- ---- ---- ------------------------------}}}



#### -- localiztion ----------------------------{{{

    vim /etc/locale.gen     [uncomment en_US.UTF-8 UTF-8]
    cat /etc/locale.gen | gdat.mnt en_US
    locale-gen

    vim /etc/locale.conf    [LANG=en_US.UTF-8]   (esc then shift zz to quit)
    vim /etc/vconsole.conf  [KEYMAP=no]
    
    setfont drdos8x14

#### -- ----------- ----------------------------}}}



#### -- network confiuration -------------------{{{

    vim /etc/hostname       [arch.k50]

    pacman -S networkmanager
    systemctl enable NetworkManager.service

    or:
    pacman -S wpa_supplicant dhcpcd
    ####systemctl enable wpa_supplicant
    vim /etc/dhcpcd.conf        [add noarp]
    systemctl enable dhcpcd

    pacman -S openssh
    systemctl enable sshd
#### -- ------- ------------ -------------------}}}



#### -- root password --------------------------{{{

    passwd
#### -- ---- -------- --------------------------}}}



#### -- bootloader ----------------------------{{{

    pacman -S grub os-prober

    grub-install --target=i386-pc /dev/sda

    vim /etc/default/grub   [uncomment: GRUB_DISABLE_OS_PROBER=false] 

    grub-mkconfig -o /boot/grub/grub.cfg


    pacman -S ntfs-3g       (if windows)

    mount /dev/sda1 /mnt/sda1
    grub-mkconfig -o /boot/grub/grub.cfg
#### -- ---------- ----------------------------}}}



#### -- reboot --------------------------------{{{

    ctrl-d
    umount -R /mnt
    reboot
#### -- ------ --------------------------------}}}



#### -- first time reboot settings ------------{{{

    login as root
    set -o vi
    alias l='ls -la --color --group-directories-first'
#### -- ----- ---- ------ -------- ------------}}}    



#### -- connect to wifi -----------------------{{{

   Using wpa_supplicant:
    wpa_passphrase "103B 2.4" "sdbyorgufjuad"\
        >> /etc/wpa_supplicant/wpa_supplicant-wlp2s0.conf
    wpa_supplicant -B -iwlp2s0\
        -c/etc/wpa_supplicant/wpa_supplicant-wlp2s0.conf

    see my .bash_profile file

   Or using networkmanager:
    nmcli device wifi list
    nmcli device wifi connect '103B 2.4' password sdbyorgufjuad


    ip -color a
    ping -c4 archlinux.org
#### -- ------- -- ---- -----------------------}}}



#### -- useradd ------------------------------{{{

    useradd -mG wheel m
    passwd m
#### -- --- ---- -----------------------------}}}



#### -- visudo -------------------------------{{{

    EDITOR=/usr/bin/vim visudo
        [add at top: Defaults editor=/usr/bin/vim]
        [uncomment %wheel
#### -- ------ -------------------------------}}}



#### -- logout login ------------------------{{{

    exit to logout
    login as m
#### -- ------ ----- ------------------------}}}



#### -- git ---------------------------------{{{

    sudo pacman -S git github-cli
    sudo mkdir dat.mnt/
    sudo chown -R m:m /dat.mnt

    cd /dat*/dot*

    (((git clone https://github.com/mort1skoda/dotfiles.git)))
#### -- --- ---------------------------------}}} 



#### create symlinks ---------------------------{{{

   "Integrate your archlinux installation with your dotfiles dat.mnto"
   
   Create symlink in ~ that point to
   /dat.mnt/dotfiles.

There is a bash (sh) file in /dat.mnt/dotfiles/ named:
dotf.symlinks.sh that creates theese symlinks.

    ./dat.mnt/dotfiles/dotf.symlinks.sh
#### ------ -------- ---------------------------}}}



