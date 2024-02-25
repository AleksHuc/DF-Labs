# 3. Vaja: Analiza diskov z GNU/Linux

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

Sedaj poženemo `radare2` nad želenim diskom, na primer `/dev/sda`, vpišemo niz `px 2042` in izpišemo prvi 512B našega diska. Prvi stolpec `- offset -` prikazuje odmik o začetka diska v šestnajstiški obliki, potem sledijo stolpci ` 0 1  2 3  4 5  6 7  8 9  A B  C D  E F`, ki prikazujejo zapisane podatke ob določenih odmikih v šestnajstiški obliki in zadnji stolpec `0123456789ABCDEF` prikazuje zapisane podatke ob določenih odmikih z [ASCII znaki](https://en.wikipedia.org/wiki/ASCII). Prvi 446B predstavlja sistemski nalagalnik, potem sledi 64B tabele razdelkov (od `0x000001be` naprej, vidimo, da imamo določena dva razdelka) in na koncu 2B predstavljata še veljavni podpis MBR (od `0x000001fe` naprej, `55aa`).

    r2 /dev/sda
    px 2042

    - offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
    0x00000000  eb63 9010 8ed0 bc00 b0b8 0000 8ed8 8ec0  .c..............
    0x00000010  fbbe 007c bf00 06b9 0002 f3a4 ea21 0600  ...|.........!..
    0x00000020  00be be07 3804 750b 83c6 1081 fefe 0775  ....8.u........u
    0x00000030  f3eb 16b4 02b0 01bb 007c b280 8a74 018b  .........|...t..
    0x00000040  4c02 cd13 ea00 7c00 00eb fe00 0000 0000  L.....|.........
    0x00000050  0000 0000 0000 0000 0000 0080 0100 0000  ................
    0x00000060  0000 0000 fffa 9090 f6c2 8074 05f6 c270  ...........t...p
    0x00000070  7402 b280 ea79 7c00 0031 c08e d88e d0bc  t....y|..1......
    0x00000080  0020 fba0 647c 3cff 7402 88c2 52be 807d  . ..d|<.t...R..}
    0x00000090  e817 01be 057c b441 bbaa 55cd 135a 5272  .....|.A..U..ZRr
    0x000000a0  3d81 fb55 aa75 3783 e101 7432 31c0 8944  =..U.u7...t21..D
    0x000000b0  0440 8844 ff89 4402 c704 1000 668b 1e5c  .@.D..D.....f..\
    0x000000c0  7c66 895c 0866 8b1e 607c 6689 5c0c c744  |f.\.f..`|f.\..D
    0x000000d0  0600 70b4 42cd 1372 05bb 0070 eb76 b408  ..p.B..r...p.v..
    0x000000e0  cd13 730d 5a84 d20f 83d8 00be 8b7d e982  ..s.Z........}..
    0x000000f0  0066 0fb6 c688 64ff 4066 8944 040f b6d1  .f....d.@f.D....
    0x00000100  c1e2 0288 e888 f440 8944 080f b6c2 c0e8  .......@.D......
    0x00000110  0266 8904 66a1 607c 6609 c075 4e66 a15c  .f..f.`|f..uNf.\
    0x00000120  7c66 31d2 66f7 3488 d131 d266 f774 043b  |f1.f.4..1.f.t.;
    0x00000130  4408 7d37 fec1 88c5 30c0 c1e8 0208 c188  D.}7....0.......
    0x00000140  d05a 88c6 bb00 708e c331 dbb8 0102 cd13  .Z....p..1......
    0x00000150  721e 8cc3 601e b900 018e db31 f6bf 0080  r...`......1....
    0x00000160  8ec6 fcf3 a51f 61ff 265a 7cbe 867d eb03  ......a.&Z|..}..
    0x00000170  be95 7de8 3400 be9a 7de8 2e00 cd18 ebfe  ..}.4...}.......
    0x00000180  4752 5542 2000 4765 6f6d 0048 6172 6420  GRUB .Geom.Hard 
    0x00000190  4469 736b 0052 6561 6400 2045 7272 6f72  Disk.Read. Error
    0x000001a0  0d0a 00bb 0100 b40e cd10 ac3c 0075 f4c3  ...........<.u..
    0x000001b0  0000 0000 0000 0000 023b b8a6 0000 8004  .........;......
    0x000001c0  0104 83fe c2ff 0008 0000 0070 610c 00fe  ...........pa...
    0x000001d0  c2ff 05fe c2ff fe7f 610c 0278 1e00 0000  ........a..x....
    0x000001e0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000001f0  0000 0000 0000 0000 0000 0000 0000 55aa  ..............U.
    0x00000200  5256 be1b 81e8 3901 5ebf f481 668b 2d83  RV....9.^...f.-.
    0x00000210  7d08 000f 84e2 0080 7cff 0074 4666 8b1d  }.......|..tFf..
    0x00000220  668b 4d04 6631 c0b0 7f39 4508 7f03 8b45  f.M.f1...9E....E
    0x00000230  0829 4508 6601 0566 8355 0400 c704 1000  .)E.f..f.U......
    0x00000240  8944 0266 895c 0866 894c 0cc7 4406 0070  .D.f.\.f.L..D..p
    0x00000250  50c7 4404 0000 b442 cd13 0f82 af00 bb00  P.D....B........
    0x00000260  70eb 6666 8b45 0466 09c0 0f85 9700 668b  p.ff.E.f......f.
    0x00000270  0566 31d2 66f7 3488 540a 6631 d266 f774  .f1.f.4.T.f1.f.t
    0x00000280  0488 540b 8944 0c3b 4408 7d79 8b04 2a44  ..T..D.;D.}y..*D
    0x00000290  0a39 4508 7f03 8b45 0829 4508 6601 0566  .9E....E.)E.f..f
    0x000002a0  8355 0400 8a54 0dc0 e206 8a4c 0afe c108  .U...T.....L....
    0x000002b0  d18a 6c0c 5a52 8a74 0b50 bb00 708e c331  ..l.ZR.t.P..p..1
    0x000002c0  dbb4 02cd 1372 468c c38e 450a 58c1 e005  .....rF...E.X...
    0x000002d0  0145 0a60 1ec1 e003 89c1 31ff 31f6 8edb  .E.`......1.1...
    0x000002e0  fcf3 a51f be23 81e8 5700 6183 7d08 000f  .....#..W.a.}...
    0x000002f0  8524 ff83 ef0c e916 ffbe 2581 e842 005a  .$........%..B.Z
    0x00000300  ea00 8200 00be 2881 e836 00eb 06be 2d81  ......(..6....-.
    0x00000310  e82e 00be 3281 e828 00eb fe6c 6f61 6469  ....2..(...loadi
    0x00000320  6e67 002e 000d 0a00 4765 6f6d 0052 6561  ng......Geom.Rea
    0x00000330  6400 2045 7272 6f72 00bb 0100 b40e cd10  d. Error........
    0x00000340  468a 043c 0075 f2c3 0000 0000 0000 0000  F..<.u..........
    0x00000350  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000360  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000370  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000380  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000390  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000003a0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000003b0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000003c0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000003d0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000003e0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000003f0  0000 0000 0200 0000 0000 0000 6900 2008  ............i. .
    0x00000400  ea1c 8200 0000 0000 e55a 0000 bcb2 0000  .........Z......
    0x00000410  fb6b 0000 5d07 0000 ffff ff00 fa31 c08e  .k..]........1..
    0x00000420  d88e d08e c066 bdf0 1f00 0066 89ec fb88  .....f.....f....
    0x00000430  161b 82cd 1366 e897 0000 00fc e867 0600  .....f.......g..
    0x00000440  008b 1508 8200 0081 c2c3 0300 008b 0d10  ................
    0x00000450  8200 008d 055d 8900 00fc e82c 0300 00e9  .....].....,....
    0x00000460  5607 0000 f0ff 0700 eb16 8db4 2600 0000  V...........&...
    0x00000470  008d b426 0000 0000 8db4 2600 0000 0090  ...&......&.....
    0x00000480  0000 0000 0000 0000 ffff 0000 009a cf00  ................
    0x00000490  ffff 0000 0092 cf00 ffff 0000 009e 0000  ................
    0x000004a0  ffff 0000 0092 0000 eb16 8db4 2600 0000  ............&...
    0x000004b0  008d b426 0000 0000 8db4 2600 0000 0090  ...&......&.....
    0x000004c0  2700 8082 0000 0004 0000 0000 0000 0000  '...............
    0x000004d0  0000 fa31 c08e d866 0f01 16c0 820f 20c0  ...1...f...... .
    0x000004e0  6683 c801 0f22 c066 eaef 8200 0008 0066  f....".f.......f
    0x000004f0  b810 008e d88e c08e e08e e88e d08b 0424  ...............$
    0x00000500  a3f0 1f00 00a1 6482 0000 89c4 89c5 a1f0  ......d.........
    0x00000510  1f00 0089 0424 31c0 0f01 0dc6 8200 000f  .....$1.........
    0x00000520  011d cc82 0000 c30f 0115 c082 0000 0f01  ................
    0x00000530  0dcc 8200 000f 011d c682 0000 89e0 a364  ...............d
    0x00000540  8200 008b 0424 a3f0 1f00 00b8 f01f 0000  .....$..........
    0x00000550  89c4 89c5 66b8 2000 8ed8 8ec0 8ee0 8ee8  ....f. .........
    0x00000560  8ed0 ea69 8300 0018 000f 20c0 6683 e0fe  ...i...... .f...
    0x00000570  0f22 c066 ea7b 8300 0000 0066 31c0 8ed8  .".f.{.....f1...
    0x00000580  8ec0 8ee0 8ee8 8ed0 fb66 c355 89e5 5756  .........f.U..WV
    0x00000590  5389 c389 cf31 c031 f685 d278 290f b60c  S....1.1...x)...
    0x000005a0  1384 c974 100f b689 0002 1000 8a8c 0800  ...t............
    0x000005b0  0010 0031 ce01 f83d fe00 0000 7e05 2dff  ...1...=....~.-.
    0x000005c0  0000 004a ebd3 89f0 5b5e 5f5d c384 d274  ...J....[^_]...t
    0x000005d0  2084 c074 1c0f b6c0 0fb6 8800 0210 000f   ..t............
    0x000005e0  b6d2 0fb6 8200 0210 008a 8401 0000 1000  ................
    0x000005f0  c331 c0c3 5589 e557 5653 83ec 3c89 45d8  .1..U..WVS..<.E.
    0x00000600  8955 e489 4dd4 31c0 3b45 d87d 0ec7 0485  .U..M.1.;E.}....
    0x00000610  0003 1000 ffff ffff 40eb ed31 c03b 45e4  ........@..1.;E.
    0x00000620  7d0a 8b7d d4c6 0407 0040 ebf1 8b45 e440  }..}.....@...E.@
    0x00000630  8945 dc31 c9c7 45e0 0000 0000 8b5d e039  .E.1..E......].9
    0x00000640  5dd8 0f8e cb00 0000 31db 395d e47e 0d80  ].......1.9].~..
    0x00000650  bc0b 0010 1000 0075 0d43 ebee 7508 ff45  .......u.C..u..E
    0x00000660  e003 4ddc ebd6 8b45 e089 1c85 0003 1000  ..M....E........
    0x00000670  0fb6 840b 0010 1000 0fb6 9000 0210 00b8  ................
    0x00000680  ff00 0000 29d0 31f6 8db9 0010 1000 897d  ....).1........}
    0x00000690  d00f b690 0000 1000 3975 e47c 1f89 4dc8  ........9u.|..M.
    0x000006a0  8b45 d08d 3c30 0fb6 0789 55cc e81c ffff  .E..<0....U.....
    0x000006b0  ff88 0746 8b4d c88b 55cc ebdc 8b45 e040  ...F.M..U....E.@
    0x000006c0  8945 d08b 45dc 8d3c 018b 75d0 3975 d874  .E..E..<..u.9u.t
    0x000006d0  8d31 f60f b694 3b00 1010 008d 8700 1010  .1....;.........
    0x000006e0  0089 45cc 3975 e47c 220f b684 0e00 1010  ..E.9u.|".......
    0x000006f0  0089 4dc4 8955 c8e8 d1fe ffff 8b55 cc30  ..M..U.......U.0
    0x00000700  0432 468b 4dc4 8b55 c8eb d9ff 45d0 037d  .2F.M..U....E..}
    0x00000710  dceb b68b 75d8 4e8b 45e4 f7d0 8945 d88b  ....u.N.E....E..
    0x00000720  5ddc 0faf de03 5de4 85f6 7857 8b3c b500  ].....]...xW.<..
    0x00000730  0310 0083 ffff 7445 89d8 2b45 e489 45dc  ......tE..+E..E.
    0x00000740  c645 e000 31c9 394d e47e 238b 45d4 0fb6  .E..1.9M.~#.E...
    0x00000750  1408 8b45 dc0f b684 0100 1010 0089 4dd0  ...E..........M.
    0x00000760  e868 feff ff30 45e0 8b4d d041 ebd8 8a45  .h...0E..M.A...E
    0x00000770  e032 8300 1010 008b 4dd4 8804 394e 035d  .2......M...9N.]
    0x00000780  d8eb a583 c43c 5b5e 5f5d c355 89e5 5756  .....<[^_].U..WV
    0x00000790  5383 ec4c 8945 bc89 55dc 894d e085 c90f  S..L.E..U..M....
    0x000007a0  84fb 0200 0001 d089 45d4 8d4c 08ff 89c8  ........E..L....
    0x000007b0  3b45 d472 0880 3800 7503 48eb f329 c1bb  ;E.r..8.u.H..)..
    0x000007c0  0200 0000 8b45 e099 f7fb 39c1 0f8f ce02  .....E....9.....
    0x000007d0  0000 c605 0002 1000 00ba 0000 1000 b001  ................
    0x000007e0  8802 8882 ff00 0000 0fb6 c888 9100 0210  ................
    0x000007f0  0084 c079 088d 0409 83f0 1deb 02d1 e042  ...y...........B

Z orodjem [`gdisk`](https://linux.die.net/man/8/gdisk) lahko spremenimo ime razdelka in ga nato poiščemo z orodjem `radare2`. To naredimo tako, da najprej izpišemo razdelke na disku z `p`, nato zaženemo urejanje razdelkov z `c`, izberemo prvi razdelek z `1`, vpišemo novo ime razdelka `DF Linux`, zapišemo spremembe na disk z `w` in potrdimo zapis z `Y`. Vidimo, da smo s tem vpeljali GPT tabelo z razdelki in zaščitili obstoječ MBR, saj MBR ne ponuja poljubnega poimenovanja razdelkov. Novo ime našega razdelka sedaj najdemo na odmiku `0x00000438`.

    gdisk /dev/sda

    Partition table scan:
      MBR: MBR only
      BSD: not present
      APM: not present
      GPT: not present

    p

    Disk /dev/sda: 209715200 sectors, 100.0 GiB
    Model: VBOX HARDDISK   
    Sector size (logical/physical): 512/512 bytes
    Disk identifier (GUID): 83372426-9E64-49CF-805D-9F6AF5CB8B15
    Partition table holds up to 128 entries
    Main partition table begins at sector 2 and ends at sector 33
    First usable sector is 34, last usable sector is 209715166
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 6077 sectors (3.0 MiB)

    Number  Start (sector)    End (sector)  Size       Code  Name
    1            2048       207714303   99.0 GiB    8300  Linux filesystem
    5       207716352       209713151   975.0 MiB   8200  Linux swap

    c

    1

    DF Linux

    w

    Y

    gdisk /dev/sda

    Partition table scan:
      MBR: protective
      BSD: not present
      APM: not present
      GPT: present

    p

    Disk /dev/sda: 209715200 sectors, 100.0 GiB
    Model: VBOX HARDDISK   
    Sector size (logical/physical): 512/512 bytes
    Disk identifier (GUID): 3057242F-46B2-4801-81B5-9422067A9E53
    Partition table holds up to 128 entries
    Main partition table begins at sector 2 and ends at sector 33
    First usable sector is 34, last usable sector is 209715166
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 6077 sectors (3.0 MiB)

    Number  Start (sector)    End (sector)  Size       Code  Name
    1            2048       207714303   99.0 GiB    8300  DF Linux
    5       207716352       209713151   975.0 MiB   8200  Linux swap

    q

    r2 /dev/sda
    px 2042

    - offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
    0x00000000  eb63 9010 8ed0 bc00 b0b8 0000 8ed8 8ec0  .c..............
    0x00000010  fbbe 007c bf00 06b9 0002 f3a4 ea21 0600  ...|.........!..
    0x00000020  00be be07 3804 750b 83c6 1081 fefe 0775  ....8.u........u
    0x00000030  f3eb 16b4 02b0 01bb 007c b280 8a74 018b  .........|...t..
    0x00000040  4c02 cd13 ea00 7c00 00eb fe00 0000 0000  L.....|.........
    0x00000050  0000 0000 0000 0000 0000 0080 0100 0000  ................
    0x00000060  0000 0000 fffa 9090 f6c2 8074 05f6 c270  ...........t...p
    0x00000070  7402 b280 ea79 7c00 0031 c08e d88e d0bc  t....y|..1......
    0x00000080  0020 fba0 647c 3cff 7402 88c2 52be 807d  . ..d|<.t...R..}
    0x00000090  e817 01be 057c b441 bbaa 55cd 135a 5272  .....|.A..U..ZRr
    0x000000a0  3d81 fb55 aa75 3783 e101 7432 31c0 8944  =..U.u7...t21..D
    0x000000b0  0440 8844 ff89 4402 c704 1000 668b 1e5c  .@.D..D.....f..\
    0x000000c0  7c66 895c 0866 8b1e 607c 6689 5c0c c744  |f.\.f..`|f.\..D
    0x000000d0  0600 70b4 42cd 1372 05bb 0070 eb76 b408  ..p.B..r...p.v..
    0x000000e0  cd13 730d 5a84 d20f 83d8 00be 8b7d e982  ..s.Z........}..
    0x000000f0  0066 0fb6 c688 64ff 4066 8944 040f b6d1  .f....d.@f.D....
    0x00000100  c1e2 0288 e888 f440 8944 080f b6c2 c0e8  .......@.D......
    0x00000110  0266 8904 66a1 607c 6609 c075 4e66 a15c  .f..f.`|f..uNf.\
    0x00000120  7c66 31d2 66f7 3488 d131 d266 f774 043b  |f1.f.4..1.f.t.;
    0x00000130  4408 7d37 fec1 88c5 30c0 c1e8 0208 c188  D.}7....0.......
    0x00000140  d05a 88c6 bb00 708e c331 dbb8 0102 cd13  .Z....p..1......
    0x00000150  721e 8cc3 601e b900 018e db31 f6bf 0080  r...`......1....
    0x00000160  8ec6 fcf3 a51f 61ff 265a 7cbe 867d eb03  ......a.&Z|..}..
    0x00000170  be95 7de8 3400 be9a 7de8 2e00 cd18 ebfe  ..}.4...}.......
    0x00000180  4752 5542 2000 4765 6f6d 0048 6172 6420  GRUB .Geom.Hard 
    0x00000190  4469 736b 0052 6561 6400 2045 7272 6f72  Disk.Read. Error
    0x000001a0  0d0a 00bb 0100 b40e cd10 ac3c 0075 f4c3  ...........<.u..
    0x000001b0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000001c0  0200 eeff ffff 0100 0000 ffff 7f0c 0000  ................
    0x000001d0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000001e0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000001f0  0000 0000 0000 0000 0000 0000 0000 55aa  ..............U.
    0x00000200  4546 4920 5041 5254 0000 0100 5c00 0000  EFI PART....\...
    0x00000210  5d25 996d 0000 0000 0100 0000 0000 0000  ]%.m............
    0x00000220  ffff 7f0c 0000 0000 2200 0000 0000 0000  ........".......
    0x00000230  deff 7f0c 0000 0000 2f24 5730 b246 0148  ......../$W0.F.H
    0x00000240  81b5 9422 067a 9e53 0200 0000 0000 0000  ...".z.S........
    0x00000250  8000 0000 8000 0000 7ca2 0c7b 0000 0000  ........|..{....
    0x00000260  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000270  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000280  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000290  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000002a0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000002b0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000002c0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000002d0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000002e0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000002f0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000300  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000310  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000320  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000330  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000340  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000350  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000360  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000370  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000380  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000390  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000003a0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000003b0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000003c0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000003d0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000003e0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000003f0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000400  af3d c60f 8384 7247 8e79 3d69 d847 7de4  .=....rG.y=i.G}.
    0x00000410  5d63 06a2 4adf 8049 8863 6fbd d3f8 d921  ]c..J..I.co....!
    0x00000420  0008 0000 0000 0000 ff77 610c 0000 0000  .........wa.....
    0x00000430  0000 0000 0000 0000 4400 4600 2000 4c00  ........D.F. .L.
    0x00000440  6900 6e00 7500 7800 0000 0000 0000 0000  i.n.u.x.........
    0x00000450  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000460  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000470  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000480  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000490  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000004a0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000004b0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000004c0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000004d0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000004e0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000004f0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000500  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000510  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000520  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000530  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000540  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000550  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000560  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000570  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000580  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000590  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000005a0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000005b0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000005c0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000005d0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000005e0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000005f0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000600  6dfd 5706 aba4 c443 84e5 0933 c84b 4f4f  m.W....C...3.KOO
    0x00000610  dd4b 5801 5069 db43 8293 ff5d d40f 0c4c  .KX.Pi.C...]...L
    0x00000620  0080 610c 0000 0000 fff7 7f0c 0000 0000  ..a.............
    0x00000630  0000 0000 0000 0000 4c00 6900 6e00 7500  ........L.i.n.u.
    0x00000640  7800 2000 7300 7700 6100 7000 0000 0000  x. .s.w.a.p.....
    0x00000650  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000660  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000670  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000680  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000690  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000006a0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000006b0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000006c0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000006d0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000006e0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000006f0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000700  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000710  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000720  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000730  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000740  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000750  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000760  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000770  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000780  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x00000790  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000007a0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000007b0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000007c0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000007d0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000007e0  0000 0000 0000 0000 0000 0000 0000 0000  ................
    0x000007f0  0000 0000 0000 0000 0000 0000 0000 0000  ................

Sedaj moramo dodati še nov razdelek in popraviti sistemski nalagalnik, da se bo naš operacijski sistem znal zagnati z GPT tabele razdelkov. Poženemo orodje `gdisk` na disku operacijskega sistema, izberemo možnost `n` za ustvarjanje novega razdelka in izberemo vrednost `2` za številko razdelka. Za prvi sektor razdelka izberemo sektor 34, za zadnji sektor razdelka izberemo sektor 2047 nato še izberemo kodo za `BIOS boot` razdelek, ki je `ef02` ali pa ga poiščemo z izbiro možnosti `l`. Zapišemo spremembe na disk z možnostjo `w` in potrdimo z `Y`. Na koncu še ponovno namestimo sistemski nalagalnik z ukazom [`grub-install`](https://linux.die.net/man/8/grub-install).

    gdisk /dev/sda

    GPT fdisk (gdisk) version 1.0.6

    Partition table scan:
    MBR: protective
    BSD: not present
    APM: not present
    GPT: present

    Found valid GPT with protective MBR; using GPT.

    p

    Disk /dev/sda: 209715200 sectors, 100.0 GiB
    Model: VBOX HARDDISK   
    Sector size (logical/physical): 512/512 bytes
    Disk identifier (GUID): 7A51BD7B-5A21-4AA2-8908-0C3E3A246C2C
    Partition table holds up to 128 entries
    Main partition table begins at sector 2 and ends at sector 33
    First usable sector is 34, last usable sector is 209715166
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 6077 sectors (3.0 MiB)

    Number  Start (sector)    End (sector)  Size       Code  Name
    1            2048       207714303   99.0 GiB    8300  DF Linux
    5       207716352       209713151   975.0 MiB   8200  Linux swap

    n

    Partition number (2-128, default 2): 2
    First sector (34-209715166, default = 207714304) or {+-}size{KMGTP}: 34
    Last sector (34-2047, default = 2047) or {+-}size{KMGTP}: 2047
    Current type is 8300 (Linux filesystem)
    Hex code or GUID (L to show codes, Enter = 8300): ef02
    Changed type of partition to 'BIOS boot partition'

    w

    Y

    grub-install /dev/sda

    Installing for i386-pc platform.
    Installation finished. No error reported.

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

Sedaj naredimo kopije obeh navideznih diskov in zgoščenke z ukazom [`cat`](https://www.man7.org/linux/man-pages/man1/cat.1.html) ter izračunamo integritetno vrednost diska z ukazom [`sha512sum`](https://man7.org/linux/man-pages/man1/sha512sum.1.html).

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

Nato ponovno poženemo naš navidezni računalnik. Začnemo s priklopom slike zgoščenke, ki je ne moremo spreminjati. Podatki so zapisani z namenskim [datotečnim sistemom ISO9660](https://en.wikipedia.org/wiki/ISO_9660). Za priključitev zgoščenke uporabimo ukaz [`mount`](https://man7.org/linux/man-pages/man8/mount.8.html) ter preverimo katere nove datoteke so nam sedaj dostopne.

    mkdir /mnt/sr0
    mount sr0_copy.img /mnt/sr0
    
    mount: /mnt/sr0: WARNING: source write-protected, mounted read-only.

    ls /mnt/sr0

    cute-cat-l.jpg	kitty1.jpeg  kitty2.jpeg  kitty3.jpeg  kitty4.jpg

ISO9660 omejuje dolžino imen, ki jih imajo lahko datoteke, da zaobidemo to omejitev se privzeto uporablja [Joliet razširitev datotečnega sistema](https://en.wikipedia.org/wiki/ISO_9660#Joliet). Sedaj bomo najprej odklopili trenutno priklopljeno zgoščenko z ukazom [`umount`](https://man7.org/linux/man-pages/man8/umount.8.html) in jo nato ponovno priklopili z ukazom `mount` z uporabo dodatnih možnost z zastavico `-o`. Uporabimo zastavico `nojoliet`, da omogočimo priključitev brez Joliet razširitve in zastavico `unhide`, da prikažemo skrite datoteke. Sedaj preverimo ali lahko dostopamo še do kakšnih datotek.

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

Sedaj priklopimo disk z ukazom `qemu-nbd` z zastavico `-c` in izpišemo trenutno priklopljene diske z ukazom `lsblk`. Po potrebi lahko odklopimo disk z ukazom `qemu-nbd` z zastavico `-d`.

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

Za priključitev drugega diska pa uporabimo orodje `kpartx` z zastavico `-a` in izpišemo trenutno priklopljene diske z ukazom `lsblk`.  Po potrebi lahko odklopimo disk z ukazom `kpartx` z zastavico `-d`.

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

Avtomatsko sistem ni zaznal nobenega RAID polja, zato ga poskusimo z uporabo orodja `mdadm` z uporabo zastavic `--assemble` in `--scan` ter preverimo RAID polje, ki smo ga pognali z zastavico `--detail`. Pononvno preverimo priključene disk z ukazom `lsblk`.

    mdadm --assemble --scan

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

Za delo z LVM logičnimi diski potrebujemo orodje [`lvm2`](https://linux.die.net/man/8/lvm), ki ga namestimo z upravljalcem paketov našega operacijskega sistema. Nato preverimo priključene disk z ukazom `lsblk` ali je prišlo do kakšne spremembe.

    apt update
    apt install lvm2

    lsblk

Podatke o `LVM` logičnih diskih pridobimo z ukazom [`pvdisplay`](https://man7.org/linux/man-pages/man8/pvdisplay.8.html) nam prikaže fizične diske in razdelke (`Physical Volume`), ki so razporejeni v skupine fizičnih diskov (`Volume Group`), ki jih nam prikaže ukaz [`vgdisplay`](https://man7.org/linux/man-pages/man8/vgdisplay.8.html) in znotraj skupin lahko določimo logične diske (`Logical Volume`), ki jih nam prikaže ukaz [`lvdisplay`](https://man7.org/linux/man-pages/man8/lvdisplay.8.html).

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

Vidimo, da imamo 3 razdelke (`Physical Volume`): `/dev/nbd0p2`, `/dev/mapper/loop1p5` in `/dev/md0p2`, ki so vključeni v logične diske.

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

Vidimo tudi, da so razdelki razdeljeni v dve skupini (`Volume Group`), in sicer razdelka `/dev/mapper/loop1p5` in `/dev/md0p2` sta člana skupine `happy` ter razdelek `/dev/nbd0p2` je član skupine `joyjoy`.

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

Prav tako vidimo znotraj skupine `joyjoy` ustvarjen en logični disk (`Logical Volume`) `ren` ter znotraj skupine `happy` imamo dva logična diska `stimpy` in `bmo`. Logični disk `bmo` zajema razdelek `md0p2` in del razdelka `loop1p5`, logični disk `stimpy` zajema del razdelka `loop1p5` in logični disk `ren` zajema razdelek `nbd0p2`. Vse logične diske lahko priklopimo na enkrat z ukazom [`vgchange`](https://man7.org/linux/man-pages/man8/vgchange.8.html) in zastavico `-a y` in preverimo priključene disk z ukazom `lsblk`.

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

Vse trenutno priključene diske, razdelke, datotčne sisteme, RAID polja ter LVM logične diske lahko prikažemo še z detaljnim izpisom ukaza `lsblk` z zastavico `-f`.

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
