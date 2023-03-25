# 4. Lab: Metadata

## Instructions

1. For the given pictures, find out what kind of camera they were taken with, whether a flash was used, and when and where they were taken.
2. For the given document, find out who the original author of the document is, whether the document was modified by someone else, and when and on which operating system it was created.
3. Modify and delete individual metadata on the given images and document.
4. Write a short program to get, change and delete EXIF metadata of images.

## More information

## Detailed instructions

### 1. Metadata in images

Many file formats support metadata storing. These usually include the date and time the file was created, the author and the program it was created with. In addition, for example, an image may contain information about the location and the device with which it was captured. This data is easy to overlook as programs often do not display it, so it is often a source of useful forensic information. Web applications are supposed to delete metadata, but no one guarantees us this and they can use it for analysis or sell it to other companies, mainly to advertisers.

Among the most widely used standards for recording metadata in images and videos are Exif, IPTC and XMP, which can be processed using the [Exiv2](https://exiv2.org/) library and tools. Documents in modern formats, such as Office Open XML, OpenDocument and EPUB, are stored as ordinary ZIP files, which include, among other things, a metadata file (usually written in XML format).

Download the following two images from [website](https://polz.si/dsrf/):

- `lovecnabiralec.jpg`
- `JFP_5195.NEF`

First open the images by double-clicking and displaying their additional properties within the application, or by right-clicking and selecting the `Details` option. Additional information about the image, which is stored in a file next to the image, is called metadata or also [EXIF data (Exchangeable Image File Format)](https://en.wikipedia.org/wiki/Exif).

To read metadata we can use dedicated tools such as [`exiv2`](https://linux.die.net/man/1/exiv2), [`exif`](https://manpages.debian.org/bullseye/exif/exif.1.en.html) and [`exiftool`](https://linux.die.net/man/1/exiftool). With them, we can access all the metadata stored in the image and not only those displayed by the image display program. We install them with the package manager on our operating system and use them to print out the metadata of our two images.

apt update
    apt install exiv2 exif exiftool

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

    exiftool lovecnabiralec.jpg

    ExifTool Version Number         : 12.16
    File Name                       : lovecnabiralec.jpg
    Directory                       : .
    File Size                       : 2.6 MiB
    File Modification Date/Time     : 2023:03:13 11:13:03+01:00
    File Access Date/Time           : 2023:03:13 11:15:24+01:00
    File Inode Change Date/Time     : 2023:03:13 11:13:03+01:00
    File Permissions                : rw-r--r--
    File Type                       : JPEG
    File Type Extension             : jpg
    MIME Type                       : image/jpeg
    Exif Byte Order                 : Big-endian (Motorola, MM)
    Make                            : Apple
    Camera Model Name               : iPhone 4
    Orientation                     : Rotate 90 CW
    X Resolution                    : 72
    Y Resolution                    : 72
    Resolution Unit                 : inches
    Software                        : 5.0.1
    Modify Date                     : 2012:01:25 14:50:25
    Y Cb Cr Positioning             : Centered
    Exposure Time                   : 1/15
    F Number                        : 2.8
    Exposure Program                : Program AE
    ISO                             : 125
    Exif Version                    : 0221
    Date/Time Original              : 2012:01:25 14:50:25
    Create Date                     : 2012:01:25 14:50:25
    Components Configuration        : Y, Cb, Cr, -
    Shutter Speed Value             : 1/15
    Aperture Value                  : 2.8
    Brightness Value                : 2.281069959
    Metering Mode                   : Multi-segment
    Flash                           : No Flash
    Focal Length                    : 3.9 mm
    Flashpix Version                : 0100
    Color Space                     : sRGB
    Exif Image Width                : 2592
    Exif Image Height               : 1936
    Sensing Method                  : One-chip color area
    Custom Rendered                 : HDR (no original saved)
    Exposure Mode                   : Auto
    White Balance                   : Auto
    Scene Capture Type              : Standard
    GPS Latitude Ref                : North
    GPS Longitude Ref               : East
    GPS Altitude Ref                : Above Sea Level
    GPS Time Stamp                  : 14:16:01
    GPS Img Direction Ref           : True North
    GPS Img Direction               : 180.9357143
    Compression                     : JPEG (old-style)
    Thumbnail Offset                : 882
    Thumbnail Length                : 13456
    Image Width                     : 2592
    Image Height                    : 1936
    Encoding Process                : Baseline DCT, Huffman coding
    Bits Per Sample                 : 8
    Color Components                : 3
    Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
    Aperture                        : 2.8
    Image Size                      : 2592x1936
    Megapixels                      : 5.0
    Shutter Speed                   : 1/15
    Thumbnail Image                 : (Binary data 13456 bytes, use -b option to extract)
    GPS Altitude                    : 310.3 m Above Sea Level
    GPS Latitude                    : 46 deg 4' 27.00" N
    GPS Longitude                   : 14 deg 28' 40.80" E
    Focal Length                    : 3.9 mm
    GPS Position                    : 46 deg 4' 27.00" N, 14 deg 28' 40.80" E
    Light Value                     : 6.6

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

    exiftool JFP_5195.NEF 

    ExifTool Version Number         : 12.16
    File Name                       : JFP_5195.NEF
    Directory                       : .
    File Size                       : 17 MiB
    File Modification Date/Time     : 2023:03:13 11:12:55+01:00
    File Access Date/Time           : 2023:03:13 11:12:55+01:00
    File Inode Change Date/Time     : 2023:03:13 11:12:55+01:00
    File Permissions                : rw-r--r--
    File Type                       : NEF
    File Type Extension             : nef
    MIME Type                       : image/x-nikon-nef
    Exif Byte Order                 : Big-endian (Motorola, MM)
    Make                            : NIKON CORPORATION
    Camera Model Name               : NIKON D4
    Orientation                     : Horizontal (normal)
    Software                        : Ver.1.02
    Modify Date                     : 2013:06:12 13:44:18
    Artist                          : 
    Jpg From Raw Start              : 786432
    Jpg From Raw Length             : 513834
    Y Cb Cr Positioning             : Co-sited
    Image Width                     : 4992
    Image Height                    : 3292
    Bits Per Sample                 : 14
    Compression                     : Nikon NEF Compressed
    Photometric Interpretation      : Color Filter Array
    Strip Offsets                   : 1310720
    Samples Per Pixel               : 1
    Rows Per Strip                  : 3292
    Strip Byte Counts               : 16201625
    X Resolution                    : 300
    Y Resolution                    : 300
    Planar Configuration            : Chunky
    Resolution Unit                 : inches
    CFA Repeat Pattern Dim          : 2 2
    CFA Pattern 2                   : 0 1 1 2
    Subfile Type                    : Reduced-resolution image
    Other Image Start               : 196608
    Other Image Length              : 528264
    Reference Black White           : 0 255 0 255 0 255
    Creator Tool                    : NIKON D4 Ver.1.02
    Copyright                       : 
    Exposure Time                   : 1/100
    F Number                        : 5.0
    Exposure Program                : Manual
    ISO                             : 200
    Sensitivity Type                : Recommended Exposure Index
    Create Date                     : 2013:06:12 13:44:18
    Exposure Compensation           : 0
    Max Aperture Value              : 2.0
    Metering Mode                   : Multi-segment
    Light Source                    : Unknown
    Flash                           : No Flash
    Focal Length                    : 35.0 mm
    Maker Note Version              : 2.10
    Quality                         : RAW
    White Balance                   : Auto1
    Focus Mode                      : AF-S
    Flash Setting                   : 
    Flash Type                      : 
    White Balance Fine Tune         : 0 0
    WB RB Levels                    : 2.05078125 1.55078125 1 1
    Program Shift                   : 0
    Exposure Difference             : -4.2
    Preview Image Start             : 37016
    Preview Image Length            : 81433
    ISO Setting                     : 200
    External Flash Exposure Comp    : 0
    Flash Exposure Bracket Value    : 0.0
    Exposure Bracket Value          : 0
    Crop Hi Speed                   : Off (4992x3292 cropped to 4992x3292 at pixel 0,0)
    Exposure Tuning                 : 0
    Serial Number                   : 2042809
    Color Space                     : sRGB
    VR Info Version                 : 0100
    Vibration Reduction             : Off
    VR Mode                         : Normal
    Active D-Lighting               : Off
    Picture Control Version         : 0100
    Picture Control Name            : Neutral
    Picture Control Base            : Neutral
    Picture Control Adjust          : Default Settings
    Picture Control Quick Adjust    : n/a
    Brightness                      : Normal
    Hue Adjustment                  : None
    Filter Effect                   : n/a
    Toning Effect                   : n/a
    Toning Saturation               : n/a
    Time Zone                       : +01:00
    Daylight Savings                : No
    Date Display Format             : D/M/Y
    ISO Expansion                   : Off
    ISO2                            : 200
    ISO Expansion 2                 : Off
    Vignette Control                : Normal
    Auto Distortion Control         : Off
    Lens Type                       : D
    Lens                            : 35mm f/2
    Flash Mode                      : Did Not Fire
    Shooting Mode                   : Single-Frame
    Contrast Curve                  : (Binary data 578 bytes, use -b option to extract)
    Shot Info Version               : 0223
    Firmware Version                : 1.02b
    Custom Settings Bank            : A
    AF-C Priority Selection         : Release
    AF-S Priority Selection         : Focus
    AF Point Selection              : 51 Points
    Focus Tracking Lock On          : 3 (Normal)
    AF Activation                   : Shutter/AF-On
    Focus Point Wrap                : No Wrap
    Pitch                           : High
    No Memory Card                  : Enable Release
    Grid Display                    : On
    Shooting Info Display           : Unknown (0)
    LCD Illumination                : Off
    Screen Tips                     : On
    Beep                            : Off
    Reverse Indicators              : + 0 -
    Rear Display                    : ISO
    Viewfinder Display              : Exposures Remaining
    Command Dials Reverse Rotation  : No
    Easy Exposure Compensation      : Off
    Exposure Control Step Size      : 1/3 EV
    ISO Step Size                   : 1/3 EV
    Exposure Comp Step Size         : 1/3 EV
    Center Weighted Area Size       : 12 mm
    Fine Tune Opt Matrix Metering   : 0
    Fine Tune Opt Center Weighted   : 0
    Fine Tune Opt Spot Metering     : 0
    Multi Selector Shoot Mode       : Select Center Focus Point (Reset)
    Multi Selector Playback Mode    : Thumbnail On/Off
    Multi Selector                  : Do Nothing
    Exposure Delay Mode             : Off
    CH Mode Shooting Speed          : 10 fps
    CL Mode Shooting Speed          : 5 fps
    Max Continuous Release          : 200
    Auto Bracket Set                : AE & Flash
    Auto Bracket Order              : 0,-,+
    Auto Bracket Mode M             : Flash/Speed
    Func Button                     : My Menu
    Func Button Plus Dials          : None
    Preview Button                  : Preview
    Preview Button Plus Dials       : None
    Assign Bkt Button               : Auto Bracketing
    Command Dials Change Main Sub   : Autofocus Off, Exposure Off
    Command Dials Menu And Playback : Off
    Command Dials Aperture Setting  : Sub-command Dial
    Shutter Release Button AE-L     : On
    Release Button To Use Dial      : No
    Standby Timer                   : 6 s
    Self Timer Time                 : 2 s
    Self Timer Shot Count           : 1
    Self Timer Shot Interval        : 0.5 s
    Image Review Monitor Off Time   : 4 s
    Live View Monitor Off Time      : 10 min
    Menu Monitor Off Time           : 20 s
    Shooting Info Monitor Off Time  : 10 s
    Flash Sync Speed                : 1/250 s
    Flash Shutter Speed             : 1/60 s
    Modeling Flash                  : Off
    Playback Monitor Off Time       : 10 s
    Playback Zoom                   : Use Separate Zoom Buttons
    Shutter Speed Lock              : Off
    Aperture Lock                   : Off
    Movie Shutter Button            : Take Photo
    Flash Exposure Comp Area        : Entire frame
    Movie Function Button           : None
    Movie Preview Button            : Index Marking
    Vertical Multi Selector         : Same as Multi-Selector with Info(U/D) & Playback(R/L)
    Vertical Func Button            : AE/AF Lock
    Vertical Func Button Plus Dials : None
    Assign Movie Record Button      : None
    Dynamic Area AF Display         : Off
    AF Point Illumination           : On in Continuous Shooting and Manual Focusing
    Store By Orientation            : Off
    Group Area AF Illumination      : Squares
    AF Point Brightness             : Normal
    AF On Button                    : AE/AF Lock
    Vertical AF On Button           : AE/AF Lock
    Sub Selector Assignment         : Focus Point Selection
    Movie Sub Selector Assignment   : AE/AF Lock
    Sub Selector                    : AE/AF Lock
    Sub Selector Plus Dials         : None
    Movie Function Button Plus Dials: None
    Movie Preview Button Plus Dials : Choose Image Area
    Movie Sub Selector Assignment Plus Dials: None
    NEF Compression                 : Lossless
    Noise Reduction                 : Off
    NEF Linearization Table         : (Binary data 46 bytes, use -b option to extract)
    WB GRBG Levels                  : 256 525 397 256
    Lens Data Version               : 0204
    Exit Pupil Position             : 58.5 mm
    AF Aperture                     : 2.1
    Focus Position                  : 0x11
    Focus Distance                  : 0.50 m
    Lens ID Number                  : 66
    Lens F Stops                    : 7.00
    Min Focal Length                : 35.6 mm
    Max Focal Length                : 35.6 mm
    Max Aperture At Min Focal       : 2.0
    Max Aperture At Max Focal       : 2.0
    MCU Version                     : 68
    Effective Max Aperture          : 2.0
    Raw Image Center                : 2496 1646
    Retouch History                 : None
    Shutter Count                   : 17094
    Flash Info Version              : 0105
    Flash Source                    : None
    External Flash Firmware         : n/a
    External Flash Flags            : (none)
    Flash Commander Mode            : Off
    Flash Control Mode              : Off
    Flash Compensation              : 0
    Flash GN Distance               : 0
    Flash Color Filter              : None
    Flash Group A Control Mode      : Off
    Flash Group B Control Mode      : Off
    Flash Group C Control Mode      : Off
    Flash Group A Compensation      : 0
    Flash Group B Compensation      : 0
    Flash Group C Compensation      : 0
    External Flash Compensation     : 0
    Flash Exposure Comp 3           : 0
    Flash Exposure Comp 4           : 0
    Multi Exposure Version          : 0100
    Multi Exposure Mode             : Off
    Multi Exposure Shots            : 0
    Multi Exposure Auto Gain        : Off
    High ISO Noise Reduction        : Normal
    Power Up Time                   : 0000:00:00 00:00:00
    AF Info 2 Version               : 0100
    Contrast Detect AF              : On
    AF Area Mode                    : Contrast-detect (normal area)
    Phase Detect AF                 : Off
    Primary AF Point                : (none)
    AF Points Used                  : (none)
    AF Image Width                  : 4928
    AF Image Height                 : 3280
    AF Area X Position              : 2464
    AF Area Y Position              : 1640
    AF Area Width                   : 264
    AF Area Height                  : 220
    Contrast Detect AF In Focus     : Yes
    File Info Version               : 0100
    Memory Card Number              : 0
    Directory Number                : 101
    File Number                     : 5195
    AF Fine Tune                    : Off
    AF Fine Tune Index              : n/a
    AF Fine Tune Adj                : 0
    AF Fine Tune Adj Tele           : 0
    Retouch Info Version            : 0200
    Retouch NEF Processing          : Off
    User Comment                    : 
    Sub Sec Time                    : 30
    Sub Sec Time Original           : 30
    Sub Sec Time Digitized          : 30
    Sensing Method                  : One-chip color area
    File Source                     : Digital Camera
    Scene Type                      : Directly photographed
    Custom Rendered                 : Normal
    Exposure Mode                   : Manual
    Digital Zoom Ratio              : 1
    Focal Length In 35mm Format     : 35 mm
    Scene Capture Type              : Standard
    Gain Control                    : None
    Contrast                        : Normal
    Saturation                      : Normal
    Sharpness                       : Normal
    Subject Distance Range          : Unknown
    GPS Version ID                  : 2.3.0.0
    Date/Time Original              : 2013:06:12 13:44:18
    TIFF-EP Standard ID             : 1 0 0 0
    Aperture                        : 5.0
    Blue Balance                    : 1.550781
    CFA Pattern                     : [Red,Green][Green,Blue]
    Image Size                      : 4992x3292
    Jpg From Raw                    : (Binary data 513834 bytes, use -b option to extract)
    Megapixels                      : 16.4
    Other Image                     : (Binary data 528264 bytes, use -b option to extract)
    Preview Image                   : (Binary data 81433 bytes, use -b option to extract)
    Red Balance                     : 2.050781
    Scale Factor To 35 mm Equivalent: 1.0
    Shutter Speed                   : 1/100
    Create Date                     : 2013:06:12 13:44:18.30
    Date/Time Original              : 2013:06:12 13:44:18.30
    Modify Date                     : 2013:06:12 13:44:18.30
    Thumbnail TIFF                  : (Binary data 57816 bytes, use -b option to extract)
    Auto Focus                      : On
    Lens ID                         : AF Nikkor 35mm f/2D
    Lens Spec                       : 35mm f/2 D
    Circle Of Confusion             : 0.030 mm
    Depth Of Field                  : 0.06 m (0.47 - 0.53 m)
    Field Of View                   : 51.1 deg (0.48 m)
    Focal Length                    : 35.0 mm (35 mm equivalent: 35.0 mm)
    Hyperfocal Distance             : 8.15 m
    Light Value                     : 10.3

Images may also contain lower resolution versions of the image that we can use to determine whether or not the original image has been altered. Also, they come in handy when the original image is damaged. A lower resolution image can be extracted using the `exiftool` tool and the `-ThumbnailImage` setting.

    exiftool -b -ThumbnailImage lovecnabiralec.jpg > lovecnabiralec_t.jpg

The tool [`dcraw`](https://linux.die.net/man/1/dcraw) allows us to extract images with a lower resolution and correct the original images. All obtained images are viewed with the program for displaying images (original image - `.NEF`, image with a lower resolution `.thumb.jpg` and corrected image - `.ppm`).

    apt update
    apt instal dcraw
    dcraw JFP_5195.NEF
    dcraw -e JFP_5195.NEF

In general, based on the captured image, we can determine with which camera it was captured, based on:

- EXIF metadata.
- [Chromatic aberration](https://en.wikipedia.org/wiki/Chromatic_aberration).
- With errors on individual pixels that are slightly lighter and darker and with a larger amount of images, we can build a model of the device.

### 2. Metadata in documents

Download the file from [online classroom](https://ucilnica.fri.uni-lj.si/course/view.php?id=178):

- [`blinkenlichten.odt`](https://ucilnica.fri.uni-lj.si/mod/resource/view.php?id=28647)

To access metadata, we can use tools for accessing metadata in images.

    exiv2 blinkenlichten.odt

    Exiv2 exception in print action for file blinkenlichten.odt:
    blinkenlichten.odt: The file contains data of an unknown image type

    exif blinkenlichten.odt

    Corrupt data
    The data provided does not follow the specification.
    ExifLoader: The data supplied does not seem to contain EXIF data.

    exiftool blinkenlichten.odt

    ExifTool Version Number         : 12.16
    File Name                       : blinkenlichten.odt
    Directory                       : .
    File Size                       : 11 KiB
    File Modification Date/Time     : 2023:03:13 11:14:39+01:00
    File Access Date/Time           : 2023:03:13 11:15:40+01:00
    File Inode Change Date/Time     : 2023:03:13 11:14:39+01:00
    File Permissions                : rw-r--r--
    File Type                       : ODT
    File Type Extension             : odt
    MIME Type                       : application/vnd.oasis.opendocument.text
    Initial-creator                 : Pišta Bači
    Creation-date                   : 2019:04:16 21:06:40.274118679
    Date                            : 2019:04:16 22:13:49.342982850
    Creator                         : Franko Frkič
    Editing-duration                : PT33S
    Editing-cycles                  : 3
    Generator                       : LibreOffice/6.1.3.2$Linux_X86_64 LibreOffice_project/10$Build-2
    Document-statistic Table-count  : 0
    Document-statistic Image-count  : 0
    Document-statistic Object-count : 0
    Document-statistic Page-count   : 1
    Document-statistic Paragraph-count: 6
    Document-statistic Word-count   : 44
    Document-statistic Character-count: 344
    Document-statistic Non-whitespace-character-count: 305
    Preview PNG                     : (Binary data 2855 bytes, use -b option to extract)

The file `blinkenlichten.odt` contains the document and the metadata in separate files as a ZIP file and can be accessed just by using the `unzip` command and then outputting the `XML` files that contain the metadata.

    mkdir blinkenlichten
    cd blinkenlichten
    unzip ../blinkenlichten.odt 

    Archive:  ../blinkenlichten.odt
    extracting: mimetype                
    extracting: Thumbnails/thumbnail.png  
    creating: Configurations2/toolpanel/
    creating: Configurations2/menubar/
    creating: Configurations2/statusbar/
    creating: Configurations2/accelerator/
    creating: Configurations2/progressbar/
    creating: Configurations2/popupmenu/
    creating: Configurations2/toolbar/
    creating: Configurations2/images/Bitmaps/
    creating: Configurations2/floater/
    inflating: content.xml             
    inflating: meta.xml                
    inflating: manifest.rdf            
    inflating: settings.xml            
    inflating: styles.xml              
    inflating: META-INF/manifest.xml

    ls
    Configurations2  content.xml  manifest.rdf  META-INF  meta.xml	mimetype  settings.xml	styles.xml  Thumbnails

    apt update
    apt install libxml2-utils

    cat meta.xml | xmllint --format -

    <?xml version="1.0" encoding="UTF-8"?>
    <office:document-meta xmlns:office="urn:oasis:names:tc:opendocument:xmlns:office:1.0" xmlns:xlink="http://www.w3.org/1999/xlink" xmlns:dc="http://purl.org/dc/elements/1.1/" xmlns:meta="urn:oasis:names:tc:opendocument:xmlns:meta:1.0" xmlns:ooo="http://openoffice.org/2004/office" xmlns:grddl="http://www.w3.org/2003/g/data-view#" office:version="1.2">
    <office:meta>
        <meta:initial-creator>Pišta Bači</meta:initial-creator>
        <meta:creation-date>2019-04-16T21:06:40.274118679</meta:creation-date>
        <dc:date>2019-04-16T22:13:49.342982850</dc:date>
        <dc:creator>Franko Frkič</dc:creator>
        <meta:editing-duration>PT33S</meta:editing-duration>
        <meta:editing-cycles>3</meta:editing-cycles>
        <meta:generator>LibreOffice/6.1.3.2$Linux_X86_64 LibreOffice_project/10$Build-2</meta:generator>
        <meta:document-statistic meta:table-count="0" meta:image-count="0" meta:object-count="0" meta:page-count="1" meta:paragraph-count="6" meta:word-count="44" meta:character-count="344" meta:non-whitespace-character-count="305"/>
    </office:meta>
    </office:document-meta>

### 3. Modification and deletion of metadata

With the `exiv2` tool, we can first extract all the data with the `ex` setting into a file of type `.exv`. The names for addressing individual metadata fields are printed with the `-pt` setting. Individual metadata can be manipulated with the `-M` setting and then we determine whether the metadata is changed with the `set` command, added with the `add` command, or deleted with the `del` command. All metadata can be deleted with the `rm` option and can be imported from a file with the `in` option.

    exiv2 ex lovecnabiralec.jpg

    ls
    blinkenlichten	blinkenlichten.odt  JFP_5195.NEF  lovecnabiralec.exv  lovecnabiralec.jpg

    exiv2 -pt lovecnabiralec.jpg

    Exif.Image.Make                              Ascii       6  Apple
    Exif.Image.Model                             Ascii       9  iPhone 4
    Exif.Image.Orientation                       Short       1  right, top
    Exif.Image.XResolution                       Rational    1  72
    Exif.Image.YResolution                       Rational    1  72
    Exif.Image.ResolutionUnit                    Short       1  inch
    Exif.Image.Software                          Ascii       6  5.0.1
    Exif.Image.DateTime                          Ascii      20  2012:01:25 14:50:25
    Exif.Image.YCbCrPositioning                  Short       1  Centered
    Exif.Image.ExifTag                           Long        1  204
    Exif.Photo.ExposureTime                      Rational    1  1/15 s
    Exif.Photo.FNumber                           Rational    1  F2.8
    Exif.Photo.ExposureProgram                   Short       1  Auto
    Exif.Photo.ISOSpeedRatings                   Short       1  125
    Exif.Photo.ExifVersion                       Undefined   4  2.21
    Exif.Photo.DateTimeOriginal                  Ascii      20  2012:01:25 14:50:25
    Exif.Photo.DateTimeDigitized                 Ascii      20  2012:01:25 14:50:25
    Exif.Photo.ComponentsConfiguration           Undefined   4  YCbCr
    Exif.Photo.ShutterSpeedValue                 SRational   1  1/15 s
    Exif.Photo.ApertureValue                     Rational    1  F2.8
    Exif.Photo.BrightnessValue                   SRational   1  2.28107
    Exif.Photo.MeteringMode                      Short       1  Multi-segment
    Exif.Photo.Flash                             Short       1  No flash
    Exif.Photo.FocalLength                       Rational    1  3.8 mm
    Exif.Photo.FlashpixVersion                   Undefined   4  1.00
    Exif.Photo.ColorSpace                        Short       1  sRGB
    Exif.Photo.PixelXDimension                   Long        1  2592
    Exif.Photo.PixelYDimension                   Long        1  1936
    Exif.Photo.SensingMethod                     Short       1  One-chip color area
    Exif.Photo.CustomRendered                    Short       1  (2)
    Exif.Photo.ExposureMode                      Short       1  Auto
    Exif.Photo.WhiteBalance                      Short       1  Auto
    Exif.Photo.SceneCaptureType                  Short       1  Standard
    Exif.Image.GPSTag                            Long        1  574
    Exif.GPSInfo.GPSLatitudeRef                  Ascii       2  North
    Exif.GPSInfo.GPSLatitude                     Rational    3  46deg 4' 0"
    Exif.GPSInfo.GPSLongitudeRef                 Ascii       2  East
    Exif.GPSInfo.GPSLongitude                    Rational    3  14deg 29' 0"
    Exif.GPSInfo.GPSAltitudeRef                  Byte        1  Above sea level
    Exif.GPSInfo.GPSAltitude                     Rational    1  310.4 m
    Exif.GPSInfo.GPSTimeStamp                    Rational    3  14:16:01
    Exif.GPSInfo.GPSImgDirectionRef              Ascii       2  True direction
    Exif.GPSInfo.GPSImgDirection                 Rational    1  25331/140
    Exif.Thumbnail.Compression                   Short       1  JPEG (old-style)
    Exif.Thumbnail.XResolution                   Rational    1  72
    Exif.Thumbnail.YResolution                   Rational    1  72
    Exif.Thumbnail.ResolutionUnit                Short       1  inch
    Exif.Thumbnail.JPEGInterchangeFormat         Long        1  870
    Exif.Thumbnail.JPEGInterchangeFormatLength   Long        1  13456

    exiv2 -M"set Exif.Image.Model iPhone 6" lovecnabiralec.jpg

    exiv2 -pt lovecnabiralec.jpg

    Exif.Image.Make                              Ascii       6  Apple
    Exif.Image.Model                             Ascii       9  iPhone 6
    Exif.Image.Orientation                       Short       1  right, top
    Exif.Image.XResolution                       Rational    1  72
    Exif.Image.YResolution                       Rational    1  72
    Exif.Image.ResolutionUnit                    Short       1  inch
    Exif.Image.Software                          Ascii       6  5.0.1
    Exif.Image.DateTime                          Ascii      20  2012:01:25 14:50:25
    Exif.Image.YCbCrPositioning                  Short       1  Centered
    Exif.Image.ExifTag                           Long        1  204
    Exif.Photo.ExposureTime                      Rational    1  1/15 s
    Exif.Photo.FNumber                           Rational    1  F2.8
    Exif.Photo.ExposureProgram                   Short       1  Auto
    Exif.Photo.ISOSpeedRatings                   Short       1  125
    Exif.Photo.ExifVersion                       Undefined   4  2.21
    Exif.Photo.DateTimeOriginal                  Ascii      20  2012:01:25 14:50:25
    Exif.Photo.DateTimeDigitized                 Ascii      20  2012:01:25 14:50:25
    Exif.Photo.ComponentsConfiguration           Undefined   4  YCbCr
    Exif.Photo.ShutterSpeedValue                 SRational   1  1/15 s
    Exif.Photo.ApertureValue                     Rational    1  F2.8
    Exif.Photo.BrightnessValue                   SRational   1  2.28107
    Exif.Photo.MeteringMode                      Short       1  Multi-segment
    Exif.Photo.Flash                             Short       1  No flash
    Exif.Photo.FocalLength                       Rational    1  3.8 mm
    Exif.Photo.FlashpixVersion                   Undefined   4  1.00
    Exif.Photo.ColorSpace                        Short       1  sRGB
    Exif.Photo.PixelXDimension                   Long        1  2592
    Exif.Photo.PixelYDimension                   Long        1  1936
    Exif.Photo.SensingMethod                     Short       1  One-chip color area
    Exif.Photo.CustomRendered                    Short       1  (2)
    Exif.Photo.ExposureMode                      Short       1  Auto
    Exif.Photo.WhiteBalance                      Short       1  Auto
    Exif.Photo.SceneCaptureType                  Short       1  Standard
    Exif.Image.GPSTag                            Long        1  574
    Exif.GPSInfo.GPSLatitudeRef                  Ascii       2  North
    Exif.GPSInfo.GPSLatitude                     Rational    3  46deg 4' 0"
    Exif.GPSInfo.GPSLongitudeRef                 Ascii       2  East
    Exif.GPSInfo.GPSLongitude                    Rational    3  14deg 29' 0"
    Exif.GPSInfo.GPSAltitudeRef                  Byte        1  Above sea level
    Exif.GPSInfo.GPSAltitude                     Rational    1  310.4 m
    Exif.GPSInfo.GPSTimeStamp                    Rational    3  14:16:01
    Exif.GPSInfo.GPSImgDirectionRef              Ascii       2  True direction
    Exif.GPSInfo.GPSImgDirection                 Rational    1  25331/140
    Exif.Thumbnail.Compression                   Short       1  JPEG (old-style)
    Exif.Thumbnail.XResolution                   Rational    1  72
    Exif.Thumbnail.YResolution                   Rational    1  72
    Exif.Thumbnail.ResolutionUnit                Short       1  inch
    Exif.Thumbnail.JPEGInterchangeFormat         Long        1  870
    Exif.Thumbnail.JPEGInterchangeFormatLength   Long        1  13456

    exiv2 rm lovecnabiralec.jpg

    exiv2 -pt lovecnabiralec.jpg

    exiv2 in lovecnabiralec.exv lovecnabiralec.jpg

    exiv2 -pt lovecnabiralec.jpg

    Exif.Image.Make                              Ascii       6  Apple
    Exif.Image.Model                             Ascii       9  iPhone 4
    Exif.Image.Orientation                       Short       1  right, top
    Exif.Image.XResolution                       Rational    1  72
    Exif.Image.YResolution                       Rational    1  72
    Exif.Image.ResolutionUnit                    Short       1  inch
    Exif.Image.Software                          Ascii       6  5.0.1
    Exif.Image.DateTime                          Ascii      20  2012:01:25 14:50:25
    Exif.Image.YCbCrPositioning                  Short       1  Centered
    Exif.Image.ExifTag                           Long        1  204
    Exif.Photo.ExposureTime                      Rational    1  1/15 s
    Exif.Photo.FNumber                           Rational    1  F2.8
    Exif.Photo.ExposureProgram                   Short       1  Auto
    Exif.Photo.ISOSpeedRatings                   Short       1  125
    Exif.Photo.ExifVersion                       Undefined   4  2.21
    Exif.Photo.DateTimeOriginal                  Ascii      20  2012:01:25 14:50:25
    Exif.Photo.DateTimeDigitized                 Ascii      20  2012:01:25 14:50:25
    Exif.Photo.ComponentsConfiguration           Undefined   4  YCbCr
    Exif.Photo.ShutterSpeedValue                 SRational   1  1/15 s
    Exif.Photo.ApertureValue                     Rational    1  F2.8
    Exif.Photo.BrightnessValue                   SRational   1  2.28107
    Exif.Photo.MeteringMode                      Short       1  Multi-segment
    Exif.Photo.Flash                             Short       1  No flash
    Exif.Photo.FocalLength                       Rational    1  3.8 mm
    Exif.Photo.FlashpixVersion                   Undefined   4  1.00
    Exif.Photo.ColorSpace                        Short       1  sRGB
    Exif.Photo.PixelXDimension                   Long        1  2592
    Exif.Photo.PixelYDimension                   Long        1  1936
    Exif.Photo.SensingMethod                     Short       1  One-chip color area
    Exif.Photo.CustomRendered                    Short       1  (2)
    Exif.Photo.ExposureMode                      Short       1  Auto
    Exif.Photo.WhiteBalance                      Short       1  Auto
    Exif.Photo.SceneCaptureType                  Short       1  Standard
    Exif.Image.GPSTag                            Long        1  574
    Exif.GPSInfo.GPSLatitudeRef                  Ascii       2  North
    Exif.GPSInfo.GPSLatitude                     Rational    3  46deg 4' 0"
    Exif.GPSInfo.GPSLongitudeRef                 Ascii       2  East
    Exif.GPSInfo.GPSLongitude                    Rational    3  14deg 29' 0"
    Exif.GPSInfo.GPSAltitudeRef                  Byte        1  Above sea level
    Exif.GPSInfo.GPSAltitude                     Rational    1  310.4 m
    Exif.GPSInfo.GPSTimeStamp                    Rational    3  14:16:01
    Exif.GPSInfo.GPSImgDirectionRef              Ascii       2  True direction
    Exif.GPSInfo.GPSImgDirection                 Rational    1  25331/140
    Exif.Thumbnail.Compression                   Short       1  JPEG (old-style)
    Exif.Thumbnail.XResolution                   Rational    1  72
    Exif.Thumbnail.YResolution                   Rational    1  72
    Exif.Thumbnail.ResolutionUnit                Short       1  inch
    Exif.Thumbnail.JPEGInterchangeFormat         Long        1  870
    Exif.Thumbnail.JPEGInterchangeFormatLength   Long        1  13456

With the `exiftool` tool, we can first extract all the metadata with the `-h` setting to a `.html` type file. The names for addressing individual metadata fields are printed with the `-args` setting. Individual metadata can be manipulated with the `-X` setting, where X represents the field we want to change with the `=` command, add it with the `+=` command, or delete it with the `-=` command. The original version of the image can be restored with the `-restore_original` command, as the tool keeps it just in case.

    exiftool -h lovecnabiralec.jpg > lovecnabiralec.html

    exiftool -args lovecnabiralec.jpg

    -ExifToolVersion=12.16
    -FileName=lovecnabiralec.jpg
    -Directory=.
    -FileSize=2.6 MiB
    -FileModifyDate=2023:03:13 15:53:09+01:00
    -FileAccessDate=2023:03:13 15:53:10+01:00
    -FileInodeChangeDate=2023:03:13 15:53:09+01:00
    -FilePermissions=rw-r--r--
    -FileType=JPEG
    -FileTypeExtension=jpg
    -MIMEType=image/jpeg
    -ExifByteOrder=Little-endian (Intel, II)
    -Make=Apple
    -Model=iPhone 4
    -Orientation=Rotate 90 CW
    -XResolution=72
    -YResolution=72
    -ResolutionUnit=inches
    -Software=5.0.1
    -ModifyDate=2012:01:25 14:50:25
    -YCbCrPositioning=Centered
    -ExposureTime=1/15
    -FNumber=2.8
    -ExposureProgram=Program AE
    -ISO=125
    -ExifVersion=0221
    -DateTimeOriginal=2012:01:25 14:50:25
    -CreateDate=2012:01:25 14:50:25
    -ComponentsConfiguration=Y, Cb, Cr, -
    -ShutterSpeedValue=1/15
    -ApertureValue=2.8
    -BrightnessValue=2.281069959
    -MeteringMode=Multi-segment
    -Flash=No Flash
    -FocalLength=3.9 mm
    -FlashpixVersion=0100
    -ColorSpace=sRGB
    -ExifImageWidth=2592
    -ExifImageHeight=1936
    -SensingMethod=One-chip color area
    -CustomRendered=HDR (no original saved)
    -ExposureMode=Auto
    -WhiteBalance=Auto
    -SceneCaptureType=Standard
    -GPSLatitudeRef=North
    -GPSLongitudeRef=East
    -GPSAltitudeRef=Above Sea Level
    -GPSTimeStamp=14:16:01
    -GPSImgDirectionRef=True North
    -GPSImgDirection=180.9357143
    -Compression=JPEG (old-style)
    -ThumbnailOffset=882
    -ThumbnailLength=13456
    -ImageWidth=2592
    -ImageHeight=1936
    -EncodingProcess=Baseline DCT, Huffman coding
    -BitsPerSample=8
    -ColorComponents=3
    -YCbCrSubSampling=YCbCr4:2:0 (2 2)
    -Aperture=2.8
    -ImageSize=2592x1936
    -Megapixels=5.0
    -ShutterSpeed=1/15
    -ThumbnailImage=(Binary data 13456 bytes, use -b option to extract)
    -GPSAltitude=310.3 m Above Sea Level
    -GPSLatitude=46 deg 4' 27.00" N
    -GPSLongitude=14 deg 28' 40.80" E
    -FocalLength35efl=3.9 mm
    -GPSPosition=46 deg 4' 27.00" N, 14 deg 28' 40.80" E
    -LightValue=6.6

    exiftool -Make="Samsung" lovecnabiralec.jpg 
    1 image files updated

    exiftool -args lovecnabiralec.jpg

    -ExifToolVersion=12.16
    -FileName=lovecnabiralec.jpg
    -Directory=.
    -FileSize=2.6 MiB
    -FileModifyDate=2023:03:13 16:16:50+01:00
    -FileAccessDate=2023:03:13 16:16:50+01:00
    -FileInodeChangeDate=2023:03:13 16:16:50+01:00
    -FilePermissions=rw-r--r--
    -FileType=JPEG
    -FileTypeExtension=jpg
    -MIMEType=image/jpeg
    -ExifByteOrder=Little-endian (Intel, II)
    -Make=Samsung
    -Model=iPhone 4
    -Orientation=Rotate 90 CW
    -XResolution=72
    -YResolution=72
    -ResolutionUnit=inches
    -Software=5.0.1
    -ModifyDate=2012:01:25 14:50:25
    -YCbCrPositioning=Centered
    -ExposureTime=1/15
    -FNumber=2.8
    -ExposureProgram=Program AE
    -ISO=125
    -ExifVersion=0221
    -DateTimeOriginal=2012:01:25 14:50:25
    -CreateDate=2012:01:25 14:50:25
    -ComponentsConfiguration=Y, Cb, Cr, -
    -ShutterSpeedValue=1/15
    -ApertureValue=2.8
    -BrightnessValue=2.281069959
    -MeteringMode=Multi-segment
    -Flash=No Flash
    -FocalLength=3.9 mm
    -FlashpixVersion=0100
    -ColorSpace=sRGB
    -ExifImageWidth=2592
    -ExifImageHeight=1936
    -SensingMethod=One-chip color area
    -CustomRendered=HDR (no original saved)
    -ExposureMode=Auto
    -WhiteBalance=Auto
    -SceneCaptureType=Standard
    -GPSLatitudeRef=North
    -GPSLongitudeRef=East
    -GPSAltitudeRef=Above Sea Level
    -GPSTimeStamp=14:16:01
    -GPSImgDirectionRef=True North
    -GPSImgDirection=180.9357143
    -Compression=JPEG (old-style)
    -ThumbnailOffset=884
    -ThumbnailLength=13456
    -ImageWidth=2592
    -ImageHeight=1936
    -EncodingProcess=Baseline DCT, Huffman coding
    -BitsPerSample=8
    -ColorComponents=3
    -YCbCrSubSampling=YCbCr4:2:0 (2 2)
    -Aperture=2.8
    -ImageSize=2592x1936
    -Megapixels=5.0
    -ShutterSpeed=1/15
    -ThumbnailImage=(Binary data 13456 bytes, use -b option to extract)
    -GPSAltitude=310.3 m Above Sea Level
    -GPSLatitude=46 deg 4' 27.00" N
    -GPSLongitude=14 deg 28' 40.80" E
    -FocalLength35efl=3.9 mm
    -GPSPosition=46 deg 4' 27.00" N, 14 deg 28' 40.80" E
    -LightValue=6.6

    ls

    blinkenlichten	blinkenlichten.odt  JFP_5195.NEF  lovecnabiralec.exv  lovecnabiralec.html  lovecnabiralec.jpg  lovecnabiralec.jpg_original

    exiftool -restore_original lovecnabiralec.jpg

    ls

    blinkenlichten	blinkenlichten.odt  JFP_5195.NEF  lovecnabiralec.exv  lovecnabiralec.html  lovecnabiralec.jpg

    exiftool lovecnabiralec.jpg

    ExifTool Version Number         : 12.16
    File Name                       : lovecnabiralec.jpg
    Directory                       : .
    File Size                       : 2.6 MiB
    File Modification Date/Time     : 2023:03:13 15:53:09+01:00
    File Access Date/Time           : 2023:03:13 16:24:38+01:00
    File Inode Change Date/Time     : 2023:03:13 16:24:37+01:00
    File Permissions                : rw-r--r--
    File Type                       : JPEG
    File Type Extension             : jpg
    MIME Type                       : image/jpeg
    Exif Byte Order                 : Little-endian (Intel, II)
    Make                            : Apple
    Camera Model Name               : iPhone 4
    Orientation                     : Rotate 90 CW
    X Resolution                    : 72
    Y Resolution                    : 72
    Resolution Unit                 : inches
    Software                        : 5.0.1
    Modify Date                     : 2012:01:25 14:50:25
    Y Cb Cr Positioning             : Centered
    Exposure Time                   : 1/15
    F Number                        : 2.8
    Exposure Program                : Program AE
    ISO                             : 125
    Exif Version                    : 0221
    Date/Time Original              : 2012:01:25 14:50:25
    Create Date                     : 2012:01:25 14:50:25
    Components Configuration        : Y, Cb, Cr, -
    Shutter Speed Value             : 1/15
    Aperture Value                  : 2.8
    Brightness Value                : 2.281069959
    Metering Mode                   : Multi-segment
    Flash                           : No Flash
    Focal Length                    : 3.9 mm
    Flashpix Version                : 0100
    Color Space                     : sRGB
    Exif Image Width                : 2592
    Exif Image Height               : 1936
    Sensing Method                  : One-chip color area
    Custom Rendered                 : HDR (no original saved)
    Exposure Mode                   : Auto
    White Balance                   : Auto
    Scene Capture Type              : Standard
    GPS Latitude Ref                : North
    GPS Longitude Ref               : East
    GPS Altitude Ref                : Above Sea Level
    GPS Time Stamp                  : 14:16:01
    GPS Img Direction Ref           : True North
    GPS Img Direction               : 180.9357143
    Compression                     : JPEG (old-style)
    Thumbnail Offset                : 882
    Thumbnail Length                : 13456
    Image Width                     : 2592
    Image Height                    : 1936
    Encoding Process                : Baseline DCT, Huffman coding
    Bits Per Sample                 : 8
    Color Components                : 3
    Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
    Aperture                        : 2.8
    Image Size                      : 2592x1936
    Megapixels                      : 5.0
    Shutter Speed                   : 1/15
    Thumbnail Image                 : (Binary data 13456 bytes, use -b option to extract)
    GPS Altitude                    : 310.3 m Above Sea Level
    GPS Latitude                    : 46 deg 4' 27.00" N
    GPS Longitude                   : 14 deg 28' 40.80" E
    Focal Length                    : 3.9 mm
    GPS Position                    : 46 deg 4' 27.00" N, 14 deg 28' 40.80" E
    Light Value                     : 6.6

The metadata in the `blinkenlichten.odt` document can be modified directly in the `meta.xml` file and then pack all files into `blinkenlichten.zip` using the `zip` tool and rename the result to `blinkenlichten.odt`.

    exiftool blinkenlichten.odt

    ExifTool Version Number         : 12.16
    File Name                       : blinkenlichten.odt
    Directory                       : .
    File Size                       : 11 KiB
    File Modification Date/Time     : 2023:03:13 11:14:39+01:00
    File Access Date/Time           : 2023:03:13 11:15:40+01:00
    File Inode Change Date/Time     : 2023:03:13 11:14:39+01:00
    File Permissions                : rw-r--r--
    File Type                       : ODT
    File Type Extension             : odt
    MIME Type                       : application/vnd.oasis.opendocument.text
    Initial-creator                 : Pišta Bači
    Creation-date                   : 2019:04:16 21:06:40.274118679
    Date                            : 2019:04:16 22:13:49.342982850
    Creator                         : Franko Frkič
    Editing-duration                : PT33S
    Editing-cycles                  : 3
    Generator                       : LibreOffice/6.1.3.2$Linux_X86_64 LibreOffice_project/10$Build-2
    Document-statistic Table-count  : 0
    Document-statistic Image-count  : 0
    Document-statistic Object-count : 0
    Document-statistic Page-count   : 1
    Document-statistic Paragraph-count: 6
    Document-statistic Word-count   : 44
    Document-statistic Character-count: 344
    Document-statistic Non-whitespace-character-count: 305
    Preview PNG                     : (Binary data 2855 bytes, use -b option to extract)

Let's check if we already have the `blinkenlichten.odt` document unpacked in the `blinkenlichten` folder, if not then repeat the steps from 2 subtasks. Open the `meta.xml` file and correct the desired metadata in it and save the file. Now we pack all the document files into `blinken.odt`.

    ls blinkenlichten

    Configurations2  manifest.rdf  meta.xml  settings.xml  Thumbnails
    content.xml	 META-INF      mimetype  styles.xml

    nano blinkenlichten/meta.xml

    apt update
    apt install zip

    cd blinkenlichten
    zip -r blinken.odt .

    exiftool blinken.odt

    ExifTool Version Number         : 12.16
    File Name                       : blinken.odt
    Directory                       : .
    File Size                       : 12 KiB
    File Modification Date/Time     : 2023:03:14 11:10:30+01:00
    File Access Date/Time           : 2023:03:14 11:11:08+01:00
    File Inode Change Date/Time     : 2023:03:14 11:10:30+01:00
    File Permissions                : rw-r--r--
    File Type                       : ODT
    File Type Extension             : odt
    MIME Type                       : application/vnd.oasis.opendocument.text
    Initial-creator                 : Janez Novak
    Creation-date                   : 2019:04:16 21:06:40.274118679
    Date                            : 2019:04:16 22:13:49.342982850
    Creator                         : Franko Frkič
    Editing-duration                : PT33S
    Editing-cycles                  : 3
    Generator                       : LibreOffice/6.1.3.2$Linux_X86_64 LibreOffice_project/10$Build-2
    Document-statistic Table-count  : 0
    Document-statistic Image-count  : 0
    Document-statistic Object-count : 0
    Document-statistic Page-count   : 1
    Document-statistic Paragraph-count: 6
    Document-statistic Word-count   : 44
    Document-statistic Character-count: 344
    Document-statistic Non-whitespace-character-count: 305
    Preview PNG                     : (Binary data 2855 bytes, use -b option to extract)

### 4. Program for manipulating EXIF metadata in images

We will write a short program in the [`Python`](https://docs.python.org/3/tutorial/index.html) programming language that reads an image and its EXIF metadata, changes one and deletes the other.

    lovecnabiralec.jpg

    ExifTool Version Number         : 12.16
    File Name                       : lovecnabiralec.jpg
    Directory                       : .
    File Size                       : 2.6 MiB
    File Modification Date/Time     : 2023:03:13 15:53:09+01:00
    File Access Date/Time           : 2023:03:13 16:24:38+01:00
    File Inode Change Date/Time     : 2023:03:13 16:24:37+01:00
    File Permissions                : rw-r--r--
    File Type                       : JPEG
    File Type Extension             : jpg
    MIME Type                       : image/jpeg
    Exif Byte Order                 : Little-endian (Intel, II)
    Make                            : Apple
    Camera Model Name               : iPhone 4
    Orientation                     : Rotate 90 CW
    X Resolution                    : 72
    Y Resolution                    : 72
    Resolution Unit                 : inches
    Software                        : 5.0.1
    Modify Date                     : 2012:01:25 14:50:25
    Y Cb Cr Positioning             : Centered
    Exposure Time                   : 1/15
    F Number                        : 2.8
    Exposure Program                : Program AE
    ISO                             : 125
    Exif Version                    : 0221
    Date/Time Original              : 2012:01:25 14:50:25
    Create Date                     : 2012:01:25 14:50:25
    Components Configuration        : Y, Cb, Cr, -
    Shutter Speed Value             : 1/15
    Aperture Value                  : 2.8
    Brightness Value                : 2.281069959
    Metering Mode                   : Multi-segment
    Flash                           : No Flash
    Focal Length                    : 3.9 mm
    Flashpix Version                : 0100
    Color Space                     : sRGB
    Exif Image Width                : 2592
    Exif Image Height               : 1936
    Sensing Method                  : One-chip color area
    Custom Rendered                 : HDR (no original saved)
    Exposure Mode                   : Auto
    White Balance                   : Auto
    Scene Capture Type              : Standard
    GPS Latitude Ref                : North
    GPS Longitude Ref               : East
    GPS Altitude Ref                : Above Sea Level
    GPS Time Stamp                  : 14:16:01
    GPS Img Direction Ref           : True North
    GPS Img Direction               : 180.9357143
    Compression                     : JPEG (old-style)
    Thumbnail Offset                : 882
    Thumbnail Length                : 13456
    Image Width                     : 2592
    Image Height                    : 1936
    Encoding Process                : Baseline DCT, Huffman coding
    Bits Per Sample                 : 8
    Color Components                : 3
    Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
    Aperture                        : 2.8
    Image Size                      : 2592x1936
    Megapixels                      : 5.0
    Shutter Speed                   : 1/15
    Thumbnail Image                 : (Binary data 13456 bytes, use -b option to extract)
    GPS Altitude                    : 310.3 m Above Sea Level
    GPS Latitude                    : 46 deg 4' 27.00" N
    GPS Longitude                   : 14 deg 28' 40.80" E
    Focal Length                    : 3.9 mm
    GPS Position                    : 46 deg 4' 27.00" N, 14 deg 28' 40.80" E
    Light Value                     : 6.6

    apt install pip
    pip install piexif
    pip install Pillow

    nano exifprogram.py

    #!/usr/bin/python3

    import piexif
    from PIL import Image

    # Open the image.
    img = Image.open("lovecnabiralec.jpg")

    # Extract EXIF metadata from the image.
    exif_dict= piexif.load(img.info["exif"])

    # Print out a single EXIF metadata datapoint.
    altitude  = exif_dict["GPS"][piexif.GPSIFD.GPSAltitude]
    print("Altitude: ", altitude[0]/altitude[1], altitude)

    # Modify a single EXIF metadata datapoint.
    print("Modifying the Altitude to (140,1)...")
    exif_dict["GPS"][piexif.GPSIFD.GPSAltitude] = (140,1)

    # Remove a single EXIF metadata datapoint.
    print("Removing the Make EXIF metadata field...")
    del exif_dict["0th"][271]

    # Convert the EXIF metadata to byte array.
    exif_bytes = piexif.dump(exif_dict)

    # Create and save the image with new EXIF metadata.
    img.save('%s' % "lovecnabiralec2.jpg", "jpeg", exif=exif_bytes)

    chmod +x exifprogram.py
    ./exifprogram.py 
    Altitude:  310.385593220339 (73251, 236)
    Modifying the Altitude to (140,1)...
    Removing the Make EXIF metadata field...

    exiftool lovecnabiralec2.jpg

    ExifTool Version Number         : 12.16
    File Name                       : lovecnabiralec2.jpg
    Directory                       : .
    File Size                       : 1050 KiB
    File Modification Date/Time     : 2023:03:14 14:49:06+01:00
    File Access Date/Time           : 2023:03:14 14:49:07+01:00
    File Inode Change Date/Time     : 2023:03:14 14:49:06+01:00
    File Permissions                : rw-r--r--
    File Type                       : JPEG
    File Type Extension             : jpg
    MIME Type                       : image/jpeg
    JFIF Version                    : 1.01
    Exif Byte Order                 : Big-endian (Motorola, MM)
    Camera Model Name               : iPhone 4
    Orientation                     : Rotate 90 CW
    X Resolution                    : 72
    Y Resolution                    : 72
    Resolution Unit                 : inches
    Software                        : 5.0.1
    Modify Date                     : 2012:01:25 14:50:25
    Y Cb Cr Positioning             : Centered
    Exposure Time                   : 1/15
    F Number                        : 2.8
    Exposure Program                : Program AE
    ISO                             : 125
    Exif Version                    : 0221
    Date/Time Original              : 2012:01:25 14:50:25
    Create Date                     : 2012:01:25 14:50:25
    Components Configuration        : Y, Cb, Cr, -
    Shutter Speed Value             : 1/15
    Aperture Value                  : 2.8
    Brightness Value                : 2.281069959
    Metering Mode                   : Multi-segment
    Flash                           : No Flash
    Focal Length                    : 3.9 mm
    Flashpix Version                : 0100
    Color Space                     : sRGB
    Exif Image Width                : 2592
    Exif Image Height               : 1936
    Sensing Method                  : One-chip color area
    Custom Rendered                 : HDR (no original saved)
    Exposure Mode                   : Auto
    White Balance                   : Auto
    Scene Capture Type              : Standard
    GPS Latitude Ref                : North
    GPS Longitude Ref               : East
    GPS Altitude Ref                : Above Sea Level
    GPS Time Stamp                  : 14:16:01
    GPS Img Direction Ref           : True North
    GPS Img Direction               : 180.9357143
    Compression                     : JPEG (old-style)
    Thumbnail Offset                : 873
    Thumbnail Length                : 13456
    Image Width                     : 2592
    Image Height                    : 1936
    Encoding Process                : Baseline DCT, Huffman coding
    Bits Per Sample                 : 8
    Color Components                : 3
    Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
    Aperture                        : 2.8
    Image Size                      : 2592x1936
    Megapixels                      : 5.0
    Shutter Speed                   : 1/15
    Thumbnail Image                 : (Binary data 13456 bytes, use -b option to extract)
    GPS Altitude                    : 140 m Above Sea Level
    GPS Latitude                    : 46 deg 4' 27.00" N
    GPS Longitude                   : 14 deg 28' 40.80" E
    Focal Length                    : 3.9 mm
    GPS Position                    : 46 deg 4' 27.00" N, 14 deg 28' 40.80" E
    Light Value                     : 6.6
