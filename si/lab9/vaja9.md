# 9. Vaja: Forenzika mobilnih naprav

## Navodila

1. Priklopite mobilno napravo ter najdite podatke, ki opisujejo uporabnikovo aktivnost.

## Dodatne informacije

## Podrobna navodila

### 1. Forenzika mobilnih naprav

V večini primerov mobilne naprave poganjajo prilagojene različice operacijskih sistemov, ki so lahko osnovani na operacijskem sistemu Linux ([Android](https://en.wikipedia.org/wiki/Android_(operating_system)), [Ubuntu Touch](https://en.wikipedia.org/wiki/Ubuntu_Touch), [postmarketOS](https://en.wikipedia.org/wiki/PostmarketOS), [SailfishOS](https://en.wikipedia.org/wiki/Sailfish_OS), [Mobian](https://mobian-project.org/), [PureOS](https://en.wikipedia.org/wiki/PureOS), [Plasma Mobile](https://en.wikipedia.org/wiki/Plasma_Mobile)), vendar imajo vsi razen Android-a težave zaradi neobstoječe podpore s strani proizvajalcev.

Drugi najbolj pogosti mobilni operacijski sistem je [iOS](https://en.wikipedia.org/wiki/IOS) podjetja Apple, ki je osnovan na operacijskem sistemu [macOS](https://en.wikipedia.org/wiki/MacOS). Forenziko mobilne naprave iOS si žal ne bomo pogledali na vajah, vendar uporablja enake postopke, kot pri drugih operacijskih sistemih mobilnih naprav, seveda z nekaj specifikami.

Najprej si bomo pogledali mobilni telefon [N900](https://en.wikipedia.org/wiki/Nokia_N900), ki uporablja operacijski sistem [Maemo 5](https://en.wikipedia.org/wiki/Maemo), ki deluje kot splošni Linux operacijski sistem in podpira ukaze in orodja, ki smo si jih pogledali na vajah do sedaj. Večina uporabniških podatkov je shranjenih v obliki [Berkeley DB](https://en.wikipedia.org/wiki/Berkeley_DB), do njih lahko dostopamo z orodjem [db-util](https://www.unix.com/man-page/linux/1/dbutil/). Sliko pomnilnika telefona N900 dobite [tukaj](https://polz.si/dsrf/georgepicobello.tgz).

    wget https://polz.si/dsrf/georgepicobello.tgz

    tar -zxvf georgepicobello.tgz

    apt update
    apt install db-util

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

    db_dump - p ./user/.feedservice2/facebook/facebook.db

Mobilni operacijski sistem Android imajo določene podatke zakrite in prav tako uporablja šifriranje za celotni pomnilnik. Če želimo dostopati do vseh podatkov, moramo postati super uporabnik oz. `root`. Določena podjetja nam to omogočajo, vendar ne privzeto, za druge naprave pa moramo poznati kakšno varnostno pomanjkljivost, ki jo izkoristimo, da postanemo super uporabnik. Za delo z Android napravami se uporablja [ADB - Android Debug Bridge](https://developer.android.com/tools/adb). Orodje je sestavljeno iz dveh delov - prvi je storitev, ki teče na telefonu (`adbd`), drugi pa orodje, ki se prek USB pogovarja s storitvijo na telefonu (`adb`). Preko orodje `adb` lahko na telefonu odpremo lupino `adb shell`. Če `adbd` teče kot super uporabnik, lahko na telefonu dobite polne pravice, s pomočjo katerih lahko preberete vse podatke. En od načinov, kako to doseči, je opisan [tule](https://wiki.mozilla.org/Mobile/Fennec/Android/Rooting/adb). Ko na Android telefonu postanete super uporabnik, lahko prenesete vse datoteke z telefona z ukazom `adb pull`. Datoteke, zajete na Android telefonu, prav tako dobite [tukaj](polz.si/media/uploads/dsrf/android/samsung_ace.tgz).

    wget https://polz.si/dsrf/android/samsung_ace.tgz

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

Razdelek `stl9` hrani operacijski sistem, kjer mapa `bin` vsebuje programe, mapa `lib` sistemske knjižnice, mapa `usr` uporabniške dele nameščenih aplikacij, mapa `xbin` vsebuje orodja za pridobitev super uporabnika in mapa `etc` vsebuje nastavitev naprave.

    ls stl9/

    app	    cameradata	  CSCVersion.txt  framework  SW_Configuration.xml  xbin
    bin	    csc		  etc		  lib	     T9DB
    build.prop  CSCFiles.txt  fonts		  media      usr

Razdelek `stl10` vsebuje del začasnih datotek in dnevnikov. 

    ls stl10/

    downloadfile.bin  lost+found  recovery

V datoteki `stl10/recovery/last_log`, ki predstavlja dnevnik osnovne nastavitve naprave, vidimo da naprava uporablja datotečni sistem [Robust FAT File System - RFS](https://www.manualslib.com/manual/147588/Samsung-V1-3-0.html?page=9#manual).

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
    persist.service.adb.els stl9/

    app	    cameradata	  CSCVersion.txt  framework  SW_Configuration.xml  xbin
    bin	    csc		  etc		  lib	     T9DB
    build.prop  CSCFiles.txt  fonts		  media      usr
nable=0
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
    persist.rild.nitz_shols stl9/

    app	    cameradata	  CSCVersion.txt  framework  SW_Configuration.xml  xbin
    bin	    csc		  etc		  lib	     T9DB
    build.prop  CSCFiles.txt  fonts		  media      usr
rt_ons_2=
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

Razdelek `stl11` vsebuje uporabniške in sistemske podatke.

    ls stl11/

    aeqcoe.txt   brcm	   dontpanic   log	      secure
    anr	     cache	   fota        lost+found     soundbooster.txt
    app	     cp_data.txt   gps	       misc	      system
    app-private  dalvik-cache  lcs.socket  property       tombstones
    backup	     data	   local       rilgps.socket  vt

