# 2. Vaja: Diski in diskovni sistemi

## Navodila

1. Pregled Master Boot Record-a.
2. Priključitev navideznih diskov.
3. Priključitev RAID diskov.
4. Priključitev LVM diskov.

## Dodatne informacije

## Podrobna navodila

### Pregled Master Boot Record-a

Za pregled Master Boot Record-a (MBR) oz. prvega sektorja na disku, ki vsebuje sistemski nalagalnik ter tabelo razdelkov bomo uporabili orodje [`radare2`](https://github.com/radareorg/radare2), ki ga namestimo z njihovega repozitorija.

    apt update
    apt install git
    git clone https://github.com/radareorg/radare2
    radare2/sys/install.sh

Sedaj poženemo `radare2` nad želenim diskom ter z črko `v` izberemo grafični vmesnik, ki nam prikaže vsebino diska

    r2 /dev/sda
    VV
    View\Hexdump

    gdisk
    p
    c
    1
    DF Linux
    w
    y

    r2 /dev/sda
    VV
    View\Hexdump


### Priključitev navideznih diskov

S [spletne strani](https://polz.si/dsrf/) prenesemo naslednje datoteke:

- `furry.iso`
- `raid0_1.vdi`
- `raid0_2.vdi`

Da dodamo navidezne diske v naš navidezni računalnik, izberemo navidezni računalnik in kliknemo na gumb `Nastavitve...` v vrstici zgoraj. V oknu, ki se nam odpre izberemo zavihek `Pomnilniške naprave` iz stolpca na levi. Nato izberemo `Krmilnik: IDE` in dodamo zgoščenko, tako da kliknemo na gumb z zgoščenko in zelenim znakom plus ter sledimo čarovniku za izbiro datoteke, kjer izberemo `furry.iso`.

![Postopek dodajanja zgoščenke navideznemu računalniku.](slike/vaja3-vbox1.png)

Navidezne diske dodamo, tako da izberemo `Krmilnik:SATA` in dodamo disk, tako da kliknemo na gumb s diskom in zelenim znakom plus ter sledimo čarovniku za izbiro datoteke, kjer izberemo `raid0_1.vdi`. Po enakem postopku dodamo še navidezni disk `raid0_2.vdi`.

![Postopek dodajanja navideznega diska navideznemu računalniku.](slike/vaja3-vbox2.png)

Sedaj imamo v zavihku `Pomnilniške naprave` pod `Krmilnik: IDE` dodano zgoščenko `furry.iso` in pod `Krmilnik:SATA` dodana navidezna diska `raid0_1.vdi` in `raid0_2.vdi`.

![Pregled vseh dodanih zgoščen in navideznih diskov.](slike/vaja3-vbox3.png)

Sedaj poženemo naš navidezni računalnik. V operacijskih sistemih GNU/Linux lahko do diskov, ki so priklopljeni dostopamo preko imenika `/dev/NAME_OF_THE_DISK`, kjer je ponavadi ime diska `sdX`, kjer `X` zamenjamo z zaporedno črko abecede.

    cd /dev
    ls /dev
    cd /dev/sda

Na katero pot v datotečnem sistemu so posamezni diski (`sdX`) in razdelki (`sdXN`) priklopljeni ugotovimo preko imenika `/dev/NAME_OF_THE_DISK_OR_PARTITION/by-path`.

    ls -al /dev/sda/by-path
    cd /dev/sda/by-path

Priklopljene diske (`sdX`) in razdelke (`sdXN`) lahko izpišemo tudi z ukazom `lsblk`.

    lsblk

    NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sda      8:0    0  100G  0 disk 
    ├─sda1   8:1    0   99G  0 part /
    ├─sda2   8:2    0    1K  0 part 
    └─sda5   8:5    0  975M  0 part [SWAP]
    sdb      8:16   0    2G  0 disk 
    ├─sdb1   8:17   0  300M  0 part 
    ├─sdb2   8:18   0  299M  0 part 
    ├─sdb3   8:19   0    1K  0 part 
    └─sdb5   8:21   0  1.4G  0 part 
    sdc      8:32   0    2G  0 disk 
    ├─sdc1   8:33   0  300M  0 part 
    ├─sdc2   8:34   0    1K  0 part 
    └─sdc5   8:37   0  1.7G  0 part 
    sr0     11:0    1  3.3M  0 rom 

V operacijskih sistemih GNU/linux za delo z diski uporabljamo orodja kot so: `mount`, `fdisk`, `gdisk`, `libguestfs`, `qemu-img`, `qemu-nbd`...  Razdelki se predstavijo operacijskemu sistemu kot datoteke. Podatke znotraj razdelkov so urejene v drevesno strukturo, ki predstavlja datotečni sistem, ki ustvari svojo glavo na začetku razdelka in nato sledijo datoteke. 

Sedaj naredimo kopije obeh navideznih diskov in zgoščenke z ukazom [`cat`]() ter izračunamo integritetno vrednost diska z ukazom [`sha512sum`]().

    sha512sum /dev/sdb >> sdb_hash.txt
    cat /dev/sdb > sdb_copy.img
    sha512sum sdb_copy.img >> sdb_hash.txt
    cat sdb_hash.txt

    d4b009680058854c20324d830f189a79c2318fc8474681a43d3a10559b1fbadeda855f9e9aab510c2bd98ed379a2800526f64c49add1eb21e68e0af5c3f76c3d  /dev/sdb
    d4b009680058854c20324d830f189a79c2318fc8474681a43d3a10559b1fbadeda855f9e9aab510c2bd98ed379a2800526f64c49add1eb21e68e0af5c3f76c3d  sdb_copy.img

    sha512sum /dev/sdc >> sdc_hash.txt
    cat /dev/sdc > sdc_copy.img
    sha512sum sdc_copy.img >> sdc_hash.txt
    cat sdc_hash.txt

    161c640b8d4db9784d288146c02d711cafdcf76d7dd4db01faa762d6e44ac5aff790938e2cc013d1264505c1ff722210f944f5047994fe0d0dbdbef87a91eca4  /dev/sdc
    161c640b8d4db9784d288146c02d711cafdcf76d7dd4db01faa762d6e44ac5aff790938e2cc013d1264505c1ff722210f944f5047994fe0d0dbdbef87a91eca4  sdc_copy.img

    sha512sum /dev/sr0 >> sr0_hash.txt
    cat /dev/sr0 > sr0_copy.img
    sha512sum sr0_copy.img >> sr0_hash.txt
    cat sr0_hash.txt

    1fc5046513087e4eead5966e4c4ff017da8c43325f3fef537648d395a313bcea332a8e9841c3a4c0c45b95ed8addf91adac89edecd91c6fdb1fa915e487a9b45  /dev/sr0
    1fc5046513087e4eead5966e4c4ff017da8c43325f3fef537648d395a313bcea332a8e9841c3a4c0c45b95ed8addf91adac89edecd91c6fdb1fa915e487a9b45  sr0_copy.img

Sedaj zaustavimo naš navidezni računalnik ter odstranimo odstranimo zgoščenko `furry.iso` iz `Krmilnik: IDE`, tako da jo izberemo, kliknemo na gumb z zgoščenko na desno strani ter izberemo možnost `Odstrani disk iz navideznega pogona`. Prav tako iz `Krmilnik:SATA` odstranimo navidezna diska `raid0_1.vdi` in `raid0_2.vdi`, tako da izberemo vsak nosilec in spodaj levo pritisnemo na gumb z disketo in rdečim križem.

Nato ponovno poženemo naš navidezni računalnik. Začnemo s priklopom slike zgoščenke, ki je ne moremo spreminjati. Podatki so zapisani z namenskim [datotečnim sistemom ISO9660](https://en.wikipedia.org/wiki/ISO_9660). Za priključitev zgoščenke uporabimo ukaz [`mount`](https://man7.org/linux/man-pages/man2/mount.2.html) ter preverimo katere nove datoteke so nam sedaj dostopne.

    mkdir /mnt/sr0
    mount sr0_copy.img /mnt/sr0
    
    mount: /mnt/sr0: WARNING: source write-protected, mounted read-only.

    ls /mnt/sr0

    cute-cat-l.jpg	kitty1.jpeg  kitty2.jpeg  kitty3.jpeg  kitty4.jpg

ISO9660 omejuje dolžino imen, ki jih imajo lahko datoteke, da zaobidemo to omejitev se privzeto uporablja [Joliet razširitev datotečnega sistema](https://en.wikipedia.org/wiki/ISO_9660#Joliet). Sedaj bomo najprej odklopili trenutno priklopljeno zgoščenko in jo nato ponovno priklopili brez Joliet razširitve ter preverili ali sedaj lahko dostopamo še do kakšnih datotek.

    mount

    sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
    proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
    udev on /dev type devtmpfs (rw,nosuid,relatime,size=1987160k,nr_inodes=496790,mode=755)
    devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
    tmpfs on /run type tmpfs (rw,nosuid,nodev,noexec,relatime,size=402440k,mode=755)
    /dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro)
    securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
    tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
    tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k)
    cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot)
    pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
    none on /sys/fs/bpf type bpf (rw,nosuid,nodev,noexec,relatime,mode=700)
    systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=29,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=14377)
    tracefs on /sys/kernel/tracing type tracefs (rw,nosuid,nodev,noexec,relatime)
    hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,pagesize=2M)
    debugfs on /sys/kernel/debug type debugfs (rw,nosuid,nodev,noexec,relatime)
    mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
    fusectl on /sys/fs/fuse/connections type fusectl (rw,nosuid,nodev,noexec,relatime)
    configfs on /sys/kernel/config type configfs (rw,nosuid,nodev,noexec,relatime)
    tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,size=402436k,nr_inodes=100609,mode=700,uid=1000,gid=1000)
    gvfsd-fuse on /run/user/1000/gvfs type fuse.gvfsd-fuse (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)
    /root/sr0_copy.img on /mnt/sr0 type iso9660 (ro,relatime,norock,check=r,map=n,blocksize=2048,iocharset=utf8)

    umount /mnt/sr0

    mount -o nojoliet,unhide sr0_copy.img /mnt/sr0

    mount: /mnt/sr0: WARNING: source write-protected, mounted read-only.

    ls /mnt/sr0

    cat_pict.jpg  evil_cat.jpg  jojkj.png	kitty2.jpe  kitty4.jpg	sl_1___w.htm cute_cat.jpg  grumpy_c.jpg  kitty1.jpe	kitty3.jpe  sl_1___w

Vidimo, da imajo datotečni sistemi svoja stikala oz. parametre, ki spremenijo kako datotečni sistemi delujejo in se odzivajo na ukaze. 

Sedaj pa bomo priklopili še navidezne diske na naš datotečni sistem. Priklopimo jih lahko z ukazom `mount` in tudi z drugimi bolj specializiranimi orodji kot sta [`qemu-nbd`](https://www.mankier.com/8/qemu-nbd) in [`kpartx`](https://linux.die.net/man/8/kpartx). Obe orodji najprej namesti z uporabo upravljalca paketov na našem operacijskem sistemu.

    apt update
    apt install qemu-utils kpartx

Omogočimo `qemu-nbd` v našem operacijskem sistemu s programom za dodajanje in odstranjevanje modulov jedra [`modprobe`](https://linux.die.net/man/8/modprobe).

    ls /dev

    autofs		    kmsg	       rtc	     tty16	tty38  tty6	     vcsa
    block		    log	           rtc0	     tty17	tty39  tty60	 vcsa1
    bsg		        loop0	       sda	     tty18	tty4   tty61	 vcsa2
    btrfs-control	loop1	       sda1	     tty19	tty40  tty62	 vcsa3
    bus		        loop2	       sda2	     tty2	tty41  tty63	 vcsa4
    cdrom		    loop3	       sda5	     tty20	tty42  tty7	     vcsa5
    char		    loop4	       sg0	     tty21	tty43  tty8	     vcsa6
    console		    loop5	       sg1	     tty22	tty44  tty9	     vcsu
    core		    loop6	       shm	     tty23	tty45  ttyS0	 vcsu1
    cpu		        loop7	       snapshot  tty24	tty46  ttyS1	 vcsu2
    cpu_dma_latency loop-control   snd	     tty25	tty47  ttyS2	 vcsu3
    cuse		    mapper         sr0	     tty26	tty48  ttyS3	 vcsu4
    disk		    mem	           stderr	 tty27	tty49  uhid	     vcsu5
    dri		        mqueue         stdin	 tty28	tty5   uinput	 vcsu6
    dvd		        net	           stdout	 tty29	tty50  urandom	 vfio
    fb0		        null	       tty	     tty3	tty51  vboxguest vga_arbiter
    fd		        nvram	       tty0	     tty30	tty52  vboxuser  vhci
    full		    port	       tty1	     tty31	tty53  vcs	     vhost-net
    fuse		    ppp	           tty10	 tty32	tty54  vcs1	     vhost-vsock
    hidraw0		    psaux	       tty11	 tty33	tty55  vcs2	     zero
    hpet		    ptmx	       tty12	 tty34	tty56  vcs3
    hugepages	    pts	           tty13	 tty35	tty57  vcs4
    initctl		    random         tty14	 tty36	tty58  vcs5
    input		    rfkill         tty15	 tty37	tty59  vcs6

    modprobe nbd max_part=32

    ls /dev

    autofs		    loop0	       nbd8	    stdout   tty30	tty54	   vcs3
    block		    loop1	       nbd9	    tty	     tty31	tty55	   vcs4
    bsg		        loop2	       net	    tty0	 tty32	tty56	   vcs5
    btrfs-control	loop3	       null	    tty1	 tty33	tty57	   vcs6
    bus		        loop4	       nvram	tty10	 tty34	tty58	   vcsa
    cdrom		    loop5	       port	    tty11	 tty35	tty59	   vcsa1
    char		    loop6	       ppp	    tty12	 tty36	tty6	   vcsa2
    console		    loop7	       psaux	tty13	 tty37	tty60	   vcsa3
    core		    loop-control   ptmx	    tty14	 tty38	tty61	   vcsa4
    cpu		        mapper         pts	    tty15	 tty39	tty62	   vcsa5
    cpu_dma_latency mem	           random	tty16	 tty4	tty63	   vcsa6
    cuse		    mqueue         rfkill	tty17	 tty40	tty7	   vcsu
    disk		    nbd0	       rtc	    tty18	 tty41	tty8	   vcsu1
    dri		        nbd1	       rtc0	    tty19	 tty42	tty9	   vcsu2
    dvd		        nbd10	       sda	    tty2	 tty43	ttyS0	   vcsu3
    fb0		        nbd11	       sda1	    tty20	 tty44	ttyS1	   vcsu4
    fd		        nbd12	       sda2	    tty21	 tty45	ttyS2	   vcsu5
    full		    nbd13	       sda5	    tty22	 tty46	ttyS3	   vcsu6
    fuse		    nbd14	       sg0	    tty23	 tty47	uhid	   vfio
    hidraw0		    nbd15	       sg1	    tty24	 tty48	uinput	   vga_arbiter
    hpet		    nbd2	       shm	    tty25	 tty49	urandom    vhci
    hugepages	    nbd3	       snapshot tty26	 tty5	vboxguest  vhost-net
    initctl		    nbd4	       snd	    tty27	 tty50	vboxuser   vhost-vsock
    input		    nbd5	       sr0	    tty28	 tty51	vcs	       zero
    kmsg		    nbd6	       stderr	tty29	 tty52	vcs1
    log		        nbd7	       stdin	tty3	 tty53	vcs2

Sedaj priklopimo disk z ukazom `qemu-nbd` in izpišemo trenutno priklopljene diske z ukazom `lsblk`.

    qemu-nbd -c /dev/nbd0 sdb_copy.img

    WARNING: Image format was not specified for 'sdb_copy.img' and probing guessed raw.
         Automatically detecting the format is dangerous for raw images, write operations on block 0 will be restricted.
         Specify the 'raw' format explicitly to remove the restrictions.

    lsblk

    NAME     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    loop0      7:0    0  3.3M  0 loop /mnt/sr0
    sda        8:0    0  100G  0 disk 
    ├─sda1     8:1    0   99G  0 part /
    ├─sda2     8:2    0    1K  0 part 
    └─sda5     8:5    0  975M  0 part [SWAP]
    sr0       11:0    1 1024M  0 rom  
    nbd0      43:0    0    2G  0 disk 
    ├─nbd0p1  43:1    0  300M  0 part 
    ├─nbd0p2  43:2    0  299M  0 part 
    ├─nbd0p3  43:3    0    1K  0 part 
    └─nbd0p5  43:5    0  1.4G  0 part 

Z ukazom [`fdisk`](https://man7.org/linux/man-pages/man8/fdisk.8.html) lahko preverimo razdelke tako da izberemo možnost `p`. Ugotovimo, da imamo `LVM` razdelek in `RAID` razdelek, ki jih trenutno še ne moremo priključiti. Razdelek `Extended` nam predstavlja vsebovalnik za dodatne razdelke ter vsebuje še en `MBR` in tako omogoča uporabo še dodatnih štirih particij. Zadnji razdelek vsebuje datotečni sistem `FAT32`, katerega lahko priklopimo z ukazom `mount`.

    fdisk /dev/nbd0

    p

    Disk /dev/nbd0: 2 GiB, 2147483648 bytes, 4194304 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x20fc8508

    Device      Boot   Start     End Sectors  Size Id Type
    /dev/nbd0p1       614400 1228799  614400  300M fd Linux raid autodetect
    /dev/nbd0p2         2048  614399  612352  299M 8e Linux LVM
    /dev/nbd0p3      1228800 4194303 2965504  1.4G  5 Extended
    /dev/nbd0p5      1230848 4194303 2963456  1.4G  c W95 FAT32 (LBA)

    q

    mkdir /mnt/nbd0p5
    mount /dev/nbd0p5 /mnt/nbd0p5

    ls /mnt/nbd0p5

    'Algorithm March (Original)-M8xtOinmByo.mp4'
    "Survival - 'Living Off The Land' 1955 US Navy Training Film-KzplUDaBMuw.mp4"

Sedaj lahko preverimo integritetno vrednost navideznega diska `sdb_copy.img` in jo primerjamo s shranjeno vrednostjo v `sdb_hash.txt`. Vidimo, da se je vrednost že spremenila.

    sha512sum sdb_copy.img

    6f85ba2d828ffecbbfb1f9f59916bae13673d441dacfaff686ede8f47f761cf3ecd91bde4c487be0aeab112e2e245b5640c0c30235abeebf6bac2cd7233ea6b1  sdb_copy.img

    cat sdb_hash.txt 
    
    d4b009680058854c20324d830f189a79c2318fc8474681a43d3a10559b1fbadeda855f9e9aab510c2bd98ed379a2800526f64c49add1eb21e68e0af5c3f76c3d  /dev/sdb
    d4b009680058854c20324d830f189a79c2318fc8474681a43d3a10559b1fbadeda855f9e9aab510c2bd98ed379a2800526f64c49add1eb21e68e0af5c3f76c3d  sdb_copy.img

Za priključitev drugega diska pa uporabimo orodje `kpartx` in izpišemo trenutno priklopljene diske z ukazom `lsblk`.

    kpartx -a sdc_copy.img

    lsblk

    NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    loop0       7:0    0  3.3M  0 loop /mnt/sr0
    loop1       7:1    0    2G  0 loop 
    ├─loop1p1 254:0    0  300M  0 part 
    ├─loop1p2 254:1    0    1K  0 part 
    └─loop1p5 254:2    0  1.7G  0 part 
    sda         8:0    0  100G  0 disk 
    ├─sda1      8:1    0   99G  0 part /
    ├─sda2      8:2    0    1K  0 part 
    └─sda5      8:5    0  975M  0 part [SWAP]
    sr0        11:0    1 1024M  0 rom  
    nbd0       43:0    0    2G  0 disk 
    ├─nbd0p1   43:1    0  300M  0 part 
    ├─nbd0p2   43:2    0  299M  0 part 
    ├─nbd0p3   43:3    0    1K  0 part 
    └─nbd0p5   43:5    0  1.4G  0 part /mnt/nbd0p5

Z ukazom `fdisk` lahko preverimo razdelke tako da izberemo možnost `p`. Ugotovimo, da imamo `LVM`, `RAID` in `Extended` razdeleke, ki jih trenutno še ne moremo priključiti.

    fdisk /dev/loop1

    p

    Disk /dev/loop1: 2 GiB, 2147483648 bytes, 4194304 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0xb1b5b4b2

    Device       Boot  Start     End Sectors  Size Id Type
    /dev/loop1p1        2048  616447  614400  300M fd Linux raid autodetect
    /dev/loop1p2      616448 4194303 3577856  1.7G  5 Extended
    /dev/loop1p5      618496 4194303 3575808  1.7G 8e Linux LVM

    q

### Priključitev RAID diskov

Za delo z RAID polji potrebuje orodje [`mdadm`](https://linux.die.net/man/8/mdadm), ki ga namestimo z upravljaljcem paketov našega operacijskega sistema. Nato preverimo priključene disk z ukazom `lsblk` ali je prišlo do kakšne spremembe.

    apt update
    apt install mdadm

    lsblk

    NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    loop0       7:0    0  3.3M  0 loop /mnt/sr0
    loop1       7:1    0    2G  0 loop 
    ├─loop1p1 254:0    0  300M  0 part 
    ├─loop1p2 254:1    0    1K  0 part 
    └─loop1p5 254:2    0  1.7G  0 part 
    sda         8:0    0  100G  0 disk 
    ├─sda1      8:1    0   99G  0 part /
    ├─sda2      8:2    0    1K  0 part 
    └─sda5      8:5    0  975M  0 part [SWAP]
    sr0        11:0    1 1024M  0 rom  
    nbd0       43:0    0    2G  0 disk 
    ├─nbd0p1   43:1    0  300M  0 part 
    ├─nbd0p2   43:2    0  299M  0 part 
    ├─nbd0p3   43:3    0    1K  0 part 
    └─nbd0p5   43:5    0  1.4G  0 part /mnt/nbd0p5

Avtomatsko sistem ni zaznal nobenega RAID polja, zato ga poskusimo z uporabo orodja `mdadm` in pononvno preverimo priključene disk z ukazom `lsblk`.

    mdadm --assemble /dev/md0

    mdadm: /dev/md0 has been started with 2 drives.

    mdadm --detail /dev/md0

    /dev/md0:
            Version : 1.2
        Creation Time : Tue Mar 11 19:06:23 2014
            Raid Level : raid0
            Array Size : 612352 (598.00 MiB 627.05 MB)
        Raid Devices : 2
        Total Devices : 2
        Persistence : Superblock is persistent

        Update Time : Tue Mar 11 19:06:23 2014
                State : clean 
        Active Devices : 2
    Working Devices : 2
        Failed Devices : 0
        Spare Devices : 0

                Layout : -unknown-
            Chunk Size : 512K

    Consistency Policy : none

                Name : razpadalcek:0
                UUID : c639acac:267a1605:f2b03f0e:230038f7
                Events : 0

        Number   Major   Minor   RaidDevice State
        0     254        0        0      active sync   /dev/dm-0
        1      43        1        1      active sync   /dev/nbd0p1

    lsblk

    NAME        MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
    loop0         7:0    0  3.3M  0 loop  /mnt/sr0
    loop1         7:1    0    2G  0 loop  
    ├─loop1p1   254:0    0  300M  0 part  
    │ └─md0       9:0    0  598M  0 raid0 
    │   ├─md0p1 259:0    0  100M  0 part  
    │   └─md0p2 259:1    0  497M  0 part  
    ├─loop1p2   254:1    0    1K  0 part  
    └─loop1p5   254:2    0  1.7G  0 part  
    sda           8:0    0  100G  0 disk  
    ├─sda1        8:1    0   99G  0 part  /
    ├─sda2        8:2    0    1K  0 part  
    └─sda5        8:5    0  975M  0 part  [SWAP]
    sr0          11:0    1 1024M  0 rom   
    nbd0         43:0    0    2G  0 disk  
    ├─nbd0p1     43:1    0  300M  0 part  
    │ └─md0       9:0    0  598M  0 raid0 
    │   ├─md0p1 259:0    0  100M  0 part  
    │   └─md0p2 259:1    0  497M  0 part  
    ├─nbd0p2     43:2    0  299M  0 part  
    ├─nbd0p3     43:3    0    1K  0 part  
    └─nbd0p5     43:5    0  1.4G  0 part  /mnt/nbd0p5

Z orodjem `mdadm` smo ponovno sestavili obstoječe RAID 0 polje pod imenom `/dev/md0`. Z ukazom `fdisk` lahko preverimo razdelka v RAID polju tako da izberemo možnost `p`. Ugotovimo, da imamo `LVM` razdelek, ki ga trenutno še ne moremo priključiti in razdelek s poznanim datotečnim sistemom `Linux filesystem`, ki ga lahko priključimo.

    fdisk /dev/md0

    p

    Disk /dev/md0: 598 MiB, 627048448 bytes, 1224704 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 524288 bytes / 1048576 bytes
    Disklabel type: gpt
    Disk identifier: B7F76598-172A-4714-BC34-B9F014C55A39

    Device      Start     End Sectors  Size Type
    /dev/md0p1   2048  206847  204800  100M Linux filesystem
    /dev/md0p2 206848 1224670 1017823  497M Linux LVM

    q

    mkdir /mnt/md0p1
    mount /dev/md0p1 /mnt/md0p1

    ls /mnt/md0p1

    "dailamo dailamo video song from srikanth's movie mahatma-kSBWzQUWAjI.mp4"

### Priključitev LVM diskov

Za delo z LVM logičnimi diski potrebujemo orodje [`lvm2`](https://linux.die.net/man/8/lvm), ki ga namestimo z upravljalcem paketov našega operacijskega sistema. Nato preverimo priključene disk z ukazom `lsblk` ali je prišlo do kakšne spremembe

    apt update
    apt install lvm2

    lsblk

Podatke o `LVM` logičnih diskih pridobimo z ukazi: `pvdisplay` za fizične diske (`Physical Volume`), `vgdisplay` za skupine fizičnih diskov (`Volume Group`) in `lvdisplay` za logične diske (`Logical Volume`).

    pvdisplay

    WARNING: PV /dev/nbd0p2 in VG joyjoy is using an old PV header, modify the VG to update.
    --- Physical volume ---
    PV Name               /dev/nbd0p2
    VG Name               joyjoy
    PV Size               299.00 MiB / not usable 3.00 MiB
    Allocatable           yes (but full)
    PE Size               4.00 MiB
    Total PE              74
    Free PE               0
    Allocated PE          74
    PV UUID               ELf5O8-UJCr-SUHD-JXza-67qJ-RMeI-ZQT3Mh
    
    WARNING: PV /dev/mapper/loop1p5 in VG happy is using an old PV header, modify the VG to update.
    WARNING: PV /dev/md0p2 in VG happy is using an old PV header, modify the VG to update.
    --- Physical volume ---
    PV Name               /dev/mapper/loop1p5
    VG Name               happy
    PV Size               <1.71 GiB / not usable 2.00 MiB
    Allocatable           yes (but full)
    PE Size               4.00 MiB
    Total PE              436
    Free PE               0
    Allocated PE          436
    PV UUID               LOyxVs-UtKI-0jIU-Clcm-Hb1L-ruGl-ZGns8c
    
    --- Physical volume ---
    PV Name               /dev/md0p2
    VG Name               happy
    PV Size               496.98 MiB / not usable 4.98 MiB
    Allocatable           yes (but full)
    PE Size               4.00 MiB
    Total PE              123
    Free PE               0
    Allocated PE          123
    PV UUID               pXyscH-UvqH-5lV6-Gn6U-TKGX-MTqH-hOrrPM

    vgdisplay

    WARNING: PV /dev/nbd0p2 in VG joyjoy is using an old PV header, modify the VG to update.
    --- Volume group ---
    VG Name               joyjoy
    System ID             
    Format                lvm2
    Metadata Areas        1
    Metadata Sequence No  2
    VG Access             read/write
    VG Status             resizable
    MAX LV                0
    Cur LV                1
    Open LV               0
    Max PV                0
    Cur PV                1
    Act PV                1
    VG Size               296.00 MiB
    PE Size               4.00 MiB
    Total PE              74
    Alloc PE / Size       74 / 296.00 MiB
    Free  PE / Size       0 / 0   
    VG UUID               SPcqO8-Awtf-N0MK-5g2y-F6Do-YO9g-v05FdI
    
    WARNING: PV /dev/mapper/loop1p5 in VG happy is using an old PV header, modify the VG to update.
    WARNING: PV /dev/md0p2 in VG happy is using an old PV header, modify the VG to update.
    --- Volume group ---
    VG Name               happy
    System ID             
    Format                lvm2
    Metadata Areas        2
    Metadata Sequence No  3
    VG Access             read/write
    VG Status             resizable
    MAX LV                0
    Cur LV                2
    Open LV               0
    Max PV                0
    Cur PV                2
    Act PV                2
    VG Size               2.18 GiB
    PE Size               4.00 MiB
    Total PE              559
    Alloc PE / Size       559 / 2.18 GiB
    Free  PE / Size       0 / 0   
    VG UUID               U6IpQJ-JTs5-FoLT-Oi6S-Q2lf-dCTO-iwC91f

    lvdisplay

    WARNING: PV /dev/nbd0p2 in VG joyjoy is using an old PV header, modify the VG to update.
    --- Logical volume ---
    LV Path                /dev/joyjoy/ren
    LV Name                ren
    VG Name                joyjoy
    LV UUID                VJMlcK-pZDE-bavi-PSB2-2Zne-Vvd3-0M2FL7
    LV Write Access        read/write
    LV Creation host, time razpadalcek, 2014-03-12 07:30:36 +0100
    LV Status              NOT available
    LV Size                296.00 MiB
    Current LE             74
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    
    WARNING: PV /dev/mapper/loop1p5 in VG happy is using an old PV header, modify the VG to update.
    WARNING: PV /dev/md0p2 in VG happy is using an old PV header, modify the VG to update.
    --- Logical volume ---
    LV Path                /dev/happy/stimpy
    LV Name                stimpy
    VG Name                happy
    LV UUID                3SEwn6-eI05-1ef8-6I8j-EJtU-e79q-YaGQ1K
    LV Write Access        read/write
    LV Creation host, time razpadalcek, 2014-03-12 07:45:03 +0100
    LV Status              NOT available
    LV Size                500.00 MiB
    Current LE             125
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    
    --- Logical volume ---
    LV Path                /dev/happy/bmo
    LV Name                bmo
    VG Name                happy
    LV UUID                C94u1d-mp2T-6HjY-wSdC-sa2Y-acnu-evlaco
    LV Write Access        read/write
    LV Creation host, time razpadalcek, 2014-03-12 07:47:11 +0100
    LV Status              NOT available
    LV Size                <1.70 GiB
    Current LE             434
    Segments               2
    Allocation             inherit
    Read ahead sectors     auto

Vse logične diske lahko priklopimo na enkrat z ukazom `vgchange` in preverimo priključene disk z ukazom `lsblk`.

    vgchange -a y

    WARNING: PV /dev/nbd0p2 in VG joyjoy is using an old PV header, modify the VG to update.
    1 logical volume(s) in volume group "joyjoy" now active
    WARNING: PV /dev/mapper/loop1p5 in VG happy is using an old PV header, modify the VG to update.
    WARNING: PV /dev/md0p2 in VG happy is using an old PV header, modify the VG to update.
    2 logical volume(s) in volume group "happy" now active

    lsblk

    NAME              MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
    loop0               7:0    0  3.3M  0 loop  /mnt/sr0
    loop1               7:1    0    2G  0 loop  
    ├─loop1p1         254:0    0  300M  0 part  
    │ └─md0             9:0    0  598M  0 raid0 
    │   ├─md0p1       259:0    0  100M  0 part  /mnt/md0p1
    │   └─md0p2       259:1    0  497M  0 part  
    │     └─happy-bmo 254:5    0  1.7G  0 lvm   /mnt/bmo
    ├─loop1p2         254:1    0    1K  0 part  
    └─loop1p5         254:2    0  1.7G  0 part  
      ├─happy-stimpy  254:4    0  500M  0 lvm   /mnt/stimpy
      └─happy-bmo     254:5    0  1.7G  0 lvm   /mnt/bmo
    sda                 8:0    0  100G  0 disk  
    ├─sda1              8:1    0   99G  0 part  /
    ├─sda2              8:2    0    1K  0 part  
    └─sda5              8:5    0  975M  0 part  [SWAP]
    sr0                11:0    1 1024M  0 rom   
    nbd0               43:0    0    2G  0 disk  
    ├─nbd0p1           43:1    0  300M  0 part  
    │ └─md0             9:0    0  598M  0 raid0 
    │   ├─md0p1       259:0    0  100M  0 part  /mnt/md0p1
    │   └─md0p2       259:1    0  497M  0 part  
    │     └─happy-bmo 254:5    0  1.7G  0 lvm   /mnt/bmo
    ├─nbd0p2           43:2    0  299M  0 part  
    │ └─joyjoy-ren    254:3    0  296M  0 lvm   /mnt/ren
    ├─nbd0p3           43:3    0    1K  0 part  
    └─nbd0p5           43:5    0  1.4G  0 part  /mnt/nbd0p5

Sedaj še priključimo vse tri logične diske z ukazom `mount` in preverimo do katerih novih datotek lahko sedaj dostopamo.

    mkdir /mnt/ren
    mount /dev/joyjoy/ren /mnt/ren

    ls /mnt/ren

    lost+found										       'Волга-Волга   Молодежная-P4f410gVNiE.mp4'
    'Po pi po ~ Miku Hatsune Vegetable Juice Dance (English subtitles, Download)-T0-2lzA7_Cg.mp4'

    mkdir /mnt/stimpy
    mount /dev/happy/stimpy /mnt/stimpy

    ls /mnt/stimpy

    '[HD_HQ Music Video] T-ara - Bo Peep Bo Peep-9403-9CptH8.mp4'

    mkdir /mnt/bmo
    mount /dev/happy/bmo /mnt/bmo

    ls /mnt/bmo

    '[HD_HQ Music Video] T-ara - Bo Peep Bo Peep-9403-9CptH8.mp4'  'Ken Ashcorp - 20 Percent Cooler-u6xRSafBV8o.mp4'   lost+found

Vse trenutno priključene diske, razdelke, datotčne sisteme, RAID polja ter LVM logične diske lahko prikažemo še z detaljnim izpisom ukaza `lsblk`.

    lsblk -f

    NAME              FSTYPE            FSVER            LABEL         UUID                                   FSAVAIL FSUSE% MOUNTPOINT
    loop0             iso9660           Joliet Extension CDROM         2014-03-11-17-03-58-00                       0   100% /mnt/sr0
    loop1                                                                                                                    
    ├─loop1p1         linux_raid_member 1.2              razpadalcek:0 c639acac-267a-1605-f2b0-3f0e230038f7                  
    │ └─md0                                                                                                                  
    │   ├─md0p1       ntfs                                             0D5AF5EB6D625788                            6M    94% /mnt/md0p1
    │   └─md0p2       LVM2_member       LVM2 001                       pXyscH-UvqH-5lV6-Gn6U-TKGX-MTqH-hOrrPM                
    │     └─happy-bmo ext4              1.0                            2169f641-3fc0-4b11-adda-2ef58ff9cc23      1.5G     5% /mnt/bmo
    ├─loop1p2                                                                                                                
    └─loop1p5         LVM2_member       LVM2 001                       LOyxVs-UtKI-0jIU-Clcm-Hb1L-ruGl-ZGns8c                
      ├─happy-stimpy  btrfs                                            0e4f1042-3865-4285-8e94-d532cbb3a6ab      426M    14% /mnt/stimpy
      └─happy-bmo     ext4              1.0                            2169f641-3fc0-4b11-adda-2ef58ff9cc23      1.5G     5% /mnt/bmo
    sda                                                                                                                      
    ├─sda1            ext4              1.0                            6a286179-47aa-4c52-b942-5198b263c2f3     83.2G     9% /
    ├─sda2                                                                                                                   
    └─sda5            swap              1                              2b7cf7ee-8008-43c0-b786-6757301b91db                  [SWAP]
    sr0                                                                                                                      
    nbd0                                                                                                                     
    ├─nbd0p1          linux_raid_member 1.2              razpadalcek:0 c639acac-267a-1605-f2b0-3f0e230038f7                  
    │ └─md0                                                                                                                  
    │   ├─md0p1       ntfs                                             0D5AF5EB6D625788                            6M    94% /mnt/md0p1
    │   └─md0p2       LVM2_member       LVM2 001                       pXyscH-UvqH-5lV6-Gn6U-TKGX-MTqH-hOrrPM                
    │     └─happy-bmo ext4              1.0                            2169f641-3fc0-4b11-adda-2ef58ff9cc23      1.5G     5% /mnt/bmo
    ├─nbd0p2          LVM2_member       LVM2 001                       ELf5O8-UJCr-SUHD-JXza-67qJ-RMeI-ZQT3Mh                
    │ └─joyjoy-ren    ext3              1.0                            4aeb7302-7cca-4f85-bb68-701d8cb59288    238.4M     8% /mnt/ren
    ├─nbd0p3                                                                                                                 
    └─nbd0p5          vfat              FAT32                          F1E7-2C1C                                 1.4G     3% /mnt/nbd0p5


Dodaj razlago PV, VG in LV
Dodaj opis ogleda MBR
Dodaj ukaze za odklop
