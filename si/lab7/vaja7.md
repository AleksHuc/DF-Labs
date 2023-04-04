# 6. Vaja: Beleženje pod operacijskim sistemom Linux

## Navodila

1. Poišči datoteke, ki beležijo dogodke pod operacijskim sistemom Linux.
2. Pošiljaj beležke v realnem času preko mreže med dvema navideznima računalnikoma.

## Dodatne informacije

## Podrobna navodila

### 1. Beleženje pod operacijskim sistemom Linux

Beleženje dogodkov lokalno in preko mreže poteka preko protokola `Syslog`. Protokol `Syslog` in format omrežnih sporočil sta določena v [`RFC5424`](https://www.rfc-editor.org/rfc/rfc5424).

Sporočilo za posamezen dogodek vsebuje:
1. prioriteto,
2. lokacijo,
3. povzročitelja,
4. čas,
5. opis.

Vrednost `PRI` nam opisuje nivo oz. resnost dogodka z vrednostmi med 0 in 23, kjer 0 opisuje najvišji nivo oz. najvišjo resnost. Več programov implementira `RFC5424`, saj ne definira kako hranimo podatke lokalno:
- [`rsyslogd`](https://linux.die.net/man/8/rsyslogd)
- [`syslog-ng`](https://linux.die.net/man/8/syslog-ng)
- [`syslogd`](https://linux.die.net/man/8/syslogd)
- ...

Ker programi implementirajo enak standard, si lahko med seboj izmenjujejo sporočila v formatu `syslog` (rsyslogd --(syslog_message)--> syslog-ng).

Program [init](https://en.wikipedia.org/wiki/Init) je prvi program, ki ga požene operacijski sistem Linux in skrbi za uporabniške procese in se zaključi, ko operacijski sistem zaustavimo. Eden izmed najbolj popularnih init programov je [`systemd`](https://en.wikipedia.org/wiki/Systemd), ki ima tudi svoj način beleženja dogodkov z [`journalctl`](https://www.man7.org/linux/man-pages/man1/journalctl.1.html). Beleženje z `journalctl` v primerjavi z `syslog` zavzame manj prostora saj shranjuje v binarni obliki in ne tekstovni. Dogodke začne beležiti bolj zgodaj med postopkom zagonom operacijskega sistema in tako lahko ujame več dogodkov. Prav tako, lahko pretvori svoje beležke v pravilni format in jih nato pošlje preko omrežja po protokolu `syslog`.

Nekateri programi ne uporabljajo `syslog` beležk, vendar imajo svoj format in lokacijo beležk, na primer [`apache2`](https://manpages.ubuntu.com/manpages/bionic/man8/apache2.8.html).

Lokacije beležk:
- `/var/log/` - sistemske beležke, na primer `/var/log/syslog`.
- `/var/lib/` - začasne datoteke, ki se ohranijo med ponovnim zagonom, na primer `/var/lib/dpkg`.
- `/var/spool/` - začasne datoteke, ki so trenutno v uporabi, na primer `/var/spool/cups`.
- `.bash_history` - zgodovina ukazov v terminalu.
- `.cache` - začasne datoteke.

Sedaj kloniramo naš Linux navidezni računalnik, nastavimo omrežje na obeh na `Bridged Adapter` ali pa na `NAT network`, da sta v skupnem omrežju ter ju poženemo. Originalni navidezni računalnik nam bo služil kot strežnik, medtem ko bo klon klient.

Z ukazom [`apt`](https://manpages.ubuntu.com/manpages/xenial/man8/apt.8.html) preverimo ali imamo nameščeno kakšno implementacijo `syslog` protokola, če ne ga namestimo z upravljalcem paketom našega operacijskega sistema. Sedaj izpišemo sistemske beležke z ukazi [`cat`](https://man7.org/linux/man-pages/man1/cat.1.html), [`less`](https://man7.org/linux/man-pages/man1/less.1.html), [`tail`](https://man7.org/linux/man-pages/man1/tail.1.html) ter podobnimi.

    apt list --installed | grep syslog

    WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

    rsyslog/stable,stable-security,now 8.2102.0-2+deb11u1 amd64 [installed]

    cat /var/log/syslog

    less /var/log/syslog

    tail -f /var/log/syslog

Sistemske beležke `systemd` izpišemo z ukazom `journalctl`.

    journalctl

Sedaj namestimo program `apache2` ter preverimo njegovo delovanje z ukazom [`service`](https://www.commandlinux.com/man-page/man8/service.8.html).

    apt update
    apt install apache2

    service apache2 status

    ● apache2.service - The Apache HTTP Server
        Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor prese>
        Active: active (running) since Tue 2023-04-04 14:13:52 CEST; 17s ago
        Docs: https://httpd.apache.org/docs/2.4/
    Main PID: 3881 (apache2)
        Tasks: 55 (limit: 4657)
        Memory: 21.3M
            CPU: 21ms
        CGroup: /system.slice/apache2.service
                ├─3881 /usr/sbin/apache2 -k start
                ├─3883 /usr/sbin/apache2 -k start
                └─3884 /usr/sbin/apache2 -k start

    Apr 04 14:13:52 debian systemd[1]: Starting The Apache HTTP Server...
    Apr 04 14:13:52 debian apachectl[3880]: AH00558: apache2: Could not reliably de>
    Apr 04 14:13:52 debian systemd[1]: Started The Apache HTTP Server.

Z ukazom [`dpkg-query`](https://man7.org/linux/man-pages/man1/dpkg-query.1.html) poiščemo vse datoteke, ki so se namestili ob namestitvi paketa `apache2`.

    dpkg-query -L apache2

    /.
    /etc
    /etc/apache2
    /etc/apache2/apache2.conf
    /etc/apache2/conf-available
    /etc/apache2/conf-available/charset.conf
    /etc/apache2/conf-available/localized-error-pages.conf
    /etc/apache2/conf-available/other-vhosts-access-log.conf
    /etc/apache2/conf-available/security.conf
    /etc/apache2/conf-available/serve-cgi-bin.conf
    /etc/apache2/conf-enabled
    /etc/apache2/envvars
    /etc/apache2/magic
    /etc/apache2/mods-available
    /etc/apache2/mods-available/access_compat.load
    /etc/apache2/mods-available/actions.conf
    /etc/apache2/mods-available/actions.load
    /etc/apache2/mods-available/alias.conf
    /etc/apache2/mods-available/alias.load
    /etc/apache2/mods-available/allowmethods.load
    /etc/apache2/mods-available/asis.load
    /etc/apache2/mods-available/auth_basic.load
    /etc/apache2/mods-available/auth_digest.load
    /etc/apache2/mods-available/auth_form.load
    /etc/apache2/mods-available/authn_anon.load
    /etc/apache2/mods-available/authn_core.load
    /etc/apache2/mods-available/authn_dbd.load
    /etc/apache2/mods-available/authn_dbm.load
    /etc/apache2/mods-available/authn_file.load
    /etc/apache2/mods-available/authn_socache.load
    /etc/apache2/mods-available/authnz_fcgi.load
    /etc/apache2/mods-available/authnz_ldap.load
    /etc/apache2/mods-available/authz_core.load
    /etc/apache2/mods-available/authz_dbd.load
    /etc/apache2/mods-available/authz_dbm.load
    /etc/apache2/mods-available/authz_groupfile.load
    /etc/apache2/mods-available/authz_host.load
    /etc/apache2/mods-available/authz_owner.load
    /etc/apache2/mods-available/authz_user.load
    /etc/apache2/mods-available/autoindex.conf
    /etc/apache2/mods-available/autoindex.load
    /etc/apache2/mods-available/brotli.load
    /etc/apache2/mods-available/buffer.load
    /etc/apache2/mods-available/cache.load
    /etc/apache2/mods-available/cache_disk.conf
    /etc/apache2/mods-available/cache_disk.load
    /etc/apache2/mods-available/cache_socache.load
    /etc/apache2/mods-available/cern_meta.load
    /etc/apache2/mods-available/cgi.load
    /etc/apache2/mods-available/cgid.conf
    /etc/apache2/mods-available/cgid.load
    /etc/apache2/mods-available/charset_lite.load
    /etc/apache2/mods-available/data.load
    /etc/apache2/mods-available/dav.load
    /etc/apache2/mods-available/dav_fs.conf
    /etc/apache2/mods-available/dav_fs.load
    /etc/apache2/mods-available/dav_lock.load
    /etc/apache2/mods-available/dbd.load
    /etc/apache2/mods-available/deflate.conf
    /etc/apache2/mods-available/deflate.load
    /etc/apache2/mods-available/dialup.load
    /etc/apache2/mods-available/dir.conf
    /etc/apache2/mods-available/dir.load
    /etc/apache2/mods-available/dump_io.load
    /etc/apache2/mods-available/echo.load
    /etc/apache2/mods-available/env.load
    /etc/apache2/mods-available/expires.load
    /etc/apache2/mods-available/ext_filter.load
    /etc/apache2/mods-available/file_cache.load
    /etc/apache2/mods-available/filter.load
    /etc/apache2/mods-available/headers.load
    /etc/apache2/mods-available/heartbeat.load
    /etc/apache2/mods-available/heartmonitor.load
    /etc/apache2/mods-available/http2.conf
    /etc/apache2/mods-available/http2.load
    /etc/apache2/mods-available/ident.load
    /etc/apache2/mods-available/imagemap.load
    /etc/apache2/mods-available/include.load
    /etc/apache2/mods-available/info.conf
    /etc/apache2/mods-available/info.load
    /etc/apache2/mods-available/lbmethod_bybusyness.load
    /etc/apache2/mods-available/lbmethod_byrequests.load
    /etc/apache2/mods-available/lbmethod_bytraffic.load
    /etc/apache2/mods-available/lbmethod_heartbeat.load
    /etc/apache2/mods-available/ldap.conf
    /etc/apache2/mods-available/ldap.load
    /etc/apache2/mods-available/log_debug.load
    /etc/apache2/mods-available/log_forensic.load
    /etc/apache2/mods-available/lua.load
    /etc/apache2/mods-available/macro.load
    /etc/apache2/mods-available/md.load
    /etc/apache2/mods-available/mime.conf
    /etc/apache2/mods-available/mime.load
    /etc/apache2/mods-available/mime_magic.conf
    /etc/apache2/mods-available/mime_magic.load
    /etc/apache2/mods-available/mpm_event.conf
    /etc/apache2/mods-available/mpm_event.load
    /etc/apache2/mods-available/mpm_prefork.conf
    /etc/apache2/mods-available/mpm_prefork.load
    /etc/apache2/mods-available/mpm_worker.conf
    /etc/apache2/mods-available/mpm_worker.load
    /etc/apache2/mods-available/negotiation.conf
    /etc/apache2/mods-available/negotiation.load
    /etc/apache2/mods-available/proxy.conf
    /etc/apache2/mods-available/proxy.load
    /etc/apache2/mods-available/proxy_ajp.load
    /etc/apache2/mods-available/proxy_balancer.conf
    /etc/apache2/mods-available/proxy_balancer.load
    /etc/apache2/mods-available/proxy_connect.load
    /etc/apache2/mods-available/proxy_express.load
    /etc/apache2/mods-available/proxy_fcgi.load
    /etc/apache2/mods-available/proxy_fdpass.load
    /etc/apache2/mods-available/proxy_ftp.conf
    /etc/apache2/mods-available/proxy_ftp.load
    /etc/apache2/mods-available/proxy_hcheck.load
    /etc/apache2/mods-available/proxy_html.conf
    /etc/apache2/mods-available/proxy_html.load
    /etc/apache2/mods-available/proxy_http.load
    /etc/apache2/mods-available/proxy_http2.load
    /etc/apache2/mods-available/proxy_scgi.load
    /etc/apache2/mods-available/proxy_uwsgi.load
    /etc/apache2/mods-available/proxy_wstunnel.load
    /etc/apache2/mods-available/ratelimit.load
    /etc/apache2/mods-available/reflector.load
    /etc/apache2/mods-available/remoteip.load
    /etc/apache2/mods-available/reqtimeout.conf
    /etc/apache2/mods-available/reqtimeout.load
    /etc/apache2/mods-available/request.load
    /etc/apache2/mods-available/rewrite.load
    /etc/apache2/mods-available/sed.load
    /etc/apache2/mods-available/session.load
    /etc/apache2/mods-available/session_cookie.load
    /etc/apache2/mods-available/session_crypto.load
    /etc/apache2/mods-available/session_dbd.load
    /etc/apache2/mods-available/setenvif.conf
    /etc/apache2/mods-available/setenvif.load
    /etc/apache2/mods-available/slotmem_plain.load
    /etc/apache2/mods-available/slotmem_shm.load
    /etc/apache2/mods-available/socache_dbm.load
    /etc/apache2/mods-available/socache_memcache.load
    /etc/apache2/mods-available/socache_redis.load
    /etc/apache2/mods-available/socache_shmcb.load
    /etc/apache2/mods-available/speling.load
    /etc/apache2/mods-available/ssl.conf
    /etc/apache2/mods-available/ssl.load
    /etc/apache2/mods-available/status.conf
    /etc/apache2/mods-available/status.load
    /etc/apache2/mods-available/substitute.load
    /etc/apache2/mods-available/suexec.load
    /etc/apache2/mods-available/unique_id.load
    /etc/apache2/mods-available/userdir.conf
    /etc/apache2/mods-available/userdir.load
    /etc/apache2/mods-available/usertrack.load
    /etc/apache2/mods-available/vhost_alias.load
    /etc/apache2/mods-available/xml2enc.load
    /etc/apache2/mods-enabled
    /etc/apache2/ports.conf
    /etc/apache2/sites-available
    /etc/apache2/sites-available/000-default.conf
    /etc/apache2/sites-available/default-ssl.conf
    /etc/apache2/sites-enabled
    /etc/cron.daily
    /etc/cron.daily/apache2
    /etc/default
    /etc/default/apache-htcacheclean
    /etc/init.d
    /etc/init.d/apache-htcacheclean
    /etc/init.d/apache2
    /etc/logrotate.d
    /etc/logrotate.d/apache2
    /lib
    /lib/systemd
    /lib/systemd/system
    /lib/systemd/system/apache-htcacheclean.service
    /lib/systemd/system/apache-htcacheclean@.service
    /lib/systemd/system/apache2.service
    /lib/systemd/system/apache2@.service
    /usr
    /usr/lib
    /usr/lib/cgi-bin
    /usr/sbin
    /usr/sbin/a2enmod
    /usr/sbin/a2query
    /usr/sbin/apache2ctl
    /usr/share
    /usr/share/apache2
    /usr/share/apache2/apache2-maintscript-helper
    /usr/share/apache2/ask-for-passphrase
    /usr/share/bash-completion
    /usr/share/bash-completion/completions
    /usr/share/bash-completion/completions/a2enmod
    /usr/share/bug
    /usr/share/bug/apache2
    /usr/share/doc
    /usr/share/doc/apache2
    /usr/share/doc/apache2/NEWS.Debian.gz
    /usr/share/doc/apache2/PACKAGING.gz
    /usr/share/doc/apache2/README.Debian.gz
    /usr/share/doc/apache2/README.backtrace
    /usr/share/doc/apache2/README.multiple-instances
    /usr/share/doc/apache2/changelog.Debian.gz
    /usr/share/doc/apache2/changelog.gz
    /usr/share/doc/apache2/copyright
    /usr/share/doc/apache2/examples
    /usr/share/doc/apache2/examples/apache2.monit
    /usr/share/doc/apache2/examples/secondary-init-script
    /usr/share/doc/apache2/examples/setup-instance
    /usr/share/lintian
    /usr/share/lintian/overrides
    /usr/share/lintian/overrides/apache2
    /usr/share/man
    /usr/share/man/man1
    /usr/share/man/man1/a2query.1.gz
    /usr/share/man/man8
    /usr/share/man/man8/a2enconf.8.gz
    /usr/share/man/man8/a2enmod.8.gz
    /usr/share/man/man8/a2ensite.8.gz
    /usr/share/man/man8/apache2ctl.8.gz
    /var
    /var/cache
    /var/cache/apache2
    /var/cache/apache2/mod_cache_disk
    /var/lib
    /var/lib/apache2
    /var/log
    /var/log/apache2
    /var/www
    /var/www/html
    /usr/sbin/a2disconf
    /usr/sbin/a2dismod
    /usr/sbin/a2dissite
    /usr/sbin/a2enconf
    /usr/sbin/a2ensite
    /usr/sbin/apachectl
    /usr/share/bash-completion/completions/a2disconf
    /usr/share/bash-completion/completions/a2dismod
    /usr/share/bash-completion/completions/a2dissite
    /usr/share/bash-completion/completions/a2enconf
    /usr/share/bash-completion/completions/a2ensite
    /usr/share/bug/apache2/control
    /usr/share/bug/apache2/script
    /usr/share/man/man8/a2disconf.8.gz
    /usr/share/man/man8/a2dismod.8.gz
    /usr/share/man/man8/a2dissite.8.gz
    /usr/share/man/man8/apachectl.8.gz

Ugotovimo, da program `apache2` beleži dogodke v mapi `/var/log/apache2` ter hrani trenutno odprte datoteke v mapi `/var/lib/apache2`. Izpišemo, na primer dnevnik dostopov do spletne strani `/var/log/apache2/access.log`, nato odpremo spletno stran z ukazom [`curl`](https://linux.die.net/man/1/curl) in brskalnikom ter ponovno pogledamo v dnevnik dostopov.

    ls /var/log/apache2

    access.log  error.log  other_vhosts_access.log

    ls /var/lib/apache2

    conf  module  site

    cat /var/log/apache2/access.log

    apt update
    apt install curl

    cat /var/log/apache2/access.log 

    curl http://localhost

    cat /var/log/apache2/access.log

    ::1 - - [04/Apr/2023:14:31:39 +0200] "GET / HTTP/1.1" 200 10956 "-" "curl/7.74.0"
    127.0.0.1 - - [04/Apr/2023:14:32:48 +0200] "GET / HTTP/1.1" 200 3380 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0"
    127.0.0.1 - - [04/Apr/2023:14:32:48 +0200] "GET /icons/openlogo-75.png HTTP/1.1" 200 6040 "http://localhost/" "Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0"
    127.0.0.1 - - [04/Apr/2023:14:32:48 +0200] "GET /favicon.ico HTTP/1.1" 404 487 "http://localhost/" "Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0"

Sedaj izpišemo še ostale beležke v mapi `/var/log`, `/var/lib` in `.cache` ter datoteko `.bash_history`.

    ls /var/log

    alternatives.log    btmp.1	    gdm3	       syslog
    alternatives.log.1  cups	    installer	       syslog.1
    apache2		    daemon.log	    journal	       unattended-upgrades
    apt		    daemon.log.1    kern.log	       user.log
    auth.log	    debug	    kern.log.1	       user.log.1
    auth.log.1	    debug.1	    lastlog	       vboxadd-install.log
    boot.log	    dpkg.log	    messages	       vboxadd-setup.log
    boot.log.1	    dpkg.log.1	    messages.1	       wtmp
    boot.log.2	    faillog	    private
    btmp		    fontconfig.log  speech-dispatcher

    ls /var/lib

    AccountsService      emacsen-common  PackageKit  ucf
    alsa		     fwupd	     pam	 udisks2
    apache2		     gdm3	     plymouth	 unattended-upgrades
    app-info	     geoclue	     polkit-1	 upower
    apt		     ghostscript     private	 usb_modeswitch
    aspell		     grub	     python	 usbutils
    boltd		     ispell	     realmd	 VBoxGuestAdditions
    colord		     libreoffice     sgml-base	 vim
    dbus		     logrotate	     snmp	 xfonts
    dhcp		     man-db	     sudo	 xkb
    dictionaries-common  misc	     synaptic	 xml-core
    dkms		     NetworkManager  systemd
    dpkg		     os-prober	     tpm

    ls .cache

    appstream

    cat .bash_history

 ### 2. Pošiljanje belež preko mreže

Potrebujemo dva navidezna računalnika, prvi bo sprejemal in izpisoval dogodke, drugi mu jih bo pošiljal preko mreže. Na prvem navideznem računalniku preverimo njegov omrežni IP naslov z ukazom [`ip`](https://linux.die.net/man/8/ip), nato odpremo nastavitveno datoteko `rsyslog` programa `/etc/rsyslog.conf` in v njej odkomentiramo vrstici, ki omogočita sprejemanje dogodkov preko `UDP` omrežnega protokola. Sedaj še ponovno poženemo program `rsyslog` z ukazom `service` ter poženemo izpisovanje sistemskih dogodkov v realnem času.

    ip a

    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host 
        valid_lft forever preferred_lft forever
    2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 08:00:27:ab:74:50 brd ff:ff:ff:ff:ff:ff
        inet 192.168.34.109/24 brd 192.168.34.255 scope global dynamic noprefixroute enp0s3
        valid_lft 283294sec preferred_lft 283294sec
        inet6 2001:1470:fffd:101c:a40a:34d4:35c:9a83/64 scope global temporary dynamic 
        valid_lft 600100sec preferred_lft 81102sec
        inet6 2001:1470:fffd:101c:a00:27ff:feab:7450/64 scope global dynamic mngtmpaddr noprefixroute 
        valid_lft 2591982sec preferred_lft 604782sec
        inet6 fe80::a00:27ff:feab:7450/64 scope link noprefixroute 
        valid_lft forever preferred_lft forever

    nano /etc/rsyslog.conf

    # /etc/rsyslog.conf configuration file for rsyslog
    #
    # For more information install rsyslog-doc and see
    # /usr/share/doc/rsyslog-doc/html/configuration/index.html


    #################
    #### MODULES ####
    #################

    module(load="imuxsock") # provides support for local system logging
    module(load="imklog")   # provides kernel logging support
    #module(load="immark")  # provides --MARK-- message capability

    # provides UDP syslog reception
    module(load="imudp")
    input(type="imudp" port="514")

    # provides TCP syslog reception
    #module(load="imtcp")
    #input(type="imtcp" port="514")


    ###########################
    #### GLOBAL DIRECTIVES ####
    ###########################

    #
    # Use traditional timestamp format.
    # To enable high precision timestamps, comment out the following line.
    #
    $ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

    #
    # Set the default permissions for all log files.
    #
    $FileOwner root
    $FileGroup adm
    $FileCreateMode 0640
    $DirCreateMode 0755
    $Umask 0022

    #
    # Where to place spool and state files
    #
    $WorkDirectory /var/spool/rsyslog

    #
    # Include all config files in /etc/rsyslog.d/
    #
    $IncludeConfig /etc/rsyslog.d/*.conf


    ###############
    #### RULES ####
    ###############

    #
    # First some standard log files.  Log by facility.
    #
    auth,authpriv.*			/var/log/auth.log
    *.*;auth,authpriv.none		-/var/log/syslog
    #cron.*				/var/log/cron.log
    daemon.*			-/var/log/daemon.log
    kern.*				-/var/log/kern.log
    lpr.*				-/var/log/lpr.log
    mail.*				-/var/log/mail.log
    user.*				-/var/log/user.log

    #
    # Logging for the mail system.  Split it up so that
    # it is easy to write scripts to parse these files.
    #
    mail.info			-/var/log/mail.info
    mail.warn			-/var/log/mail.warn
    mail.err			/var/log/mail.err

    #
    # Some "catch-all" log files.
    #
    *.=debug;\
        auth,authpriv.none;\
        mail.none		-/var/log/debug
    *.=info;*.=notice;*.=warn;\
        auth,authpriv.none;\
        cron,daemon.none;\
        mail.none		-/var/log/messages

    #
    # Emergencies are sent to everybody logged in.
    #
    *.emerg				:omusrmsg:*

    service rsyslog restart

    tail -f /var/log/syslog

Na drugem navideznem računalniku dodamo v nastavitveno datoteko `/etc/rsyslog.conf` nastavitev, ki preusmeri vse sistemske dogodke preko mreže našemu prvemu navideznemu račnunalniki. Sedaj še ponovno poženemo program `rsyslog` z ukazom `service` ter ustvarimo poljubne tekstovne dogodke z ukazom [`logger`](https://linux.die.net/man/1/logger).

    nano /etc/rsyslog.conf

    # /etc/rsyslog.conf configuration file for rsyslog
    #
    # For more information install rsyslog-doc and see
    # /usr/share/doc/rsyslog-doc/html/configuration/index.html


    #################
    #### MODULES ####
    #################

    module(load="imuxsock") # provides support for local system logging
    module(load="imklog")   # provides kernel logging support
    #module(load="immark")  # provides --MARK-- message capability

    # provides UDP syslog reception
    #module(load="imudp")
    #input(type="imudp" port="514")

    # provides TCP syslog reception
    #module(load="imtcp")
    #input(type="imtcp" port="514")


    ###########################
    #### GLOBAL DIRECTIVES ####
    ###########################

    #
    # Use traditional timestamp format.
    # To enable high precision timestamps, comment out the following line.
    #
    $ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

    #
    # Set the default permissions for all log files.
    #
    $FileOwner root
    $FileGroup adm
    $FileCreateMode 0640
    $DirCreateMode 0755
    $Umask 0022

    #
    # Where to place spool and state files
    #
    $WorkDirectory /var/spool/rsyslog

    #
    # Include all config files in /etc/rsyslog.d/
    #
    $IncludeConfig /etc/rsyslog.d/*.conf


    ###############
    #### RULES ####
    ###############

    #
    # First some standard log files.  Log by facility.
    #
    auth,authpriv.*			/var/log/auth.log
    *.*;auth,authpriv.none		-/var/log/syslog
    #cron.*				/var/log/cron.log
    daemon.*			-/var/log/daemon.log
    kern.*				-/var/log/kern.log
    lpr.*				-/var/log/lpr.log
    mail.*				-/var/log/mail.log
    user.*				-/var/log/user.log

    #
    # Logging for the mail system.  Split it up so that
    # it is easy to write scripts to parse these files.
    #
    mail.info			-/var/log/mail.info
    mail.warn			-/var/log/mail.warn
    mail.err			/var/log/mail.err

    #
    # Some "catch-all" log files.
    #
    *.=debug;\
        auth,authpriv.none;\
        mail.none		-/var/log/debug
    *.=info;*.=notice;*.=warn;\
        auth,authpriv.none;\
        cron,daemon.none;\
        mail.none		-/var/log/messages

    #
    # Emergencies are sent to everybody logged in.
    #
    *.emerg				:omusrmsg:*
    *.* @192.168.34.109

    service rsyslog restart

    logger "This is a test event!"

Na prvem navideznem računalniku se nam sedaj pojavi nov dogodek v datoteki `/var/log/syslog`.

    tail -f /var/log/syslog
    Apr  4 14:57:06 debian systemd[1]: Started System Logging Service.
    Apr  4 14:59:58 debian systemd[1]: Stopping System Logging Service...
    Apr  4 14:59:58 debian systemd[1]: rsyslog.service: Succeeded.
    Apr  4 14:59:58 debian rsyslogd: [origin software="rsyslogd" swVersion="8.2102.0" x-pid="509" x-info="https://www.rsyslog.com"] exiting on signal 15.
    Apr  4 14:59:58 debian systemd[1]: Stopped System Logging Service.
    Apr  4 14:59:58 debian systemd[1]: Starting System Logging Service...
    Apr  4 14:59:58 debian rsyslogd: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3) from systemd.  [v8.2102.0]
    Apr  4 14:59:58 debian rsyslogd: [origin software="rsyslogd" swVersion="8.2102.0" x-pid="3603" x-info="https://www.rsyslog.com"] start
    Apr  4 14:59:58 debian systemd[1]: Started System Logging Service.
    Apr  4 15:10:28 debian aleks: This is a test event!

###

