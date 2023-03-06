# 2. Vaja: Diski in diskovni sistemi

## Navodila

1. Kako dostopamo do podatkov na slikah virtualnih diskov pod sistemom GNU/linux?

## Dodatne informacije

## Podrobna navodila

### Prenos navideznih diskov

S [spletne strani](https://polz.si/dsrf/) prenesemo naslednje datoteke:

- `furry.iso`
- `raid0_1.vdi`
- `raid0_2.vdi`

V operacijskih sistemih GNU/Linux lahko do diskov, ki so priklopljeni dostopamo preko imenika `/dev/NAME_OF_THE_DISK`, kjer je ponavadi ime diska `sdX`, kjer `X` zamenjamo z zaporedno črko abecede.

    cd /dev
    ls /dev
    cd /dev/sda

Na katero pot v datotečnem sistemu so posamezni diski (`sdX`) in razdelki (`sdXN`) priklopljeni ugotovimo preko imenika `/dev/NAME_OF_THE_DISK_OR_PARTITION/by-path`.

    ls -al /dev/sda/by-path
    cd /dev/sda/by-path

Priklopljene diske (`sdX`) in razdelke (`sdXN`) lahko izpišemo tudi z ukazom `lsblk`.

    lsblk

Radare2 pregled MBR

V operacijskih sistemih GNU/linux za delo z diski uporabljamo orodja kot so: `mount`, `fdisk`, `gdisk`, `libguestfs`, `qemu-img`, `qemu-nbd`...  Razdelki se predstavijo operacijskemu sistemu kot datoteke. Podatke znotraj razdelkov so urejene v drevesno strukturo, ki predstavlja datotečni sistem, ki ustvari svojo glavo na začetku razdelka in nato sledijo datoteke. 

