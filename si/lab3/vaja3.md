# 2. Vaja: Diski in diskovni sistemi

## Navodila

1. Kako dostopamo do podatkov na slikah virtualnih diskov pod sistemom GNU/linux?

## Dodatne informacije

## Podrobna navodila

### Pregled MBR

Radare2 pregled MBR

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

Sedaj zaustavimo naš navidezni računalnik ter odstranimo pod `Krmilnik: IDE` odstranimo zgoščenko `furry.iso` in pod `Krmilnik:SATA` odstranimo navidezna diska `raid0_1.vdi` in `raid0_2.vdi`, tako da izberemo vsak nosile in spodaj levo pritisnemo na gumb z disketo in rdečim križem.

