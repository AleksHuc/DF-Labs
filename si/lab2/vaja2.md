# 2. Vaja: Diski in diskovni sistemi

## Navodila

1. Kako podatke shranjujemo na računalnikih in kako so urejeni?

## Dodatne informacije

## Podrobna navodila

### Kako so sestavljeni diski

V splošnem je [trdi disk](https://en.wikipedia.org/wiki/Hard_disk_drive) sestavljen iz več vrtečih plošč in bralno/pisalnih glav, ki se premikajo po obodu in berejo/zapisujejo binarne podatke v naprej določene sektorje s fiksno velikostjo.

**MFM diski**

Prvi diski na IBM kompatibilnih osebnih računalnikih so bili na računalnik priključeni tako, da je računalnik povsem nadziral disk. Računalnik je torej povedal, kam naj se premakne glava, preštel sektorje in prebral podatke. Za povezovanje diskov se je uporabljal na primer vmesnik [ST506/ST412](https://en.wikipedia.org/wiki/ST506/ST412#Connector_pinouts) ter podatki so se zapisovalni na disk s kodiranjem osnovanim na modificirani frekvenčni modulaciji ([Modified frequency modulation - MFM](https://en.wikipedia.org/wiki/Modified_frequency_modulation)).

**CHS naslavljanje**

Zaradi enostavnosti je bilo potrebno za branje ali pisanje podatkov podati sled (Cyllinder), glavo na disku (Head) in sektor (Sector), ki kažejo na željen sektor s podatki. Temo procesu rečemo CHS naslavljanje ([CHS Addressing](https://en.wikipedia.org/wiki/Cylinder-head-sector)), danes pa naslavljamo podatke na disku tako da podamo samo številko sektorja na katerem so zapisani.

CHS naslavljanje uporablja 24 bitov dolge zapise za naslavljanje posameznega sektorja na disku:
- C: sled, ki ima vrednosti med 0 in 1023 (10 bitov). Sled je koncentrični krog na diskovni glavi.
- H: glava, Ki ima vrednosti med 0 in 254 (8 bitov). Glava nam določi stran in število diskovnih plošč v disku. 
- S: sektor, ki ima vrednosti med 1 in 63 (6 bitov). Vsaka sled je razdeljene na sektorje.

Prvi sektor je namenjen za sistemski nalagalnik in tabelo razdelkov. Sektor je imel privzeto velikost $512B$, novejši diski pa imajo sektorje velikosti $4096B$. Če predpostavimo velikost sektorja $512B$, potem lahko z CHS naslovimo maksimalno $1024*255*63*512 = 8,422,686,720B$.

**IDE/ATA diski**

Ker dolg kabel z veliko paralelnimi žicami med bralno/pisalno glavo in računalnikom povzroča preglavice, so proizvajalci diskov kmalu premaknili večino elektronike z razširitvene kartice na sam disk. Tako so nastali [Integrated Drive Electronics - IDE diski](https://en.wikipedia.org/wiki/Parallel_ATA#IDE_and_ATA-1). Vmesnik je skoraj enak kot razširitveno vodilo na prvih IBM PC-ih.

Z leti so standard razširili, nastal je [(IBM PC) AT Attachment - ATA](https://en.wikipedia.org/wiki/Parallel_ATA). Tega so nato razširili tako, da so dodali [podporo za različne naprave](https://en.wikipedia.org/wiki/ATA_Packet_Interface).

**SATA diski**

Ker je hitrosti prenosa na kablih, ki imajo veliko žic, težko višati, so proizvajalci diskov ustvarili standard, ki je, kar se tiče programske opreme, enak ATA, dejanski vmesnik pa je [serijski SATA](https://en.wikipedia.org/wiki/SATA).

**SCSI diski**

Vzporedno z ATA so se razvili tudi vmesnik, namenjen prav napravam za shranjevanje podatkov - [Small Computer Storage Interface](https://en.wikipedia.org/wiki/SCSI). Kasneje diski s tem vmesnikom niso bili nič bolj zapleteni, so pa ostali dražji od ATA diskov, saj so ciljali drugačno tržišče (strežnike).

**Flash - SSD**

Zadnjih nekaj let večina naprav vsebuje [pomnilnik flash (Solid State Drive)](https://en.wikipedia.org/wiki/Solid-state_drive). Sestavljeni so iz čipov, ki hranijo veliko število celic, ki lahko hranijo binarne vrednosti in vsaka celica ima svoj naslov. Pri tem pomnilniku je podatke, ko so enkrat zapisani, težko popravljati. Posamezne celice (bita), ki je nastavljena na 0, ne moremo zlahka nastaviti nazaj na 1. Edini način, da to storimo, je, da na 1 nastavimo cel blok celic.

Velikost bloka je tipično nekaj MiB. To pomeni, da bi pri naivni rešitvi ob vsaki spremembi sektorja (512B) skopirati celoten blok brez popravljenega sektorja, torej nekaj tisočkrat več podatkov, drugam, nato pa zapisati nov blok.

Ker je takšen pristop neučinkovit, so razvijalci jedra Linux že pred več kot desetletjem razvili nekaj datotečnih sistemov, ki so učinkovitejši. Namesto, da bi podatke popravljali na istem mestu, jih vedno samo zapisujejo, datotečni sistem pa vsebuje preslikovalno tabelo, tako da je vedno mogoče hitro najti zadnjo različico vsakega sektorja. Pri takšnih datotečnih sistemih je pomembna tudi operacija brisanja, saj lahko bloke, ki vsebujejo samo sektorje izbrisanih datotek, ponovno uporabimo. Takšne datotečne sisteme so v tistem času uporabljali na vgrajenih napravah, kjer so prednosti pomnilnikov flash, kot sta majhna masa ter odpornost na udarce, odtehtali njihovo visoko ceno glede na kapaciteto.

S padanjem cene so pomnilniki flash postali zanimivi tudi za ostale uporabnike, predvsem v obliki prenosnih naprav za izmenjavo podatkov. Tako so USB ključki začeli nadomeščati diskete in zapisljive optične diske (CD-RW, DVD-RW). Seveda proizvajalci niso mogli pričakovati, da bi uporabniki zamenjali operacijski sistem samo zato, da bi lahko uporabili njihove naprave. Ker Windows ne podpirajo datotečnih sistemov, namenjenih pomnilnikom flash, se morajo tudi USB flash "ključki" obnašati, kot bi bili stari, vrteči se diski. To pomeni, da vsak ključek USB poleg pomnilnika flash vsebuje še mikroprocesor, ki skrbi za sledenje polnim in praznim blokom in skrbi za večino tega, za kar je prej skrbel gonilnik za datotečni sistem - torej za vzdrževanje preslikovalnih tabel, brisanje ne več zasedenih blokov in podobno.

Ko je cena pomnilnikov flash padla dovolj nizko, so postali konkurenčni tudi vrtljivim diskom v računalnikih. Na trgu so se pojavile SATA SSD naprave, ki se prav tako obnašajo kot navadni trdi diski, le da računalnik poleg ukazov za branje in pisanje sektorja "disku" lahko pošlje še ukaz za brisanje. Takšne pomnilniške naprave, tako kot ključke USB, lahko uporabljamo tudi z operacijskimi sistemi, ki ne podpirajo datotečnih sistemov, namenjenih pomnilnikom flash.

S stališča forenzike so tovrstni diski zanimivi, saj podatki, ki so bili "popravljeni" ali izbrisani, ostanejo na samih flash čipih, dokler se mikroprocesor ne odloči, da bo bloke, na katerih se nahajajo, ponovno uporabil. To se lahko zgodi kadar koli, na primer ob priklopu "diska" na napajanje, ko hočemo ustvariti forenzično kopijo. Za razliko od trdih diskov torej s priklopom naprave, na kateri bomo izvajali zajem, lahko že uničimo del dokazov.

Ker proizvajalci ne objavljajo, kako točno njihove naprave vzdržujejo sezname prostih blokov in preslikovalne tabele, je sestavljanje datotek iz podatkov, ki bi jih lahko pridobili iz čipov, težavno. Sami čipi tudi običajno niso zlahka dostopni, procesor pa lahko podatke ne samo razmeče po pomnilniku, temveč jih še šifrira. To pomeni, da se običajno zajem vseh podatkov ne izplača, zajamemo le tiste podatke, ki jih lahko dobimo, če se pretvarjamo, da imamo opravka z navadnim, vrtečim se diskom.

### Zagon računalnika

Prvi IBMovi osebni računalniki so imeli več možnosti za zagon:

- v ROM vgrajen BASIC.
- z diskete.
- s trdega diska, ki ni bil del standardne opreme.

Količini ROM in RAM sta bili omejeni - [40KiB za ROM](https://www.pcjs.org/machines/pcx86/ibm/5150/rom/), 128KiB za RAM. Na takšnem sistemu je jasno, da bo zagon čim bolj preprost. Princip je:

1. Izvedi program v ROM [začni na (naslovu 0xffff0; PC=0, CS=0xffff](https://en.wikipedia.org/wiki/Intel_8086#Segmentation).
2. Če na računalnik ni priklopljen noben disk ali disketa, zaženi [BASIC](https://en.wikipedia.org/wiki/IBM_BASIC).
3. Če ja na voljo disk, naloži 1. sektor z diska v RAM na [naslov 0x7c00](https://wiki.osdev.org/Memory_Map_(x86)) - to je zagonski program.
4. Preveri, ali je program veljaven (ima na koncu posebno vrednost).
5. Izvedi program.

**Basic Input Output System (BIOS)**

Prvi program, ki ga požene CPU je program BIOS:

1. Izvajati začne ukaze shranjene v [Basic Input Output System (BIOS)](https://en.wikipedia.org/wiki/BIOS) sistemu, ki se nahaja na ločenem pomnilniku na matični plošči, ki zazna in upravlja s strojno opremo ter preda izvajanje sistemskemu nalagalniku. Izvede se [Power-On Self-Test (POST)](https://en.wikipedia.org/wiki/Power-on_self-test) proces, ki zazna ter preveri strojno opremo, na primer procesor, pomnilnik, grafična kartica, trdi diski ter ostale vhodno/izhodno naprave.
2. Nato se izvedejo še BIOS razširitve (BIOS Extensions), ki omogočajo izvedbo ukazov shranjenih v BIOS pomnilnikih razširitvenih kartic za njihov zagon, na primer mrežne kartice, diskovni krmilniki, grafični pospeševalniki in ostale naprave.
3. BIOS prebere prvih 512B na izbranem nosilcu in jih zažene, omenjenemu programu nudi tudi možnost za dostop do nadaljnjih podatkov na napravi. Teh prvih 512B na nosilcu imenujemo [Master Boot Record (MBR)](https://en.wikipedia.org/wiki/Master_boot_record), ki v prvih 446B vsebuje sistemski nalagalnik, nato v naslednjih 64B [tabelo razdelkov - partition table](https://en.wikipedia.org/wiki/Master_boot_record#Disk_partitioning) in v zadnjih 2B še podpis, ki potrjuje veljavnost MBR. MBR se lahko nahaja na trdem disku, USB prenosnem pomnilniku, CD ali DVD nosilcu.
4. [Sistemski nalagalnik (Bootloader)](https://en.wikipedia.org/wiki/Bootloader) v MBR poskrbi za zagon operacijskega sistema. Ker sistemski nalagalnik za sodobni operacijski sistem potrebuje več prostora kot 446B, ga razdelimo v dva dela. 1. stopnja sistemskega nalagalnika se nahaja v MBR in poskrbi za zagon 2. stopnje sistemskega nalagalnika, ki se nahaja v eni od razdelkov na podatkovnem nosilcu.
5. Sistemski nalagalnik na 2. stopnji poskrbi za zagon operacijskega sistema, tako da požene [jedro (kernel)](https://en.wikipedia.org/wiki/Linux_kernel) z dodatnimi parametri ter [začetni navidezni disk (initial RAM disk - initrd or initramfs)](https://en.wikipedia.org/wiki/Initial_ramdisk) z začasnim datotečnim izhodiščnim datotečnim sistemom.
6. Nato se zažene [prvi program (init)](https://en.wikipedia.org/wiki/Init) na operacijskem sistemu po poskrbi za zagon in dobi procesorsko oznako 1.
7. Sedaj sledi zagon uporabniških programov, grafičnega okolja in drugih programov.

**Unified Extensible Firmware Interface (UEFI)**

Prvi program, ki ga pa požene sodobni CPU je pa program [UEFI](https://en.wikipedia.org/wiki/UEFI), ki nadomesti BIOS. Sam postopek zagona je podoben vendar UEFI ne pričakuje posebnega sektorja MBR za zagon operacijskega sistema, ampak sam zazna nameščene sistemske nalagalnike in operacijske sisteme, ter zažene tistega na katerega kažejo trenutne nastavitve. Sistemski nalagalniki se nahajajo v posebnih EFI razdelkih na disku in na v naprej določenih poteh `<EFI_SYSTEM_PARTITION>\EFI\BOOT\BOOT<MACHINE_TYPE_SHORT_NAME>.EFI`. Prav tako ker so EFI razdelki lahko poljubne velikosti, sistemski nalagalniki niso več omejeni po velikosti. UEFI sistemi prav tako podpirajo sodobnejšo ureditev razdelkov z [GUID tabelami razdelkov (GPT)](https://en.wikipedia.org/wiki/GUID_Partition_Table).

### Razdelki

Razdelki nam posamezne diske razdelijo na več logičnih enot, ki nam omogočajo namestitev enega ali več različnih operacijskih sistemov z več različnimi datotečnimi sistemi ter uporabo razdelkov za druge naloge, kot so zagon operacijskega sistema, varnostno kopiranje operacijskega sistema ...

Poznamo več formatov, ki nam določijo razdelke:

- [MBR/DOS disk label](https://en.wikipedia.org/wiki/Master_boot_record)
- [BSD disklabel](https://en.wikipedia.org/wiki/BSD_disklabel)
- [Apple Partition Map](https://en.wikipedia.org/wiki/Apple_Partition_Map)
- [GUID Partition Tabel](https://en.wikipedia.org/wiki/GUID_Partition_Table)

**MBR/DOS disk label**

MBR vsebuje rezerviran prostor za opis 4 razdelkov. Tem štirim razdelkom rečemo tudi primarni razdelki (primary partition). Vsak [16 bajtni vnos v tabeli razdelkov](https://en.wikipedia.org/wiki/Master_boot_record#Partition_table_entries) vsebuje začetek, konec, tip razdelka ter zastavice. Začetek in konec sta bila nekdaj zapisana v [formatu CHS](https://en.wikipedia.org/wiki/Cylinder-head-sector).

Ker se je že precej zgodaj izkazalo, da 4 razdelki niso vedno dovolj, so razvijalci dodali možnost, da se na začetek razdelka doda [še ena kopija MBR (Extended boot record- EBR)](https://en.wikipedia.org/wiki/Extended_boot_record) in v njej opiše dodatne 4 razdelke, kar nam da skupaj 16 možnih razdelkov.

Poleg tega se je izkazalo, da je 2^24^ sektorjev ($16777216*512B = 8589934592B = 8,58GB$) prav tako premalo. V prostor, ki je bil v vsakem razdelku rezerviran, so dodali 32-bitni začetek razdelka v [Logical Block Addressing (LBA)](https://en.wikipedia.org/wiki/Logical_block_addressing) formatu ter 32-bitno dolžino v številu sektorjev.

MBR tabelo razdelkov lahko upravljamo z več orodji: [`fdisk`](https://www.man7.org/linux/man-pages/man8/fdisk.8.html), [`gdisk`](https://linux.die.net/man/8/gdisk), [`sfdisk`](https://man7.org/linux/man-pages/man8/sfdisk.8.html)...

**GUID Partition Tabel (GPT)**

Ker je 16 razdelkov premalo in ker imajo sodobni diski lahko tudi več kot 2^32^ sektorjev, so se proizvajalci računalniške opreme malo po letu 2000 dogovorili za nov standard, ki omogoča več večjih razdelkov in za vsakega od njih več podatkov.

[GPT](https://en.wikipedia.org/wiki/GUID_Partition_Table) se nahaja takoj za 1. sektorjem. V 1. sektorju še vedno ostaja nekaj podobnega MBR, ki vsebuje en ogromen, ne povsem veljaven razdelek, ki je tam le zato, da stari programi ne bi prepisali nove tabele razdelkov. Poleg tega je na koncu diska z GPT kopija tabele razdelkov. Tako poskrbimo, da v primeru odpovedi enega sektorja na disku ne izgubimo vseh podatkov.

GPT glava vsebuje table za do 128 razdelkov. Vsak vnos v tabeli je dolžine 128B ter v [formatu LBA](https://en.wikipedia.org/wiki/Logical_block_addressing) in med drugim vsebuje 16B številko, ki je za vsak razdelek na svetu verjetno enolična - od tod tudi ime nove sheme. Tej številki rečemo Globally Unique IDentifier ali GUID. Od tod tudi ime tabele - GUID partition table.

Začetki in konci razdelkov so zapisani kot 64-bitna števila. Za popravljanje tabele razdelkov lahko uporabimo iste programe kot za popravljanje tabele razdelkov v MBR: [`fdisk`](https://www.man7.org/linux/man-pages/man8/fdisk.8.html), [`gdisk`](https://linux.die.net/man/8/gdisk), [`sfdisk`](https://man7.org/linux/man-pages/man8/sfdisk.8.html)...

### Upravljanje z več diski hkrati

Ko imamo več diskov hkrati priklopljenih na računalnik, nam to omogoča smotrno povezavo le teh v skupno polje, da se operacijskemu sistemu prikažejo le kot eden fizičen disk. Pri tem lahko dosežemo večjo z zmogljivostjo in/ali boljšo redundanco podatkov v primerjavo z enim ali več neodvisnimi diski. RAID pričakuje diske enake velikosti, v primeru različnih velikosti določi velikost posameznega diska najmanjši disk v polju.

Diske lahko povežemo v različna [Redundant array of independent disks (RAID) polja](https://en.wikipedia.org/wiki/RAID), glede na naše potrebe. Na začetku RAID razdelka imamo podatke o RAID polju oz. metapodatke. Metapodatki so nam načelom znani v primeru programskega RAID polja ustvarjena znotraj operacijskih sistemov kot sta GNU/Linux in BSD. Enako velja za RAID polja ustvarjena s gonilniki, ki so del matičnih plošč računalnika oz. z drugo besedo "fakeraid". Medtem ko nam v primer strojnega RAID polja, ustvarjenega z namenskimi zaprto-kodnimi strojnimi RAID kontrolerjem, le ti niso znani. Od tega ali znamo prebrati metapodatke je odvisna naša zmožnost ponovne vzpostavitve RAID polja.

Za delo s programskih RAID poljem uporabljamo orodje [`mdadm`](https://www.man7.org/linux/man-pages/man8/mdadm.8.html) in za delo s fakeraid RAID poljem uporabljamo orodje [`dmraid`](https://linux.die.net/man/8/dmraid).

Najbolj pogoste oblike [RAID polij](https://en.wikipedia.org/wiki/RAID#Standard_levels):

- **RAID 0**: Polje ima velikost enako vsoti velikosti diskov. Omogoča hitrejše branje in pisanje saj so datoteke razporejene po vseh diskih hkrati. Polje nima redundance, torej v primeru odpovedi enega diska izgubimo vse podatke v polju.
- **RAID 1**: Polje ima velikost enako polovici vsote velikosti diskov. Omogoča hitrejše branje saj so vse datoteke zapisane zrcalno med vsemi diski, pisanje pa je tako hitro, kot lahko zapisuje najpočasnejši disk. Polje ima redundanco, saj so podatki zrcaljeni med diski, kar nam omogoča ohranitev podatkov pri odpovedi enega diska (lahko tudi več, vendar odvisno od organizacije).
- **RAID 5**: Polje ima velikost enako vsoti velikosti diskov minus velikost enega diska. Omogoča hitrejše branje in pisanje saj so datoteke razporejen po vseh diskih hkrati, prav tako pa se za vsak sektor zapiše pariteta (na primer rezultat XOR). Polje ima redundanco in nam omogoča ohranitev podatkov v primeru odpovedi enega diska.
- **RAID 6**: Polje ima velikost enako vsoti velikosti diskov minus velikost dveh diskov. Omogoča hitrejše branje in pisanje saj so datoteke razporejen po vseh diskih hkrati, prav tako pa se za vsak sektor zapišeta dve pariteti (na primer rezultat XOR). Polje ima redundanco in nam omogoča ohranitev podatkov v primeru odpovedi dveh diskov hkrati.
- **RAID 10**: Polje je sestavljeno iz več RAID 1 in 0 polj. Omogoča hitrejše branje in pisanje podatkov saj so datoteke razporejene po več diskih hkrati ter diski so tudi zrcaljeni. Polje ima redundanco razen v primeru, ko hkrati izgubimo vse zrcaljene kopije posameznega diska.

### Logični diski

Več diskov lahko povežemo skupaj v enega tudi na logičnem nivoju, ne glede ali so to posamezni diski, posamezni razdelki in RAID polja ter poljubnih velikosti. To v GNU/Linux operacijskih sistemih lahko naredimo z uporabo [Logical volume Manager (LVM)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_%28Linux%29) orodjem.

LVM definirajo naslednje entitete:

- **Physical Volume (PV)** predstavljajo fizične diske ali razdelke.
- **Volume Group (VG)** predstavljajo skupine sestavljene iz PV.
- **Logical Volume (LV)** predstavljajo logične diske, ki so sestavljeni iz PV znotraj ene VG.

LVM prav tako omogočajo izdelavo posnetkov diska (snapshots), ki se uporabljajo za varnostno kopiranje diska in možnost povrnitve na starejšo različico diska.

Pod GNU/Linux upravljamo z LVM logičnimi diski z orodjem [`lvm2`](https://www.man7.org/linux/man-pages/man8/lvm.8.html).

Windows operacijski sistem omogoča podobno funkcionalnost preko [Dynamic Volumes - Logical Disk Manager](https://learn.microsoft.com/en-us/windows/win32/fileio/basic-and-dynamic-disks). Pod GNU/Linux-om lahko dostopamo do takih diskov z orodjem [`ldmtool`](https://man.archlinux.org/man/ldmtool.1.en).

Znotraj logičnih diskov lahko potem vzpostavimo poljubne datotečne sisteme, ki imajo svoje prednosti, slabosti in posebnosti:

- [EXT4](https://en.wikipedia.org/wiki/Ext4)
- [BTRFS](https://en.wikipedia.org/wiki/Btrfs)
- [ZFS](https://en.wikipedia.org/wiki/ZFS)
- [NTFS](https://en.wikipedia.org/wiki/NTFS)
- [FAT32](https://en.wikipedia.org/wiki/File_Allocation_Table)
