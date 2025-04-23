# 9. Lab: Mobile Device Forensics

## Instructions

1. Connect the mobile device and find data describing the user's activity.

## More information

## Detailed instructions

### 1. Mobile device forensics

In most cases, mobile devices run customized versions of operating systems that may be based on the Linux operating system ([Android](https://en.wikipedia.org/wiki/Android_(operating_system)), [Ubuntu Touch](https://en.wikipedia.org/wiki/Ubuntu_Touch), [postmarketOS](https://en.wikipedia.org/wiki/PostmarketOS), [SailfishOS](https://en.wikipedia.org/wiki/Sailfish_OS), [Mobian](https://mobian-project.org/), [PureOS](https://en.wikipedia.org/wiki/PureOS), [Plasma Mobile](https://en.wikipedia.org/wiki/Plasma_Mobile)), but all but Android have problems due to non-existent support from manufacturers.

The second most common mobile operating system is Apple's [iOS](https://en.wikipedia.org/wiki/IOS), which is based on the [macOS](https://en.wikipedia.org/wiki/macOS). Unfortunately, we will not look at iOS mobile device forensics in the lab, but it uses the same procedures as with other mobile device operating systems, of course with some specifics.

Mobile devices to access the mobile network usually also need a [SIM card (Subscriber Identification Module)](https://en.wikipedia.org/wiki/SIM_card), which is an independent computer of its kind and serves to identify and authenticate the user. Securely stores the [International Mobile Subscriber Identity (IMSI)](https://en.wikipedia.org/wiki/International_mobile_subscriber_identity) and the associated data encryption key. It can also store a limited number of phone numbers and short messages for the purpose of easy transfer between mobile phones, but it is no longer used for this in practice today, as this data is stored on the phone or in the cloud. To read SIM cards we need a dedicated reader and dedicated software, for example [reader](https://www.ladyada.net/make/simreader/) and [pySIM](https://github.com/twhiteman/pySIM).

Each mobile device also has a unique [serial number (International Mobile Equipment Identity (IMEI))](https://en.wikipedia.org/wiki/International_Mobile_Equipment_Identity).

First, we'll look at the [N900](https://en.wikipedia.org/wiki/Nokia_N900) mobile phone, which uses the [Maemo 5](https://en.wikipedia.org/wiki/Maemo) operating system, which works like a general Linux operating system and supports the commands and tools we've looked at in the exercises so far. Most user data is stored in the [Berkeley DB](https://en.wikipedia.org/wiki/Berkeley_DB) format and can be accessed using the [db-util](https://www.unix.com/man-page/linux/1/dbutil/). You can get an image of the N900 phone memory [here](https://polaris.fri.uni-lj.si/georgepicobello.tgz). The image can be downloaded with [`wget`](https://linux.die.net/man/1/wget) and then opened with [`tar`](https://linux.die.net/man/1/tar). Now let's install the `db-util` database inspection tool through our operating system's package manager. Then we list all files ending in `.db` with the [`find`](https://linux.die.net/man/1/find) command. The individual database is then printed with the `db_dump` command.


    wget https://polaris.fri.uni-lj.si/georgepicobello.tgz

    tar -zxvf georgepicobello.tgz

    apt update
    apt install db-util

    cd user

    find . -type f -name "*.db"

    ./user/.intellisyncd/isync_mailstore.db
    ./user/.feedservice2/facebook/facebook.db
    ./user/.feedservice2/associatedpress/ap.db
    ./user/.feedservice2/amazon/amazon.db
    ./user/.local/share/tracker/data/common.db
    ./user/.thumbnails/meta.db
    ./user/.FBReader/network.db
    ./user/.FBReader/state.db
    ./user/.FBReader/books.db
    ./user/.maemo-mapper/paths.db
    ./user/.rtcom-messaging-ui/draft.db
    ./user/.mozilla/rtcom/cert8.db
    ./user/.mozilla/rtcom/secmod.db
    ./user/.mozilla/rtcom/key3.db
    ./user/.mozilla/nokia-maps/cert8.db
    ./user/.mozilla/nokia-maps/secmod.db
    ./user/.mozilla/nokia-maps/key3.db
    ./user/.mozilla/microb/cert8.db
    ./user/.mozilla/microb/secmod.db
    ./user/.mozilla/microb/key3.db
    ./user/.feedservice/associated-press/ap.db
    ./user/.feedservice/amazon/amazon.db
    ./user/fuelpad.db
    ./user/.mafw.db
    ./user/.modest/cache/cert8.db
    ./user/.modest/cache/secmod.db
    ./user/.modest/cache/camel-cert.db
    ./user/.modest/cache/key3.db
    ./user/.dreamremote/DreamRemote.db
    ./user/.fmms/mms.db
    ./user/.osso-abook/db/index_email.db
    ./user/.osso-abook/db/index_first_last.db
    ./user/.osso-abook/db/addressbook.db
    ./user/.osso-abook/db/index_full_name.db
    ./user/.osso-abook/db/index_im_jabber.db
    ./user/.osso-abook/db/index_nick.db
    ./user/.osso-abook/db/index_phone.db
    ./user/.osso-abook/db/fre1.changes.db
    ./user/.osso-abook/db/running_id.db
    ./user/.osso-abook/db/index_last_first.db
    ./user/MyDocs/.maps/poi.db
    ./user/MyDocs/.symfonie-library.sqlite.db
    ./user/.osso/tutorial/tutorial/cert8.db
    ./user/.osso/tutorial/tutorial/secmod.db
    ./user/.osso/tutorial/tutorial/key3.db
    ./user/.geeps/cache/persistent/WebpageIcons.db
    ./user/.rtcom-eventlogger/el.db
    ./user/.rtcom-eventlogger/el-v1.db
    ./user/.rtcom-eventlogger/el-v1-before-restore.db
    ./user/.agtl/caches.db
    ./user/.radars.db
    ./user/.cache/telepathy/gabble/caps-cache.db
    ./user/.cache/tracker/file-contents.db
    ./user/.cache/tracker/file-index.db
    ./user/.cache/tracker/email-contents.db
    ./user/.cache/tracker/email-meta.db
    ./user/.cache/tracker/file-meta.db
    ./user/.cache/tracker/email-index.db
    ./user/.config/hildon-desktop/notifications.db
    ./user/.config/Nokia/QtServiceFramework_4.7_user.db
    ./user/.config/Nokia/QtServiceFramework_4.6_user.db
    ./user/.topos.db

    db_dump -p ./user/.osso-abook/db/addressbook.db

    VERSION=3
    format=print
    type=hash
    db_pagesize=4096
    HEADER=END
    105\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aUID:105\0d\0aN:Team;RapidShare AG - Support\0d\0aFN:RapidShare AG - Support Team\0d\0aORG:RapidShare AG\0d\0aTITLE:Support\0d\0aTEL;TYPE=WORK;TYPE=VOICE:+41 41 748 78 80\0d\0aADR;TYPE=WORK:;;Schochenm\c3\bchlestrasse 6;Baar;;6340;Switzerland\0d\0aURL;TYPE=WORK:http://rapidshare.com\0d\0aEMAIL;TYPE=PREF;TYPE=INTERNET:support@rapidshare.com\0d\0aREV:20110119T145706Z\0d\0aEND:VCARD\00
    109\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:26Z\0d\0aUID:109\0d\0aTEL:0684987407\0d\0aN:Fits;Andy;\0d\0aFN:Andy Fits\0d\0aEND:VCARD\00
    112\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:27Z\0d\0aUID:112\0d\0aTEL:00393202744611\0d\0aN:;Esam;\0d\0aFN:Esam\0d\0aEND:VCARD\00
    116\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:29Z\0d\0aUID:116\0d\0aTEL:+31619991327\0d\0aN:Montr;Hatm;\0d\0aFN:Hatm Montr\0d\0aEND:VCARD\00
    123\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:31Z\0d\0aUID:123\0d\0aTEL:09009596\0d\0aN:;Klantenservice;\0d\0aFN:Klantenservice\0d\0aEND:VCARD\00
    127\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aUID:127\0d\0aREV:2011-09-06T00:21:22Z\0d\0aTEL:0649260789\0d\0aN:;Renzo;\0d\0aFN:Renzo\0d\0aEND:VCARD\00
    130\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-04-01T09:20:16Z\0d\0aUID:130\0d\0aTEL;TYPE=CELL,VOICE:+201226537847\0d\0aFN:Mama\0d\0aN:;Mama;;;\0d\0aEND:VCARD\00
    134\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-04-04T10:10:17Z\0d\0aUID:134\0d\0aN:;Ebrahim owaza;;;\0d\0aTEL;TYPE=CELL:+31647282888\0d\0aX-CLASS:private\0d\0aEND:VCARD\00
    104\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2011-09-04T04:40:16Z\0d\0aUID:104\0d\0aFN:Gogofo2\0d\0aN:;Gogofo2;;;\0d\0aEND:VCARD\00
    106\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aUID:106\0d\0aN:Team;RapidShare AG - Support\0d\0aFN:RapidShare AG - Support Team\0d\0aORG:RapidShare AG\0d\0aTITLE:Support\0d\0aTEL;TYPE=WORK;TYPE=VOICE:+41 41 748 78 80\0d\0aADR;TYPE=WORK:;;Schochenm\c3\bchlestrasse 6;Baar;;6340;Switzerland\0d\0aURL;TYPE=WORK:http://rapidshare.com\0d\0aEMAIL;TYPE=PREF;TYPE=INTERNET:support@rapidshare.com\0d\0aREV:20110119T145706Z\0d\0aEND:VCARD\00
    108\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:25Z\0d\0aUID:108\0d\0aTEL:+31617027585\0d\0aN:N900;Nokia;\0d\0aFN:Nokia N900\0d\0aEND:VCARD\00
    111\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:26Z\0d\0aUID:111\0d\0aTEL:+31641760230\0d\0aN:;Laura;\0d\0aFN:Laura\0d\0aEND:VCARD\00
    113\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:27Z\0d\0aUID:113\0d\0aTEL:0206360694\0d\0aN:;Hausarts;\0d\0aFN:Hausarts\0d\0aEND:VCARD\00
    115\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:28Z\0d\0aUID:115\0d\0aTEL:0648097211\0d\0aN:;Daniella;\0d\0aFN:Daniella\0d\0aEND:VCARD\00
    117\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:30Z\0d\0aUID:117\0d\0aTEL:+31621993263\0d\0aN:Ferf;Samy;\0d\0aFN:Samy Ferf\0d\0aEND:VCARD\00
    119\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:30Z\0d\0aUID:119\0d\0aTEL:+31684979689\0d\0aN:Motsbshi;Memo;\0d\0aFN:Memo Motsbshi\0d\0aEND:VCARD\00
    120\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:30Z\0d\0aUID:120\0d\0aTEL:0206622931\0d\0aN:Devries;Willm;\0d\0aFN:Willm Devries\0d\0aEND:VCARD\00
    122\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:31Z\0d\0aUID:122\0d\0aTEL:0645091980\0d\0aN:2010;Anwary Golf;\0d\0aFN:Anwary Golf 2010\0d\0aEND:VCARD\00
    124\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:32Z\0d\0aUID:124\0d\0aTEL:+31206323845\0d\0aN:2;Soria;\0d\0aFN:Soria 2\0d\0aEND:VCARD\00
    126\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:33Z\0d\0aUID:126\0d\0aTEL:+31626001233\0d\0aN:;Voicemail;\0d\0aFN:Voicemail\0d\0aEND:VCARD\00
    128\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T12:32:47Z\0d\0aUID:128\0d\0aTEL;TYPE=CELL,VOICE:+31306344055\0d\0aFN:Verzekning auto\0d\0aN:;Verzekning auto;;;\0d\0aEND:VCARD\00
    131\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-04-01T09:35:00Z\0d\0aUID:131\0d\0aTEL;TYPE=CELL,VOICE:+201116434812\0d\0aFN:Baba\0d\0aN:;Baba;;;\0d\0aEND:VCARD\00
    133\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-04-03T14:42:32Z\0d\0aUID:133\0d\0aTEL;TYPE=CELL,VOICE:+31651930272\0d\0aFN:Lpg rashid\0d\0aN:;Lpg rashid;;;\0d\0aEND:VCARD\00
    107\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:25Z\0d\0aUID:107\0d\0aTEL:0206240363\0d\0aN:Huur;Huis Te;\0d\0aFN:Huis Te Huur\0d\0aEND:VCARD\00
    110\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:26Z\0d\0aUID:110\0d\0aTEL:0627123321\0d\0aN:Pelardo;Samer;\0d\0aFN:Samer Pelardo\0d\0aEND:VCARD\00
    114\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:27Z\0d\0aUID:114\0d\0aTEL:0685253503\0d\0aN:;Soria;\0d\0aFN:Soria\0d\0aEND:VCARD\00
    118\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:30Z\0d\0aUID:118\0d\0aTEL:+31206976337\0d\0aN:klantenserv;Int;\0d\0aFN:Int klantenserv\0d\0aEND:VCARD\00
    121\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:31Z\0d\0aUID:121\0d\0aTEL:+31654347999\0d\0aN:;Piatar;\0d\0aFN:Piatar\0d\0aEND:VCARD\00
    125\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-30T08:49:33Z\0d\0aUID:125\0d\0aTEL:0619478366\0d\0aN:Band;Timo;\0d\0aFN:Timo Band\0d\0aEND:VCARD\00
    129\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-03-31T11:30:14Z\0d\0aUID:129\0d\0aTEL;TYPE=CELL,VOICE:+31686436053\0d\0aFN:Ashrf shosho\0d\0aN:;Ashrf shosho;;;\0d\0aEND:VCARD\00
    132\00
    BEGIN:VCARD\0d\0aVERSION:3.0\0d\0aREV:2012-04-01T12:42:33Z\0d\0aUID:132\0d\0aFN:Nermine Shafan\0d\0aTEL;TYPE=CELL,VOICE:+201273098664\0d\0aX-SKYPE;X-OSSO-VALID=yes;TYPE=skype:nermine.shafan\0d\0aNICKNAME:Nermine Shafan\0d\0aN:;Nermine Shafan\0d\0aEND:VCARD\00
    DATA=END

The Android mobile operating system has certain data hidden and also uses encryption for the entire disk. If we want to access all the data, we have to become a super user or `root`. Certain companies allow us to do this, but not by default, and for other devices we need to know some security flaw that we exploit to become a super user. [ADB - Android Debug Bridge](https://developer.android.com/tools/adb) is used to work with Android devices. The tool consists of two parts - the first is the service that runs on the phone (`adb`) and the second is the tool that talks to the service on the phone (`adb`) via USB. The `adb shell` can be opened on the phone using the `adb` tool. If `adbd` is running as superuser, you can get full access to the phone with the help of which you can read all the data. One way to achieve this is described in [here](https://wiki.mozilla.org/Mobile/Fennec/Android/Rooting/adb). Once you become a super user on your Android phone, you can download all the files from your phone using the `adb pull` command. You can also get partitions captured on an Android phone [here](https://polaris.fri.uni-lj.si/samsung_ace.tgz). We can download the image with the `wget` command and then open it with the `tar` command. We now mount the resulting partitions with the `mount` command so that we can access their files.

    wget https://polaris.fri.uni-lj.si/samsung_ace.tgz

    tar -zxvf samsung_ace.tgz

    ls

    samsung_ace.tgz  stl10-cache.raw  stl11-data.raw  stl6-lfs.raw	stl9-system.raw

    mkdir stl6
    mkdir stl9
    mkdir stl10
    mkdir stl11

    mount stl6-lfs.raw ./stl6

    mount: /root/stl6: wrong fs type, bad option, bad superblock on /dev/loop0, missing codepage or helper program, or other error.

    mount stl9-system.raw ./stl9
    mount stl10-cache.raw ./stl10
    mount stl11-data.raw ./stl11

The `stl9` partition holds the operating system, where the `bin` folder contains the programs, the `lib` folder contains the system libraries, the `usr` folder contains the user parts of the installed applications, the `xbin` folder contains the tools for rooting the phone, and the `etc` folder contains the device settings.

    ls stl9/

    app	    cameradata	  CSCVersion.txt  framework  SW_Configuration.xml  xbin
    bin	    csc		  etc		  lib	     T9DB
    build.prop  CSCFiles.txt  fonts		  media      usr

Partition `stl10` contains part of temporary files and logs.

    ls stl10/

    downloadfile.bin  lost+found  recovery

In the `stl10/recovery/last_log` file, which represents the device's basic setup log, we see that the device uses the [Robust FAT File System - RFS](https://www.manualslib.com/manual/147588/Samsung-V1-3-0.html?page=9#manual) file system.

    cat stl10/recovery/last_log 

    Starting recovery on Sat Jan  1 00:02:35 2011

    [initialize ui and event]
    framebuffer: fd 4 (320 x 480)

    [collecting table information]
    load_volume_table2 :: format 'rfs16' '/sbin/fat.format -F 16 -s 8 -S 512'
    load_volume_table2 :: format 'rfs32' '/sbin/fat.format -F 32 -s 4 -S 4096'
    load_volume_table2 :: mount 'rfs_opt1' 'nosuid,noatime,nodev,nodiratime' 'check=no' 
    load_volume_table2 :: mount 'rfs_opt2' 'ro,nosuid,nodev' 'check=no' 
    recovery filesystem table
    =========================
    0 /tmp ramdisk (null) (null) '(null)' 0000 '(null)'
    1 /system rfs /dev/stl9 (null) '(null)' 0000 '(null)'
    2 /cache rfs /dev/stl10 (null) '/sbin/fat.format -F 16 -s 8 -S 512' 0000 '(null)'
    3 /sdcard vfat /dev/block/mmcblk0p1 (null) '(null)' 0000 '(null)'
    4 /data rfs /dev/stl11 (null) '/sbin/fat.format -F 16 -s 8 -S 512' 0000 '(null)'



    [collecting command]
    previous_runs = 0
    send_intent = (null)
    update_package = (null)
    wipe_data = 0
    wipe_cache = 0
    encrypted_fs_mode = (null)
    toggle_secure_fs = 0
    show_ui_text = 0
    wipe_delete_files = 0
    Command: "/sbin/recovery"


    [property list]
    ro.secure=1
    ro.allow.mock.location=0
    ro.debuggable=0
    persist.service.adb.enable=0
    ro.factorytest=0
    ro.serialno=
    ro.bootmode=unknown
    ro.baseband=unknown
    ro.carrier=unknown
    ro.bootloader=unknown
    ro.hardware=bcm21553
    ro.revision=0
    ro.emmc=0
    ro.build.id=GINGERBREAD
    ro.build.display.id=GINGERBREAD.XXMC1
    ro.build.version.incremental=XXMC1
    ro.build.version.sdk=10
    ro.build.version.codename=REL
    ro.build.version.release=2.3.6
    ro.build.date=Mon Mar 25 21:18:18 KST 2013
    ro.build.date.utc=1364213898
    ro.build.type=user
    ro.build.user=dpi
    ro.build.host=DELL236
    ro.build.tags=release-keys
    ro.product.model=GT-S5830i
    ro.product.brand=samsung
    ro.product.name=GT-S5830i
    ro.product.device=GT-S5830i
    ro.product.board=cooperve
    ro.product.cpu.abi=armeabi
    ro.product.manufacturer=samsung
    ro.product.locale.language=en
    ro.product.locale.region=GB
    ro.wifi.channels=
    ro.board.platform=bcm21553
    ro.build.product=GT-S5830i
    ro.build.description=GT-S5830i-user 2.3.6 GINGERBREAD XXMC1 release-keys
    ro.build.fingerprint=samsung/GT-S5830i/GT-S5830i:2.3.6/GINGERBREAD/XXMC1:user/release-keys
    ro.build.PDA=S5830iXXMC1
    ro.build.hidden_ver=S5830iXXMC1
    ro.build.changelist=0000
    ro.build.buildtag=
    rild.libpath=/system/lib/libbrcm_ril.so
    rild.libargs=-d /dev/smd0
    persist.rild.nitz_plmn=
    persist.rild.nitz_long_ons_0=
    persist.rild.nitz_long_ons_1=
    persist.rild.nitz_long_ons_2=
    persist.rild.nitz_long_ons_3=
    persist.rild.nitz_short_ons_0=
    persist.rild.nitz_short_ons_1=
    persist.rild.nitz_short_ons_2=
    persist.rild.nitz_short_ons_3=
    DEVICE_PROVISIONED=1
    debug.sf.hw=1
    debug.composition.type=mdp
    dalvik.vm.heapsize=64m
    ro.sf.lcd_density=160
    ro.opengles.version=131072
    ro.media.enc.hprof.file.format=3gp
    ro.media.enc.lprof.file.format=3gp
    ro.media.enc.hprof.codec.vid=m4v
    ro.media.enc.lprof.codec.vid=h263
    ro.media.enc.hprof.codec.aud=aac
    ro.media.enc.lprof.codec.aud=amrnb
    ro.media.enc.hprof.vid.width=352
    ro.media.enc.lprof.vid.width=176
    ro.media.enc.hprof.vid.height=288
    ro.media.enc.lprof.vid.height=144
    ro.media.enc.hprof.vid.fps=30
    ro.media.enc.lprof.vid.fps=30
    ro.media.enc.hprof.vid.bps=384000
    ro.media.enc.lprof.vid.bps=192000
    ro.media.enc.hprof.aud.bps=23450
    ro.media.enc.lprof.aud.bps=23450
    ro.media.enc.hprof.aud.ch=1
    ro.media.enc.lprof.aud.ch=1
    ro.media.enc.hprof.aud.hz=8000
    ro.media.enc.lprof.aud.hz=8000
    ro.media.cam.preview.fps=15
    dalvik.vm.jniopts=warnonly
    net.streaming.rtsp.uaprof=http://wap.samsungmobile.com/uaprof/GT-S5830i.xml
    net.change=net.bt.name
    ro.kernel.qemu=0
    ro.url.legal=http://www.google.com/intl/%s/mobile/android/basic/phone-legal.html
    ro.url.legal.android_privacy=http://www.google.com/intl/%s/mobile/android/basic/privacy.html
    ro.com.google.locationfeatures=1
    ro.com.google.clientidbase=android-samsung
    ro.com.google.gmsversion=2.3_r11
    ro.setupwizard.mode=OPTIONAL
    ro.config.notification_sound=13_Whistle_Default_message.ogg
    ro.config.alarm_alert=1_Good_Morning.ogg
    alsa.mixer.playback.master=Speaker
    alsa.mixer.capture.master=Mic
    alsa.mixer.playback.earpiece=Earpiece
    alsa.mixer.capture.earpiece=Mic
    alsa.mixer.playback.headset=Headset
    alsa.mixer.capture.headset=Mic
    alsa.mixer.playback.speaker=Speaker
    alsa.mixer.capture.speaker=Mic
    alsa.mixer.playback.bt.sco=BTHeadset
    alsa.mixer.capture.bt.sco=BTHeadset
    alsa.mixer.playback.bt.a2dp=BTHeadset
    alsa.mixer.capture.bt.a2dp=BTHeadset
    dev.sfbootcomplete=0
    keyguard.no_require_sim=true
    ro.config.ringtone=Classic_bell.ogg
    +=wifi.interface=wl0.1
    ro.error.receiver.default=com.samsung.receiver.error
    net.bt.name=Android
    dalvik.vm.stack-trace-file=/data/anr/traces.txt
    init.svc.recovery=running
    init.svc.adbd=stopped


    [checking Manufacturing mode]
    check_point /efs/imei/selective
    shaman: check_point: access /
    shaman: check_point: access /efs/
    check_point: (/efs/) absent
    check_point: failed, try again rc /efs/imei/selective (0xffffffff)


    [Appling muti_csc]
    multi_csc: found /proc/LinuStoreIII/efs_info 
    multi_csc :: code '/system/csc/SIM
    /system/'
    multi_csc : warning : last byte is not  readable symbol 0x0a

    0x00000000 53 49 4d 0a 00 00 00 00       00 00 00 00 00 00 00 00
    0x00000010 00 00 00 00 00 00 00 00       00 00 00 00 00 00 00 00
    0x00000020 00 00 00 00 00 00 00 00       00 00 00 00 00 00 00 00
    0x00000030 00 00 00 00 00 00 00 00       00 00 00 00 00 00 00 00
    0x00000040 00 00 00 00 00 00 00 00       00 00 00 00 00 00 00 00
    0x00000050 00 00 00 00 00 00 00 00       00 00 00 00 00 00 00 00
    0x00000060 00 00 00 00
    multi_csc : warning : correction

    0x00000000 53 49 4d 00 00 00 00 00       00 00 00 00 00 00 00 00
    0x00000010 00 00 00 00 00 00 00 00       00 00 00 00 00 00 00 00
    0x00000020 00 00 00 00 00 00 00 00       00 00 00 00 00 00 00 00
    0x00000030 00 00 00 00 00 00 00 00       00 00 00 00 00 00 00 00
    0x00000040 00 00 00 00 00 00 00 00       00 00 00 00 00 00 00 00
    0x00000050 00 00 00 00 00 00 00 00       00 00 00 00 00 00 00 00
    0x00000060 00 00 00 00

    0x00000000 2f 73 79 73 74 65 6d 2f       63 73 63 2f 53 49 4d 2f
    0x00000010 73 79 73 74 65 6d 2f
    check_point2: /system/
    check_point2: access /
    check_point2: access /system/
    check_point2: access /system/

    /system/csc/SIM/system/:
    cp_file++ src:/system/csc/SIM/system//CSCFiles.txt dst:/system//CSCFiles.txt 
    cp_file: write to /system//CSCFiles.txt 233 bytes
    cp_file++ src:/system/csc/SIM/system//CSCVersion.txt dst:/system//CSCVersion.txt 
    cp_file: write to /system//CSCVersion.txt 13 bytes

    /system/csc/SIM/system//csc:
    check_point2: /system//csc
    check_point2: access /
    check_point2: access /system/
    check_point2: access /system//
    check_point2: access /system//csc
    cp_file++ src:/system/csc/SIM/system//csc/contents.db dst:/system//csc/contents.db 
    cp_file: write to /system//csc/contents.db 3072 bytes
    cp_file++ src:/system/csc/SIM/system//csc/customer.xml dst:/system//csc/customer.xml 
    cp_file: write to /system//csc/customer.xml 3510 bytes
    cp_file++ src:/system/csc/SIM/system//csc/others.xml dst:/system//csc/others.xml 
    cp_file: write to /system//csc/others.xml 311 bytes
    cp_file++ src:/system/csc/SIM/system//csc/sales_code.dat dst:/system//csc/sales_code.dat 
    cp_file: write to /system//csc/sales_code.dat 4 bytes
    [AJK]device_handle_key [key_code : 114] [visible : 1] 
    [AJK]device_handle_key [key_code : 114] [visible : 1] 
    [AJK]device_handle_key [key_code : 115] [visible : 1] 
    [AJK]device_handle_key [key_code : 115] [visible : 1] 
    [AJK]device_handle_key [key_code : 102] [visible : 1] 
    copy_kernel_file :: create kernel log file '/tmp/kernel.log' 
    copy_kernel_file :: create kernel log file '/cache/recovery/recovery_kernel_log'

The `stl11` partition contains user and system data. Each application has its own folder where it can store data, and each program is launched with its own user. Thus, the application is limited and does not have access to data stored by other programs. The `data` folder contains individual program folders containing [SQLite](https://en.wikipedia.org/wiki/SQLite) databases which we can read using `sqlite3` or `sqlitebrowser` tool.

    ls stl11/

    aeqcoe.txt   brcm	   dontpanic   log	      secure
    anr	     cache	   fota        lost+found     soundbooster.txt
    app	     cp_data.txt   gps	       misc	      system
    app-private  dalvik-cache  lcs.socket  property       tombstones
    backup	     data	   local       rilgps.socket  vt

    cd stl11/data

    find . | grep \\.db

    ./com.noshufou.android.su/databases/su.db
    ./com.google.android.youtube/databases/downloads.db
    ./com.sec.android.app.unifiedinbox/databases/UniboxProvider.db
    ./com.android.browser/databases/browser.db
    ./com.android.browser/databases/webviewCache.db
    ./com.android.browser/databases/webview.db
    ./com.android.browser/app_appcache/ApplicationCache.db
    ./com.android.browser/app_databases/Databases.db
    ./com.android.browser/app_geolocation/CachedGeoposition.db
    ./com.android.browser/app_geolocation/GeolocationPermissions.db
    ./com.android.browser/app_icons/WebpageIcons.db
    ./com.google.android.gm/databases/downloads.db
    ./com.google.android.gm/databases/gmail.db
    ./com.google.android.gm/databases/webview.db
    ./com.google.android.gm/databases/mailstore.friforenzik@gmail.com.db
    ./com.google.android.gm/databases/webviewCache.db
    ./com.android.email/databases/EmailProvider.db
    ./com.android.email/databases/EmailProviderBody.db
    ./com.cooliris.media/databases/picasa.db
    ./com.android.providers.userdictionary/databases/user_dict.db
    ./com.google.android.gsf/databases/googlesettings.db
    ./com.google.android.gsf/databases/gservices.db
    ./com.google.android.gsf/databases/gls.db
    ./com.google.android.gsf/databases/subscribedfeeds.db
    ./com.google.android.gsf/databases/talk.db
    ./com.google.android.gsf/databases/talk.db-mj41628A22
    ./com.sec.android.providers.downloads/databases/sisodownloads.db
    ./com.android.providers.telephony/databases/mmssms.db
    ./com.android.providers.telephony/databases/telephony.db
    ./com.android.providers.telephony/databases/nwk_info.db
    ./com.android.providers.telephony/databases/tether_dun.db
    ./com.sec.android.app.memo/databases/Memo.db
    ./com.sec.android.app.twlauncher/databases/launcher.db
    ./com.android.deskclock/databases/alarms.db
    ./com.android.providers.contacts/databases/contacts2.db
    ./com.android.providers.calendar/databases/calendar.db
    ./com.sec.android.provider.logsprovider/databases/logs.db
    ./com.google.android.googlequicksearchbox/databases/qsb-history.db
    ./com.android.settings/databases/webview.db
    ./com.android.settings/databases/webviewCache.db
    ./com.google.android.apps.maps/databases/webview.db
    ./com.google.android.apps.maps/databases/webviewCache.db
    ./com.google.android.apps.maps/databases/google_analytics.db
    ./com.google.android.apps.maps/databases/userfeedback.db
    ./com.smlds/databases/smldsdb.db
    ./com.sec.android.provider.badge/databases/badge.db
    ./com.android.vending/databases/library.db
    ./com.android.vending/databases/localappstate.db
    ./com.android.vending/databases/package_verification.db
    ./com.android.vending/databases/suggestions.db
    ./com.android.providers.settings/databases/settings.db
    ./com.android.providers.media/databases/internal.db
    ./com.android.providers.media/databases/external-4b122b9f.db
    ./com.osp.app.signin/databases/osp.db
    ./com.wssyncmldm/databases/wssdmdatabase.db
    ./com.android.providers.drm/databases/drm.db
    ./com.sec.android.providers.drm/databases/drmdatabase.db
    ./com.android.providers.downloads/databases/downloads.db
    ./com.google.android.partnersetup/databases/rlz_data.db
    ./com.facebook.lite/databases/webview.db
    ./com.facebook.lite/databases/webviewCache.db
    ./com.facebook.orca/databases/webview.db
    ./com.facebook.orca/databases/webviewCache.db

    apt update
    apt install sqlite3 sqlitebrowser

    sqlite3 com.android.providers.telephony/databases/mmssms.db

    .help

    .schema

    .tables

    addr                 part                 sr_pending         
    android_metadata     pdu                  threads            
    attachments          pending_msgs         words              
    canonical_addresses  rate                 words_content      
    drm                  raw                  words_segdir       
    mychannels           sms                  words_segments     

    select * from sms;

    1|2|668||1459156825198|0|1|-1|1|0||veljavnost tvojega bob racuna potece 31.05.2016, zato napolni svoj racun in brezskrbno uporabljaj bob storitve se naprej. cena minute, sms/mms ali MB je 6,6 centa v Sloveniji. bob|+3864044100010|0|0|1|1|
    2|3|bob||1459426618423|0|1|-1|1|0||pozdravljen-a. obvescamo te, da s 30.4.2016 stopijo v veljavo novi pogoji gostovanja v drzavah EU. hkrati se spremenijo pogoji o nacinu obvescanja ob nizkem dobroimetju. ko stanje na tvojem racunu pade pod 0,51 eur, bos o tem obvescen-a s sms sporocilom. vec info na 068 680680 in na www.bob.si. tvoj bob|+3864044100010|0|0|0|1|
    3|5|+38640792227||1460625266320|0|1|-1|1|0||Pozdravljeni, danes lahko pridem na ogled. Prosim za potrditev do 14.00 ure in naslov.
    Lep dan, Rado Jurca.|+3864044100010|0|0|0|1|
    4|6|040458386||1463398229275||1|-1|2|||Drugic daj studentom pravo telefonsko!||0|0|1|1|
    5|6|+38640458386||1463398299796|0|1|-1|1|0||Ha ha hi hi ho ho.|+3864044100010|0|0|1|1|
    6|6|+38640458386||1463398330261|0|1|-1|1|0||Mogoce jim bom raje dal kak piskot... ali nekaj podobnega.|+3864044100010|0|0|1|1|
    7|6|+38640458386||1463404603265|0|1|-1|1|0||Ej, kreten! Dvign, no!|+3864044100010|0|0|1|1|

    .quit

    sqlite3 com.android.providers.contacts/databases/contacts2.db

    .tables

    _sync_state                       settings                        
    _sync_state_metadata              speed_dial                      
    accounts                          status_updates                  
    activities                        v1_settings                     
    agg_exceptions                    view_contacts                   
    android_metadata                  view_contacts_restricted        
    calls                             view_data                       
    contact_entities_view             view_data_restricted            
    contact_entities_view_restricted  view_groups                     
    contacts                          view_raw_contacts               
    data                              view_raw_contacts_restricted    
    groups                            view_v1_contact_methods         
    mimetypes                         view_v1_extensions              
    name_lookup                       view_v1_group_membership        
    nickname_lookup                   view_v1_groups                  
    packages                          view_v1_organizations           
    phone_lookup                      view_v1_people                  
    properties                        view_v1_phones                  
    raw_contacts                      view_v1_photos 

    select * from contacts;

    1|1|||0|0|0|0|1|1|0i1||0|0|0|1|1|vnd.sec.contact.sim|0||0||0||0|
    2|2|||0|0|0|0|1|1|0i249||0|0|0|1|2|vnd.sec.contact.sim|0||0||0||0|
    3|3|||0|0|0|0|1|1|0i250||0|0|0|1|3|vnd.sec.contact.sim|0||0||0||0|
    4|4|||0|0|0|0|1|1|2238i6e2a07280ebf74c4||0|0|0|1|4|com.google|0||0||0||0|
    5|5|||0|0|0|0|1|1|2238i2e28b2820b50eb4d||0|0|0|1|5|com.google|0||0||0||0|
    6|6|||0|0|0|0|0|0|2238i153efc500fbd8d8d||0|0|1|1|6|com.google|0||0||0||0|
    7|7|||0|0|0|1|1|0|2238i5b427c9c8ac2b926||0|0|1|1|7|com.google|0||0||0||0|
    8|8|||0|0|0|0|0|0|2238i25445b56096a409b||0|0|1|1|8|com.google|0||0||0||0|
    9|9|||0|2|1463742641757|0|0|0|2238i3cda596d8ec4895c||0|0|1|1|9|com.google|0||0||0||0|
    10|10|45||0|0|0|0|0|0|2238i20c130e309806749||0|0|1|1|10|com.google|0||0||0||0|
    11|11|||0|0|0|0|1|1|2238i26bdd8020a9bcbad||0|0|0|1|11|com.google|0||0||0||0|
    13|13|||0|0|0|0|0|0|2238i5dfa6b398a24e891||0|0|1|1|13|com.google|0||0||0||0|

    select * from [view_v1_people];

    1|Dusan K.|Dusan K.|Dusan K.|||vnd.sec.contact.sim|vnd.sec.contact.sim|0|||0|0|||2|051254503|2||305452150
    2|Info|Info|Info|||vnd.sec.contact.sim|vnd.sec.contact.sim|0|||0|0|||4|090068068|2||860860090
    3|Sklenite narocnino|Sklenite narocnino|Sklenite narocnino|||vnd.sec.contact.sim|vnd.sec.contact.sim|0|||0|0|||6|080680680|2||086086080
    4|Jernej Korosec|Jernej Korosec|Korosec, Jernej|||friforenzik@gmail.com|com.google|0|||0|0|||13|041424007|2||700424140
    5|Forenzicarka 1|Forenzicarka 1|Forenzicarka 1|||friforenzik@gmail.com|com.google|0|||0|0|||20|031844429|2||924448130
    6|polz|polz|polz|||friforenzik@gmail.com|com.google|0|||0|0||26|||||
    7||polz@polz.si|polz@polz.si|||friforenzik@gmail.com|com.google|0|||0|1||32|||||
    8||matic.junk@gmail.com|matic.junk@gmail.com|||friforenzik@gmail.com|com.google|0|||0|0||38|||||
    9||polz@fri.uni-lj.si|polz@fri.uni-lj.si|||friforenzik@gmail.com|com.google|2|1463742641757||0|0||44|||||
    10||ozbolt.menegatti@gmail.com|ozbolt.menegatti@gmail.com|||friforenzik@gmail.com|com.google|0|||0|0||50|||||
    11|Luka Krsnik|Luka Krsnik|Krsnik, Luka|||friforenzik@gmail.com|com.google|0|||0|0|||57|040914122|2||221419040
    13||jan@k0s.si|jan@k0s.si|||friforenzik@gmail.com|com.google|0|||0|0||68|||||

    select * from [view_v1_phones];

    20|5|1|031844429|2||924448130|Forenzicarka 1|Forenzicarka 1|Forenzicarka 1|||friforenzik@gmail.com|com.google|0|||0|0|||20|031844429|2||924448130
    57|11|1|040914122|2||221419040|Luka Krsnik|Luka Krsnik|Krsnik, Luka|||friforenzik@gmail.com|com.google|0|||0|0|||57|040914122|2||221419040
    13|4|1|041424007|2||700424140|Jernej Korosec|Jernej Korosec|Korosec, Jernej|||friforenzik@gmail.com|com.google|0|||0|0|||13|041424007|2||700424140
    2|1|1|051254503|2||305452150|Dusan K.|Dusan K.|Dusan K.|||vnd.sec.contact.sim|vnd.sec.contact.sim|0|||0|0|||2|051254503|2||305452150
    6|3|1|080680680|2||086086080|Sklenite narocnino|Sklenite narocnino|Sklenite narocnino|||vnd.sec.contact.sim|vnd.sec.contact.sim|0|||0|0|||6|080680680|2||086086080
    4|2|1|090068068|2||860860090|Info|Info|Info|||vnd.sec.contact.sim|vnd.sec.contact.sim|0|||0|0|||4|090068068|2||860860090

    .quit

    sqlite3 com.android.browser/databases/browser.db

    .tables

    android_metadata  bookmarks         folders           searches

    select * from bookmarks;

    1|Vodafone live!|http://wap.simobil.si|0|0|0||1|�PNG
    �
    |�PNG
    �
    |||0
    2|Sign in - Google Accounts|https://accounts.google.com/ServiceLogin?continue=https%3A%2F%2Fmyaccount.google.com%2Fprivacy%2Fo%2Fads%3Futm_source%3DAndroid%26utm_campaign%3DMobileSettings&sacu=1&passive=1209600&ignoreShadow=0#Email=friforenzik%40gmail.com|1|1463394846518|0||0||||0|99
    3|https://accounts.google.com/AccountChooser?hl=en_US&Email=friforenzik%40gmail.com&continue=https%3A%2F%2Fmyaccount.google.com%2Fprivacy%2Fo%2Fads%3Futm_source%3DAndroid%26utm_campaign%3DMobileSettings|https://accounts.google.com/AccountChooser?hl=en_US&Email=friforenzik%40gmail.com&continue=https%3A%2F%2Fmyaccount.google.com%2Fprivacy%2Fo%2Fads%3Futm_source%3DAndroid%26utm_campaign%3DMobileSettings|1|1463394846524|0||0||||0|99
    4|Sign in - Google Accounts|https://accounts.google.com/AccountLoginInfo|2|1463394992373|0||0||||0|99
    5|Sign in - Google Accounts|https://accounts.google.com/signin/challenge/sl/password|4|1463395091625|0||0||||0|99
    6|https://accounts.google.com/ServiceLogin?continue=https%3A%2F%2Fmyaccount.google.com%2Fprivacy%2Fo%2Fdashboard%3Futm_source%3DAndroid%26utm_campaign%3DMobileSettings&sacu=1&passive=1209600&ignoreShadow=0#Email=friforenzik%40gmail.com|https://accounts.google.com/ServiceLogin?continue=https%3A%2F%2Fmyaccount.google.com%2Fprivacy%2Fo%2Fdashboard%3Futm_source%3DAndroid%26utm_campaign%3DMobileSettings&sacu=1&passive=1209600&ignoreShadow=0#Email=friforenzik%40gmail.com|1|1463394885412|0||0||||0|99
    7|https://accounts.google.com/AccountChooser?hl=en_US&Email=friforenzik%40gmail.com&continue=https%3A%2F%2Fmyaccount.google.com%2Fprivacy%2Fo%2Fdashboard%3Futm_source%3DAndroid%26utm_campaign%3DMobileSettings|https://accounts.google.com/AccountChooser?hl=en_US&Email=friforenzik%40gmail.com&continue=https%3A%2F%2Fmyaccount.google.com%2Fprivacy%2Fo%2Fdashboard%3Futm_source%3DAndroid%26utm_campaign%3DMobileSettings|1|1463394885409|0||0||||0|99
    8|https://accounts.google.com/AccountChooser?hl=en_US&Email=friforenzik%40gmail.com&continue=https%3A%2F%2Fmyaccount.google.com%2Fprivacy%2Fo%2Fpersonalinfo%3Futm_source%3DAndroid%26utm_campaign%3DMobileSettings|https://accounts.google.com/AccountChooser?hl=en_US&Email=friforenzik%40gmail.com&continue=https%3A%2F%2Fmyaccount.google.com%2Fprivacy%2Fo%2Fpersonalinfo%3Futm_source%3DAndroid%26utm_campaign%3DMobileSettings|1|1463394893870|0||0||||0|99
    9|Sign in - Google Accounts|https://accounts.google.com/ServiceLogin?continue=https%3A%2F%2Fmyaccount.google.com%2Fprivacy%2Fo%2Fpersonalinfo%3Futm_source%3DAndroid%26utm_campaign%3DMobileSettings&sacu=1&passive=1209600&ignoreShadow=0#Email=friforenzik%40gmail.com|1|1463394893874|0||0||||0|99
    10|Personal info & privacy|https://myaccount.google.com/privacy?utm_source=Android&utm_campaign=MobileSettings&pli=1#personalinfo|1|1463395025167|0||0||||0|99
    11|Gender|https://myaccount.google.com/gender?utm_source=Android&utm_campaign=MobileSettings&pli=1|1|1463395059884|0||0||||0|99
    12|Personal info & privacy|https://myaccount.google.com/privacy?utm_source=Android&utm_campaign=MobileSettings#personalinfo|1|1463395091625|0||0||||0|99
    13|About me|https://aboutme.google.com/|1|1463395124246|0||0||||0|99
    14|Storitve | Si.mobil|https://www.simobil.si/storitve|3|1463645687550|0||0||||0|99
    15|http://wap.simobil.si/|http://wap.simobil.si/|3|1463645687552|0||0||||0|99
    16|Web page not available|http://upyachka.com/|0|1463400008473|0||0||||1|99
    17|upyachka|upyachka|0|1463400027487|0||0||||1|99
    18|http://www.google.si/search?hl=en&redir_esc=&source=android-browser-type&v=133247963&qsubts=1463400027664&action=devloc&q=upyachka&v=133247963|http://www.google.si/search?hl=en&redir_esc=&source=android-browser-type&v=133247963&qsubts=1463400027664&action=devloc&q=upyachka&v=133247963|1|1463400028166|0||0||||0|99
    19|Web page not available|http://oldupyachka.ru/|1|1463400032633|0||0||||0|99
    20|http://www.google.si/url?q=http://oldupyachka.ru/&sa=U&ved=0ahUKEwiF1sX3xd7MAhWFbhQKHZXzDW4QFggPMAA&sig2=_cpP0t9FYjADFzhZhaKQPA&usg=AFQjCNGgMFqrpj_3VVvSasLgu9SuPvs8JQ|http://www.google.si/url?q=http://oldupyachka.ru/&sa=U&ved=0ahUKEwiF1sX3xd7MAhWFbhQKHZXzDW4QFggPMAA&sig2=_cpP0t9FYjADFzhZhaKQPA&usg=AFQjCNGgMFqrpj_3VVvSasLgu9SuPvs8JQ|1|1463400032630|0||0||||0|99
    21|Web page not available|http://oldupyachka.ru/5/|1|1463400111296|0||0||||0|99
    22|Web page not available|http://oldupyachka.ru/7/|1|1463400505740|0||0||||0|99
    23|http://kuvaton.fi/|http://kuvaton.fi/|1|1463466999224|0||0||||1|99
    24|KuvatON.com - Just some shitty pics|http://m.kuvaton.com/|1|1463466999225|0||0||||0|99
    25|KuvatON.com - Just some shitty pics|http://m.kuvaton.com/2/|1|1463467223719|0||0||||0|99
    26|KuvatON.com - Just some shitty pics|http://m.kuvaton.com/browse/42093/hop32.gif|1|1463467286713|0||0||||0|99
    27|http://m.kuvaton.com/browse/42094/autokaistalla2.jpg|http://m.kuvaton.com/browse/42094/autokaistalla2.jpg|1|1463467316740|0||0||||0|99
    28|KuvatON.com - Just some shitty pics|http://m.kuvaton.com/3/|1|1463467405634|0||0||||0|99
    29|http://m.kuvaton.com/4/|http://m.kuvaton.com/4/|1|1463467479965|0||0||||0|99
    30|KuvatON.com - Just some shitty pics|http://m.kuvaton.com/5/|1|1463467664345|0||0||||0|99
    31|http://m.kuvaton.com/browse/42062/kid_cant_go_to_jail.gif|http://m.kuvaton.com/browse/42062/kid_cant_go_to_jail.gif|1|1463467710843|0||0||||0|99
    32|http://hyves.nl/|http://hyves.nl/|1|1463645757465|0||0||||1|99
    33|http://m.hyvesgames.nl/|http://m.hyvesgames.nl/|1|1463645757468|0||0||||0|99
    34|Web page not available|http://atdmt.com/|0|1463645813707|0||0||||1|99
    35|http://weborama.fr/|http://weborama.fr/|1|1463645941280|0||0||||1|99
    36|http://www.weborama.com/|http://www.weborama.com/|1|1463645941282|0||0||||0|99
    37|http://9292ov.nl/|http://9292ov.nl/|1|1463646060376|0||0||||1|99
    38|http://m.9292.nl/|http://m.9292.nl/|1|1463646060378|0||0||||0|99
    39|Web page not available|http://xgraph.net/|0|1463646193329|0||0||||1|99
    40|GetItOn|http://getiton.com/|1|1463646322835|0||0||||1|99
    41|Adult AdWorld : Adult Advertising Network|http://adultadworld.com/|1|1463646366822|0||0||||1|99
    42|http://adultadworld.com/advertisers.html|http://adultadworld.com/advertisers.html|1|1463646381483|0||0||||0|99
    43|Web page not available|http://rts.doublepimp.com/|0|1463646457267|0||0||||1|99
    44|girlgaze.com - girlgaze Resources and Information.|http://girlgaze.com/|1|1463646488145|0||0||||1|99
    45|http://girlgaze.com/caf/?ses=Y3JlPTE0NjM2NDY0ODkmdGNpZD1naXJsZ2F6ZS5jb201NzNkNzkxOThkZjYxOC45NzUyODMwMCZma2k9ODM5NDE3MjImdGFzaz1zZWFyY2gmZG9tYWluPWdpcmxnYXplLmNvbSZzPTU0M2JlMmRhMWU1MGRiMmZlNmI4Jmxhbmd1YWdlPWVuJmFfaWQ9Mw==&query=Free%20Arab%20Dating%20Site&afdToken=CvgBChMI_NicitzlzAIVlTHTCh0XIAZcGAEgBlDw0KABULGJtANQw8_sD1C6z_wPUOL97BBQveuEG1Dj57AqUJmk3DlQu7rfOVCx3Yw6UPexpzpQ6O-zOlCdjdZ_ULery5UBUNmc7pcBUP6c7pcBUMud7pcBUIie7pcBUKvxgZoBUO-hvp0BUKSdhq8BUKzJuLkBUIrRuLkBUMSeqPcBUO6y9oYFUPnop60FUJ-RiuEGUOLcrMIHUI6ulNcHaPDQoAFxuj6UZg0WNRmCARMIp9ieitzlzAIV8APTCh1zMwHFjQFiv8otkQEj9PP4B9hYmpEBpNKEh_qigfQSGQBtOoqQAGL5vweztygKL4y37wTgjYRiiWQ|http://girlgaze.com/caf/?ses=Y3JlPTE0NjM2NDY0ODkmdGNpZD1naXJsZ2F6ZS5jb201NzNkNzkxOThkZjYxOC45NzUyODMwMCZma2k9ODM5NDE3MjImdGFzaz1zZWFyY2gmZG9tYWluPWdpcmxnYXplLmNvbSZzPTU0M2JlMmRhMWU1MGRiMmZlNmI4Jmxhbmd1YWdlPWVuJmFfaWQ9Mw==&query=Free%20Arab%20Dating%20Site&afdToken=CvgBChMI_NicitzlzAIVlTHTCh0XIAZcGAEgBlDw0KABULGJtANQw8_sD1C6z_wPUOL97BBQveuEG1Dj57AqUJmk3DlQu7rfOVCx3Yw6UPexpzpQ6O-zOlCdjdZ_ULery5UBUNmc7pcBUP6c7pcBUMud7pcBUIie7pcBUKvxgZoBUO-hvp0BUKSdhq8BUKzJuLkBUIrRuLkBUMSeqPcBUO6y9oYFUPnop60FUJ-RiuEGUOLcrMIHUI6ulNcHaPDQoAFxuj6UZg0WNRmCARMIp9ieitzlzAIV8APTCh1zMwHFjQFiv8otkQEj9PP4B9hYmpEBpNKEh_qigfQSGQBtOoqQAGL5vweztygKL4y37wTgjYRiiWQ|1|1463646500043|0||0||||0|99
    46|Web page not available|http://hardcrap.com/|0|1463646570675|0||0||||1|99
    47|Etology – Dating Traffic Provider|http://etology.com/|2|1463646651752|0||0||||1|99
    48|Online Mercuriusgids. Bedrijvengids en internet telefoongids | Mercuriusgids.nl|http://mercuriusgids.nl/|2|1463646784802|0||0||||1|99
    49|Not Found|http://tackfilm.se/|0|1463646828840|0||0||||1|99
    50|http://timoco.eu/|http://timoco.eu/|1|1463646873575|0||0||||1|99
    51|Timoco B.V.|http://business.timoco.eu/|1|1463646873576|0||0||||0|99
    52|http://startvagina.nl/|http://startvagina.nl/|1|1463646908589|0||0||||1|99
    53|http://m.startvagina.nl/|http://m.startvagina.nl/|1|1463646908591|0||0||||0|99
    54|Buienradar.nl - Weer - Actuele neerslag, weerbericht, weersverwachting, sneeuwradar en satellietbeelden|http://buienradar.nl/|1|1463646949213|0||0||||1|99
    55|http://cookiewet.buienradar.nl/?s=http%3A%2F%2Fbuienradar.nl%2F|http://cookiewet.buienradar.nl/?s=http%3A%2F%2Fbuienradar.nl%2F|1|1463646950139|0||0||||0|99
    56|http://buienradar.nl/?cookieakkoord=true|http://buienradar.nl/?cookieakkoord=true|1|1463646956658|0||0||||0|99
    57|Web page not available|http://rhinogym.nl/|0|1463647004412|0||0||||1|99
    58|InsideGamer - De grootste bron voor games, cheats, trailers en recensies|http://insidegamer.nl/|2|1463647046723|0||0||||1|99
    59|http://xxlnutrition.nl/|http://xxlnutrition.nl/|1|1463647093280|0||0||||1|99
    60|https://xxlnutrition.com/nl/nld|https://xxlnutrition.com/nl/nld|1|1463647093283|0||0||||0|99
    61|VVGi.nl / Vincent van Gogh voor geestelijke gezondheidszorg|http://vvgi.nl/|2|1463647142816|0||0||||1|99
    62|http://foreca.com/|http://foreca.com/|1|1463647168621|0||0||||1|99
    63|Weather Forecast Ljubljana - Foreca.com|http://www.foreca.com/Slovenia/Ljubljana|1|1463647168623|0||0||||0|99
    64|Chatten doe je op Kletsen.com - De gratis chat voor Nederland en België|http://kletsen.com/|1|1463647329568|0||0||||1|99
    65|Wij kopen je oude toestel - ZONZOO|http://zonzoo.nl/|1|1463647439912|0||0||||1|99
    66|Databasefout|http://gezondenzo.net/|0|1463647538975|0||0||||1|99
    67|http://noknok.tv/|http://noknok.tv/|1|1463647572916|0||0||||1|99
    68|http://lumiaconversationsuk.microsoft.com/|http://lumiaconversationsuk.microsoft.com/|1|1463647572918|0||0||||0|99
    69|http://asterpix.com/|http://asterpix.com/|1|1463647609240|0||0||||1|99
    70|http://ww38.asterpix.com/|http://ww38.asterpix.com/|1|1463647609242|0||0||||0|99
    71|http://tbidvzc.com/sk-clkrdr.php?_t=zro&_d=195fNgWY.JvV&_p=t+nTHHOtH&_pr=HEFHbAt&_v=zzGGzzztFGzzTFzFEGE&_rdfu=X55g%3ADD9fJ3Nf.%2FW4mfN4N4.JvVDgfNQvNV1BJfD%2F4moN4.4%2FV%3FfBg1NV9GPztAG%2CzTZbztT%2CzTtHTHE%2CztFF%2CztFE%2CFtFG%2CGzzG%2Cb%2Cb%2CztFT%2Cb%2CzTZEttt%2CFEtTGt%2CAZHAF%2CzFGFAzHFFTbt%2CzHEFbbbbz%2CBaY.JNUWmsXC%26Wv1Pb%26BJVPz%26%2F4oNfQomPiii.%2FW4mfN5W9fN.JvV%26r7-_Pz%26SKLoLpl-P%26e15fsvNqPT%26viBW4PHETZtGGHE%263o1sB5P%269U5fNPXmN5msCWsX%25Gh5VNsmgWCB%269Ui4%2FPvvCoimm3&_bku=X55g%3ADD3BWmfN91aQi4WBs.JvVD%3F%26QgP%25GhNx5VfvnJ%2FKE7BF838NC%25G_xgFggt5WyLiTOIy8ru0LSTpikzEzmSS4U4II%25G_9J_7Ouj%3DX76Gvxbm8ya-AMSmV7H%3DU%25GhyTfl%25G_0-KSZzuZ6kU-mpWyTnxlEtV2GL7pQ72mLg-%3DO0QHET%25G_Xe9fqWRutE7bRw%25FI%25FI|http://tbidvzc.com/sk-clkrdr.php?_t=zro&_d=195fNgWY.JvV&_p=t+nTHHOtH&_pr=HEFHbAt&_v=zzGGzzztFGzzTFzFEGE&_rdfu=X55g%3ADD9fJ3Nf.%2FW4mfN4N4.JvVDgfNQvNV1BJfD%2F4moN4.4%2FV%3FfBg1NV9GPztAG%2CzTZbztT%2CzTtHTHE%2CztFF%2CztFE%2CFtFG%2CGzzG%2Cb%2Cb%2CztFT%2Cb%2CzTZEttt%2CFEtTGt%2CAZHAF%2CzFGFAzHFFTbt%2CzHEFbbbbz%2CBaY.JNUWmsXC%26Wv1Pb%26BJVPz%26%2F4oNfQomPiii.%2FW4mfN5W9fN.JvV%26r7-_Pz%26SKLoLpl-P%26e15fsvNqPT%26viBW4PHETZtGGHE%263o1sB5P%269U5fNPXmN5msCWsX%25Gh5VNsmgWCB%269Ui4%2FPvvCoimm3&_bku=X55g%3ADD3BWmfN91aQi4WBs.JvVD%3F%26QgP%25GhNx5VfvnJ%2FKE7BF838NC%25G_xgFggt5WyLiTOIy8ru0LSTpikzEzmSS4U4II%25G_9J_7Ouj%3DX76Gvxbm8ya-AMSmV7H%3DU%25GhyTfl%25G_0-KSZzuZ6kU-mpWyTnxlEtV2GL7pQ72mLg-%3DO0QHET%25G_Xe9fqWRutE7bRw%25FI%25FI|1|1463647615087|0||0||||0|99
    72|http://secure.bidverdrd.com/performance/bdv_rd.dbm?enparms2=1982,1760197,1795754,1933,1934,3932,2112,0,0,1937,0,1764999,349729,86583,132381533709,154300001,nlx.crkivghz&ioa=0&ncm=1&bd_ref_v=www.bidvertiser.com&TREF=1&WIN_NAME=&Category=7&ownid=547692254&u_agnt=&skter=hvrtvgzigh%2Btmrgvpizn&skwdb=ooz_wvvu|http://secure.bidverdrd.com/performance/bdv_rd.dbm?enparms2=1982,1760197,1795754,1933,1934,3932,2112,0,0,1937,0,1764999,349729,86583,132381533709,154300001,nlx.crkivghz&ioa=0&ncm=1&bd_ref_v=www.bidvertiser.com&TREF=1&WIN_NAME=&Category=7&ownid=547692254&u_agnt=&skter=hvrtvgzigh%2Btmrgvpizn&skwdb=ooz_wvvu|1|1463647615099|0||0||||0|99
    73|Video Game Cheats, Codes, Cheat Codes, Walkthroughs, Guides, FAQs, Reviews, Previews, News, Videos for Xbox 360, PS3, PC, Xbox One, PS4, Wii U, 3DS, PS Vita, Wii, PS2, PSP, DS, and more from Cheat Code Central.|http://cheatcc.com/|1|1463647672040|0||0||||1|99
    74|Video Game News - Cheat Code Central|http://news.cheatcc.com/|1|1463647716139|0||0||||0|99
    75|Uitzendbureau.nl - Vacatures van uitzendbureaus|http://uitzendbureau.nl/|2|1463647752619|0||0||||1|99
    76|404 pagina niet gevonden|http://bodyresource.nl/|0|1463647859764|0||0||||1|99
    77|paper mario|paper mario|0|1463648098543|0||0||||1|99
    78|http://www.google.si/search?hl=en&redir_esc=&source=android-browser-type&v=133247963&qsubts=1463648098695&action=devloc&q=paper+mario&v=133247963|http://www.google.si/search?hl=en&redir_esc=&source=android-browser-type&v=133247963&qsubts=1463648098695&action=devloc&q=paper+mario&v=133247963|1|1463648099293|0||0||||0|99
    79|https://en.m.wikipedia.org/wiki/Paper_Mario|https://en.m.wikipedia.org/wiki/Paper_Mario|1|1463648112915|0||0||||0|99
    80|http://www.google.si/url?q=https://en.m.wikipedia.org/wiki/Paper_Mario&sa=U&ved=0ahUKEwie-fCI4uXMAhWEPRQKHbOdA_QQFggyMA4&sig2=MxtG1qG3Y3DGqveVW31i9w&usg=AFQjCNFJeEYaNA-Nh8sTBTc_q-QDUBT9ag|http://www.google.si/url?q=https://en.m.wikipedia.org/wiki/Paper_Mario&sa=U&ved=0ahUKEwie-fCI4uXMAhWEPRQKHbOdA_QQFggyMA4&sig2=MxtG1qG3Y3DGqveVW31i9w&usg=AFQjCNFJeEYaNA-Nh8sTBTc_q-QDUBT9ag|1|1463648112913|0||0||||0|99
    81|Benov kotiček — Andrejeva spletna stran|http://andro.splet.arnes.si/|1|1463648635025|0||0||||1|99
    82|http://universepeople.com/|http://universepeople.com/|1|1463648753701|0||0||||1|99
    83|http://www.afternic.com/domain/universepeople.com|http://www.afternic.com/domain/universepeople.com|1|1463648753702|0||0||||0|99
    84|Web page not available|http://universepeople.cz/|0|1463648821735|0||0||||1|99
    85|Izklop|http://izklop.com/|1|1463648849860|0||0||||1|99
    86|Izklop|http://izklop.com/?mod=mod_ext_links&act=get_links&page=2|1|1463649360066|0||0||||1|99
    87|http://izklop.com/?mod=mod_ext_links&act=get_links&page=4|http://izklop.com/?mod=mod_ext_links&act=get_links&page=4|1|1463648935570|0||0||||0|99
    88|Izklop|http://izklop.com/?mod=mod_ext_links&act=get_links&page=3|1|1463648977965|0||0||||0|99
    89|http://izklop.com/?mod=mod_ext_links&act=get_link&id=98707|http://izklop.com/?mod=mod_ext_links&act=get_link&id=98707|1|1463648986560|0||0||||0|99

    select * from searches;

    1|upyachka|1463400028141
    2|paper mario|1463648099384

    .quit

    sqlite3 com.android.providers.media/databases/internal.db

    .tables

    album_art           artist_info         audio_meta          thumbnails        
    album_info          artists             images              video             
    albums              artists_albums_map  search              videothumbnails   
    android_metadata    audio               searchhelpertitle 

    select * from audio;

    1|/system/media/audio/ringtones/Polaris.ogg|Polaris.ogg|136841|application/ogg|1293840200|1336569214|PolarisQQQQQQQQ|8348|1||1|0||1|0|0|0|Ringtone|0||1QQQQQQQQ|Samsung|1QQQQQQQQ293500348|Samsung
    2|/system/media/audio/ringtones/Emotive_sensation.ogg|Emotive_sensation.ogg|62654|application/ogg|1293840202|1336569214|Emotive sensationQQQQQQQQQQQQQQQQQ|5205|1||1|0||1|0|0|0|Ringtone|0||1QQQQQQQQ|Samsung|1QQQQQQQQ293500348|Samsung
    3|/system/media/audio/ringtones/Steppin_Out.ogg|Steppin_Out.ogg|29989|application/ogg|1293840206|1336569214|Steppin' outQQQQQQQQQQQ|4000|2||2|0||1|0|0|0|Ringtone|0||2QQQQQQ|Dr. MAD|2QQQQQQQQQQ293500348|ringtones
    4|/system/media/audio/ringtones/Rich_Tone.ogg|Rich_Tone.ogg|38287|application/ogg|1293840206|1336569214|Rich toneQQQQQQQQQ|3110|1||1|0||1|0|0|0|Ringtone|0||1QQ|Samsung|1QQQQQQQQ293500348|Samsung
    5|/system/media/audio/ringtones/Basic_tone.ogg|Basic_tone.ogg|26729|application/ogg|1293840207|1336569214|Basic toneQQQQQQQQQQ|3448|1||1|0||1|0|0|0|Ringtone|0||1QQQQQQQQ|Samsung|1QQQQQQQQ293500348|Samsung
    6|/system/media/audio/ringtones/Andromeda.ogg|Andromeda.ogg|35809|application/ogg|1293840213|1336569214|AndromedaQQQQQQQQQQ|3000|3||3|0||1|0|0|0|Ringtone|0||3Q|pdx, ORT=|3QQQQQQQQ293500348|Unknown
    7|/system/media/audio/ringtones/Third_Eye.ogg|Third_Eye.ogg|24230|application/ogg|1293840213|1336569214|Third eyeQQQQQQQQQ|4000|2||2|0||1|0|0|0|Ringtone|0||2QQ|Dr. MAD|2QQQQQQQQQQ293500348|ringtones
    8|/system/media/audio/ringtones/Over_the_horizon_Default_ringtone.ogg|Over_the_horizon_Default_ringtone.ogg|325196|application/ogg|1293840213|1336569214|Over the horizonQQQQQQQQQQQQQQQ|19999|1||1|0||1|0|0|0|Ringtone|0||1QQQQQQQQ|Samsung|1Q293500348|Samsung
    9|/system/media/audio/ringtones/Cassiopeia.ogg|Cassiopeia.ogg|37066|application/ogg|1293840213|1336569214|CassiopeiaQQQQQQQQQQQ|4000|3||3|0||1|0|0|0|Ringtone|0||3QQQQQQQ|pdx, ORT=|3QQQQQQQQ293500348|Unknown
    10|/system/media/audio/ringtones/Classic_bell.ogg|Classic_bell.ogg|32894|application/ogg|1293840213|1336569214|Classic bellQQQQQQQQQQQQ|4362|1||1|0||1|0|0|0|Ringtone|0||1QQQQQQQQ|Samsung|1QQQQQQQQ293500348|Samsung
    11|/system/media/audio/ringtones/Basic_bell.ogg|Basic_bell.ogg|20504|application/ogg|1293840213|1336569214|Basic bellQQQQQQQQQQ|2325|1||1|0||1|0|0|0|Ringtone|0||1QQQQQQQQ|Samsung|1QQQQQQQQ293500348|Samsung
    12|/system/media/audio/ringtones/Beat_Plucker.ogg|Beat_Plucker.ogg|27500|application/ogg|1293840214|1336569214|Beat pluckerQQQQQQQQQQQQ|4000|2||2|0||1|0|0|0|Ringtone|0||2QQQQQQ|Dr. MAD|2QQQQQQQQQQ293500348|ringtones
    13|/system/media/audio/ringtones/No_Limits.ogg|No_Limits.ogg|25956|application/ogg|1293840214|1336569214|No limitsQQQQQQQQQ|3582|2||2|0||1|0|0|0|Ringtone|0||2Q|Dr. MAD|2QQQQQQQQQQ293500348|ringtones
    14|/system/media/audio/ringtones/Gimme_Mo_Town.ogg|Gimme_Mo_Town.ogg|60804|application/ogg|1293840214|1336569214|Gimme mo' townQQQQQQQQQQQQ|4000|2||2|0||1|0|0|0|Ringtone|0||2QQQQQQ|Dr. MAD|2QQQQQQQQQQ293500348|ringtones
    15|/system/media/audio/ringtones/Bollywood.ogg|Bollywood.ogg|39174|application/ogg|1293840215|1336569214|BollywoodQQQQQQQQQQ|4211|2||2|0||1|0|0|0|<unknown>|0||2QQQQQQ|Dr. MAD|2QQQQQQQQQQ293500348|ringtones
    16|/system/media/audio/ringtones/Curve_Ball_Blend.ogg|Curve_Ball_Blend.ogg|66482|application/ogg|1293840215|1336569214|Curve ball blendQQQQQQQQQQQQQQQ|4000|2||2|0||1|0|0|0|Ringtone|0||2QQQQQQ|Dr. MAD|2QQQQQQQQQQ293500348|ringtones
    17|/system/media/audio/ringtones/Funk_Yall.ogg|Funk_Yall.ogg|67077|application/ogg|1293840215|1336569214|Funk y'allQQQQQQQQQ|4103|2||2|0||1|0|0|0|Ringtone|0||Q|Dr. MAD|2QQQQQQQQQQ293500348|ringtones
    18|/system/media/audio/ringtones/Single_tone.ogg|Single_tone.ogg|24185|application/ogg|1293840215|1336569214|Single toneQQQQQQQQQQQ|3680|1||1|0||1|0|0|0|Ringtone|0||1QQQQQQQQ|Samsung|1QQQQQQQQ293500348|Samsung
    19|/system/media/audio/ringtones/Savannah.ogg|Savannah.ogg|64346|application/ogg|1293840215|1336569214|SavannahQQQQQQQQQ|4000|2||2|0||1|0|0|0|Ringtone|0||2QQQQ|Dr. MAD|2QQQQQQQQQQ293500348|ringtones
    20|/system/media/audio/ringtones/Pure_tone.ogg|Pure_tone.ogg|67204|application/ogg|1293840215|1336569214|Pure toneQQQQQQQQQ|8438|1||1|0||1|0|0|0|Ringtone|0||1Q|Samsung|1QQQQQQQQ293500348|Samsung
    21|/system/media/audio/ringtones/Chime.ogg|Chime.ogg|35378|application/ogg|1293840215|1336569214|ChimeQQQQQQ|3631|1||1|0||1|0|0|0|Ringtone|0||1QQQQQQQQ|Samsung|1QQQQQQQQ293500348|Samsung
    22|/system/media/audio/notifications/03_Bubbles.ogg|03_Bubbles.ogg|8046|application/ogg|1293840216|1336569214|BubblesQQQQQQQQ|534|1||4|0||0|0|0|1|<unknown>|0||1QQQQQQQQ|Samsung|4QQQQQQQQQQQQQQ-1608219629|notifications
    23|/system/media/audio/notifications/04_Chopsticks.ogg|04_Chopsticks.ogg|7281|application/ogg|1293840216|1336569214|ChopsticksQQQQQQQQQQQ|351|1||4|0||0|0|0|1|<unknown>|0||1QQQQQQQQ|Samsung|4QQQQQQQQQQQQQQ-1608219629|notifications
    24|/system/media/audio/notifications/05_Good_News.ogg|05_Good_News.ogg|18459|application/ogg|1293840216|1336569214|Good newsQQQQQQQQQ|1600|1||4|0||0|0|0|1|<unknown>|0||1QQQQQQQQ|Samsung|4QQQQQQQQQQQQQQ-1608219629|notifications
    25|/system/media/audio/notifications/12_Sherbet_Default_message.ogg|12_Sherbet_Default_message.ogg|17292|application/ogg|1293840216|1336569214|SherbetQQQQQQQQ|1373|1||4|0||0|0|0|1|<unknown>|0||1QQQQQQQQ|Samsung|4QQQQQQQQQQQQQQ-1608219629|notifications
    26|/system/media/audio/notifications/13_Whistle_Default_message.ogg|13_Whistle_Default_message.ogg|11638|application/ogg|1293840216|1336569214|WhistleQQQQQQQQ|1620|1||4|0||0|0|0|1|<unknown>|0||1QQQQQQQQ|Samsung|4QQQQQQQQQQQQQQ-1608219629|notifications
    27|/system/media/audio/notifications/02_Haze.ogg|02_Haze.ogg|13672|application/ogg|1293840216|1336569214|HazeQQQQQ|1028|1||4|0||0|0|0|1|<unknown>|0||1QQQQQQQQ|Samsung|4QQQQQQQQQQQQQQ-1608219629|notifications
    28|/system/media/audio/notifications/11_Pixiedust.ogg|11_Pixiedust.ogg|20105|application/ogg|1293840216|1336569214|Pixie dustQQQQQQQQQQ|1729|4||4|0||0|0|0|1|<unknown>|0||4||<unknown>|4QQQQQQQQQQQQQQ-1608219629|notifications
    29|/system/media/audio/notifications/10_Pizzicato.ogg|10_Pizzicato.ogg|24249|application/ogg|1293840216|1336569214|PizzicatoQQQQQQQQQQ|2003|4||4|0||0|0|0|1|<unknown>|0||4||<unknown>|4QQQQQQQQQQQQQQ-1608219629|notifications
    30|/system/media/audio/notifications/06_Knock.ogg|06_Knock.ogg|10984|application/ogg|1293840216|1336569214|KnockQQQQQQ|968|1||4|0||0|0|0|1|<unknown>|0||1QQQQQQ|Samsung|4QQQQQQQQQQQQQQ-1608219629|notifications
    31|/system/media/audio/notifications/08_Pure_Bell.ogg|08_Pure_Bell.ogg|12702|application/ogg|1293840216|1336569214|Pure bellQQQQQQQQQ|2021|1||4|0||0|0|0|1|<unknown>|0||1QQQQQQQQ|Samsung|4QQQQQQQQQQQQQQ-1608219629|notifications
    32|/system/media/audio/notifications/01_Cloud.ogg|01_Cloud.ogg|22531|application/ogg|1293840216|1336569214|CloudQQQQQQ|1910|1||4|0||0|0|0|1|<unknown>|0||1QQQQQ|Samsung|4QQQQQQQQQQQQQQ-1608219629|notifications
    33|/system/media/audio/notifications/09_Tweeters.ogg|09_Tweeters.ogg|7626|application/ogg|1293840216|1336569214|TweetersQQQQQQQQQ|788|4||4|0||0|0|0|1|<unknown>|0||4||<unknown>|4QQQQQQQQQQQQQQ-1608219629|notifications
    34|/system/media/audio/notifications/07_Postman_Default_Email.ogg|07_Postman_Default_Email.ogg|14081|application/ogg|1293840216|1336569214|PostmanQQQQQQQQ|2160|1||4|0||0|0|0|1|<unknown>|0||1QQQQQQQQ|Samsung|4QQQQQQQQQQQQQQ-1608219629|notifications
    35|/system/media/audio/notifications/14_On_time_Default_Calendar.ogg|14_On_time_Default_Calendar.ogg|15079|application/ogg|1293840216|1336569214|On timeQQQQQQQ|2494|1||4|0||0|0|0|1|<unknown>|0||1QQQQQQQQ|Samsung|4QQQQQQQQQQQQQQ-1608219629|notifications
    36|/system/media/audio/ui/Silent_mode_off.ogg|Silent_mode_off.ogg|6750|application/ogg|1293840216|1336569214|Silent mode offQQQQQQQQQQQQQQ|287|1||5|0||0|1|0|0|<unknown>|0||1QQQQQQQQ|Samsung|5QQQ78638153|ui
    37|/system/media/audio/ui/camera_click.ogg|camera_click.ogg|4851|application/ogg|1293840217|1336569214|Camera ShutterQQQQQQQQQQQQQQ|140|4||5|0||0|1|0|0|<unknown>|0||4||<unknown>|5QQQ78638153|ui
    38|/system/media/audio/ui/VideoRecord.ogg|VideoRecord.ogg|5582|application/ogg|1293840217|1336569214|Video RecordQQQQQQQQQQQQ|381|2||5|0||0|1|0|0|<unknown>|0||2QQQQQQ|Dr. MAD|5QQQ78638153|ui
    39|/system/media/audio/ui/OTP.ogg|OTP.ogg|4161|application/ogg|1293840217|1336569214|OTPQQQQ|250|4||5|0||0|1|0|0|<unknown>|0||4||<unknown>|5QQQ78638153|ui
    40|/system/media/audio/ui/Volume_control.ogg|Volume_control.ogg|5515|application/ogg|1293840217|1336569214|Volume controlQQQQQQQQQQQQQQ|141|4||5|0||0|1|0|0|<unknown>|0||4||<unknown>|5QQQ78638153|ui
    41|/system/media/audio/ui/Lock.ogg|Lock.ogg|8476|application/ogg|1293840217|1336569214|LockQQQQQ|314|4||5|0||0|1|0|0|<unknown>|0||4||<unknown>|5QQQ78638153|ui
    42|/system/media/audio/ui/KeypressDelete.ogg|KeypressDelete.ogg|6193|application/ogg|1293840217|1336569214|KeypressDeleteQQQQQQQQQQQQQQQ|169|5||5|0||0|1|0|0|<unknown>|0||5QQQQQQQQQQQQQ*QQQQQQQQQQQQQQQQQQQQQQQQQ|Copyright 2009 Android Open Source Project|5QQQ78638153|ui
    43|/system/media/audio/ui/TW_Touch.ogg|TW_Touch.ogg|4894|application/ogg|1293840217|1336569214|TouchQQQQQQ|350|4||5|0||0|1|0|0|<unknown>|0||4||<unknown>|5QQQ78638153|ui
    44|/system/media/audio/ui/MuteTone.ogg|MuteTone.ogg|3740|application/ogg|1293840217|1336569214|MuteToneQQQQQQQQQ|331|4||5|0||0|1|0|0|Blues|0||4||<unknown>|5QQQ78638153|ui
    45|/system/media/audio/ui/Charger_Connection.ogg|Charger_Connection.ogg|10478|application/ogg|1293840217|1336569214|Charger_ConnectionQQQQQQQQ
                                                                QQQQQQQQQQQ|528|4||5|0||0|1|0|0|<unknown>|0||4||<unknown>|5QQQ78638153|ui
    46|/system/media/audio/ui/Insert.ogg|Insert.ogg|7274|application/ogg|1293840217|1336569214|InsertQQQQQQQ|331|4||5|0||0|1|0|0|Blues|0||4||<unknown>|5QQQ78638153|ui
    47|/system/media/audio/ui/Unlock.ogg|Unlock.ogg|8436|application/ogg|1293840217|1336569214|UnlockQQQQQQQ|441|1||5|0||0|1|0|0|<unknown>|0||1QQQQQQQQ|Samsung|5QQ78638153|ui
    48|/system/media/audio/ui/KeypressSpacebar.ogg|KeypressSpacebar.ogg|7392|application/ogg|1293840217|1336569214|KeypressSpacebarQQQQQQQQQQQQQQQQQ|279|5||5|0||0|1|0|0|<unknown>|0||5QQQQQQQQQQQQQ*QQQQQQQQQQQQQQQQQQQQQQQQQ|Copyright 2009 Android Open Source Project|5QQQ78638153|ui
    49|/system/media/audio/ui/Effect_Tick.ogg|Effect_Tick.ogg|3994|application/ogg|1293840217|1336569214|Effect_TickQQQQQQQ
                                        QQQQQ|32|4||5|0||0|1|0|0|<unknown>|0||4||<unknown>|5QQQ78638153|ui
    50|/system/media/audio/ui/KeypressReturn.ogg|KeypressReturn.ogg|7972|application/ogg|1293840218|1336569214|KeypressReturnQQQQQQQQQQQQQQQ|364|5||5|0||0|1|0|0|<unknown>|0||5QQQQQQQQQQQQQ*QQQQQQQQQQQQQQQQQQQQQQQQQ|Copyright 2009 Android Open Source Project|5QQQ78638153|ui
    51|/system/media/audio/ui/TW_Waterdrop.ogg|TW_Waterdrop.ogg|9627|application/ogg|1293840218|1336569214|TW_WaterdropQQQ
                                        QQQQQQQQQQ|826|4||5|0||0|1|0|0|<unknown>|0||4||<unknown>|5QQQ78638153|ui
    52|/system/media/audio/ui/KeypressStandard.ogg|KeypressStandard.ogg|5194|application/ogg|1293840218|1336569214|KeypressStandardQQQQQQQQQQQQQQQQQ|101|5||5|0||0|1|0|0|<unknown>|0||5QQQQQQQQQQQQQ*QQQQQQQQQQQQQQQQQQQQQQQQQ|Copyright 2009 Android Open Source Project|5QQQ78638153|ui
    53|/system/media/audio/ui/TW_SIP.ogg|TW_SIP.ogg|4298|application/ogg|1293840218|1336569214|SIPQQQQ|149|4||5|0||0|1|0|0|<unknown>|0||4||<unknown>|5QQQ78638153|ui
    54|/system/media/audio/ui/camera_click_short.ogg|camera_click_short.ogg|5432|application/ogg|1293840218|1336569214|camera_click_shortQQQQQQQ
                                                            QQQQQQ
                                                                    QQQQQQ|289|4||5|0||0|1|0|0|<unknown>|0||4||<unknown>|5QQQ78638153|ui
    55|/system/media/audio/alarms/Alarm_Beep_02.ogg|Alarm_Beep_02.ogg|5898|application/ogg|1293840218|1336569214|BeeBeep AlarmQQQQQQQQQQQQQ|251|4||6|0||0|0|1|0|<unknown>|0||4||<unknown>|6QQQQQQQ-183121033|alarms
    56|/system/media/audio/alarms/Alarm_Beep_03.ogg|Alarm_Beep_03.ogg|21153|application/ogg|1293840218|1336569214|Beep-Beep-Beep AlarmQQQQQ

