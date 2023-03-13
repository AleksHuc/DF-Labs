# 4. Vaja: Metapodatki

## Navodila

1. Za podane slike ugotovite s kakšnim fotoaparatom so bile zajete, ali je bila uporabljena bliskavica in kdaj in kje je nastala.
2. Za podano datoteko ugotovite kdo je originalni avtor dokumenta, ali je dokument spreminjal še kdo drug ter kdaj in na katerem operacijskem sistemu je nastal.
3. Na podanih slikah in dokumentu spremenite in izbrišite posamezne metapodatke.

## Dodatne informacije

## Podrobna navodila

### 1. Metapodatki v slikah

S [spletne strani](https://polz.si/dsrf/) prenesemo naslednji dve sliki:

- `lovecnabiralec.jpg`
- `JFP_5195.NEF`

    wget https://polz.si/dsrf/images/lovecnabiralec.jpg
    wget https://polz.si/dsrf/JFP_5195.NEF

Sliki najprej odpremo kar z dvojnim klikom ter prikažemo njune dodatne lastnosti znotraj aplikacije ali pa z desnim klikom in izberemo možnost `Podrobnosti`. Dodatne podatke o sliki, ki so shranjeni v datoteki poleg slike, imenujemo metapodatki ali tudi [EXIF podatki (Exchangeable Image File Format)](https://en.wikipedia.org/wiki/Exif).

Za branje metapodatkov lahko uporabimo namenska orodja, kot sta  [`exiv2`](https://linux.die.net/man/1/exiv2) in [`exif`](https://manpages.debian.org/bullseye/exif/exif.1.en.html). Z njima lahko dostopamo do vseh metapodatkov, ki so shranjeni v sliki in ne le do tistih, ki jih prikaže program za prikazovanje slik. Namestimo jo z upraviteljem paketov na našem operacijskem sistemu in z njima izpišemo metapodatke naših dveh slik

    apt update
    apt install exiv2 exif

    exiv2 lovecnabiralec.jpg

    File name       : lovecnabiralec.jpg
    File size       : 2765775 Bytes
    MIME type       : image/jpeg
    Image size      : 2592 x 1936
    Camera make     : Apple
    Camera model    : iPhone 4
    Image timestamp : 2012:01:25 14:50:25
    Image number    : 
    Exposure time   : 1/15 s
    Aperture        : F2.8
    Exposure bias   : 
    Flash           : No flash
    Flash bias      : 
    Focal length    : 3.8 mm
    Subject distance: 
    ISO speed       : 125
    Exposure mode   : Auto
    Metering mode   : Multi-segment
    Macro mode      : 
    Image quality   : 
    Exif Resolution : 2592 x 1936
    White balance   : Auto
    Thumbnail       : image/jpeg, 13456 Bytes
    Copyright       : 
    Exif comment    : 

    exif lovecnabiralec.jpgAnalysis of disks with GNU/Linux

    EXIF tags in 'lovecnabiralec.jpg' ('Motorola' byte order):
    --------------------+----------------------------------------------------------
    Tag                 |Value
    --------------------+----------------------------------------------------------
    Manufacturer        |Apple
    Model               |iPhone 4
    Orientation         |Right-top
    X-Resolution        |72
    Y-Resolution        |72
    Resolution Unit     |Inch
    Software            |5.0.1
    Date and Time       |2012:01:25 14:50:25
    YCbCr Positioning   |Centered
    Compression         |JPEG compression
    X-Resolution        |72
    Y-Resolution        |72
    Resolution Unit     |Inch
    Exposure Time       |1/15 sec.
    F-Number            |f/2.8
    Exposure Program    |Normal program
    ISO Speed Ratings   |125
    Exif Version        |Exif Version 2.21
    Date and Time (Origi|2012:01:25 14:50:25
    Date and Time (Digit|2012:01:25 14:50:25
    Components Configura|Y Cb Cr -
    Shutter Speed       |3.91 EV (1/15 sec.)
    Aperture            |2.97 EV (f/2.8)
    Brightness          |2.28 EV (16.65 cd/m^2)
    Metering Mode       |Pattern
    Flash               |Flash did not fire
    Focal Length        |3.9 mm
    FlashPixVersion     |FlashPix Version 1.0
    Color Space         |sRGB
    Pixel X Dimension   |2592
    Pixel Y Dimension   |1936
    Sensing Method      |One-chip color area sensor
    Custom Rendered     |2
    Exposure Mode       |Auto exposure
    White Balance       |Auto white balance
    Scene Capture Type  |Standard
    North or South Latit|N
    Latitude            |46, 4.45,  0
    East or West Longitu|E
    Longitude           |14, 28.68,  0
    Altitude Reference  |Sea level
    Altitude            |310.386
    GPS Time (Atomic Clo|13:50:1561.00
    GPS Image Direction |T
    GPS Image Direction |180.936
    --------------------+----------------------------------------------------------
    EXIF data contains a thumbnail (13456 bytes).

    exiv2 JFP_5195.NEF 

    File name       : JFP_5195.NEF
    File size       : 17512345 Bytes
    MIME type       : image/x-nikon-nef
    Image size      : 4992 x 3292
    Camera make     : NIKON CORPORATION
    Camera model    : NIKON D4
    Image timestamp : 2013:06:12 13:44:18
    Image number    : 
    Exposure time   : 1/100 s
    Aperture        : F5
    Exposure bias   : 0 EV
    Flash           : No flash
    Flash bias      : 
    Focal length    : 35.0 mm (35 mm equivalent: 35.0 mm)
    Subject distance: 
    ISO speed       : 200
    Exposure mode   : Manual
    Metering mode   : Multi-segment
    Macro mode      : 
    Image quality   : RAW    
    Exif Resolution : 160 x 120
    White balance   : AUTO1       
    Thumbnail       : None
    Copyright       :                                                       
    Exif comment    : charset=Ascii

    exif JFP_5195.NEF 

    Corrupt data
    The data provided does not follow the specification.
    ExifLoader: The data supplied does not seem to contain EXIF data.

### 2. Metapodatki v dokumentih

S [spletne učilnice]() prenesite datoteko:

- [`blinkenlichten.odt`](https://ucilnica.fri.uni-lj.si/mod/resource/view.php?id=28647)





