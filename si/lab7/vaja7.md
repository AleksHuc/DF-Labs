# 6. Vaja: Beleženje pod operacijskim sistemom Linux

## Navodila

1. Poišči datoteke, ki beležijo dogodke pod operacijskim sistemom Linux.
2. Pošlji beležke v realnem času preko mreže med dvema navideznima računalnikoma.
3. Implementiraj program, ki bere pritiske tipkovnice in jih pošilja v sistemski dnevnik.

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

    alternatives.log    btmp	  fontconfig.log  speech-dispatcher
    alternatives.log.1  btmp.1	  gdm3		  syslog
    apache2		    cups	  installer	  syslog.1
    apt		    daemon.log	  journal	  unattended-upgrades
    auth.log	    daemon.log.1  kern.log	  user.log
    auth.log.1	    debug	  kern.log.1	  user.log.1
    boot.log	    debug.1	  lastlog	  vboxadd-install.log
    boot.log.1	    dpkg.log	  messages	  vboxadd-setup.log
    boot.log.2	    dpkg.log.1	  messages.1	  wtmp
    boot.log.3	    faillog	  private

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

### 3. Program za beleženje pritiskov na tipkovnici

Najprej moramo ugotoviti, kako sploh pridemo do dogodkov, ki jih proži tipkovnica. Na operacijskem sistemu Linux imamo mapo `/dev`, ki vsebuje vse naprave, medtem ko mapa `/dev/input` vsebuje vse vhodne naprave. Vse naprave, ki prožije dogodke pa so navedene v datoteki `/proc/bus/input/devices`

    ls /dev

    autofs		 input	       sg0	 tty20	tty41  tty62	  vcsa2
    block		 kmsg	       sg1	 tty21	tty42  tty63	  vcsa3
    bsg		 log	       shm	 tty22	tty43  tty7	  vcsa4
    btrfs-control	 loop-control  snapshot  tty23	tty44  tty8	  vcsa5
    bus		 mapper        snd	 tty24	tty45  tty9	  vcsa6
    cdrom		 mem	       sr0	 tty25	tty46  ttyS0	  vcsu
    char		 mqueue        stderr	 tty26	tty47  ttyS1	  vcsu1
    console		 net	       stdin	 tty27	tty48  ttyS2	  vcsu2
    core		 null	       stdout	 tty28	tty49  ttyS3	  vcsu3
    cpu		 nvram	       tty	 tty29	tty5   uhid	  vcsu4
    cpu_dma_latency  port	       tty0	 tty3	tty50  uinput	  vcsu5
    cuse		 ppp	       tty1	 tty30	tty51  urandom	  vcsu6
    disk		 psaux	       tty10	 tty31	tty52  vboxguest  vfio
    dri		 ptmx	       tty11	 tty32	tty53  vboxuser   vga_arbiter
    dvd		 pts	       tty12	 tty33	tty54  vcs	  vhci
    fb0		 random        tty13	 tty34	tty55  vcs1	  vhost-net
    fd		 rfkill        tty14	 tty35	tty56  vcs2	  vhost-vsock
    full		 rtc	       tty15	 tty36	tty57  vcs3	  zero
    fuse		 rtc0	       tty16	 tty37	tty58  vcs4
    hidraw0		 sda	       tty17	 tty38	tty59  vcs5
    hpet		 sda1	       tty18	 tty39	tty6   vcs6
    hugepages	 sda2	       tty19	 tty4	tty60  vcsa
    initctl		 sda5	       tty2	 tty40	tty61  vcsa1

    ls /dev/input

    by-id	 event0  event2  event4  event6  js0  mice    mouse1
    by-path  event1  event3  event5  event7  js1  mouse0

    cat /proc/bus/input/devices

    I: Bus=0011 Vendor=0001 Product=0001 Version=ab41
    N: Name="AT Translated Set 2 keyboard"
    P: Phys=isa0060/serio0/input0
    S: Sysfs=/devices/platform/i8042/serio0/input/input0
    U: Uniq=
    H: Handlers=sysrq kbd leds event0 
    B: PROP=0
    B: EV=120013
    B: KEY=402000000 3803078f800d001 feffffdfffefffff fffffffffffffffe
    B: MSC=10
    B: LED=7

    I: Bus=0019 Vendor=0000 Product=0001 Version=0000
    N: Name="Power Button"
    P: Phys=LNXPWRBN/button/input0
    S: Sysfs=/devices/LNXSYSTM:00/LNXPWRBN:00/input/input2
    U: Uniq=
    H: Handlers=kbd event1 
    B: PROP=0
    B: EV=3
    B: KEY=10000000000000 0

    I: Bus=0019 Vendor=0000 Product=0006 Version=0000
    N: Name="Video Bus"
    P: Phys=LNXVIDEO/video/input0
    S: Sysfs=/devices/LNXSYSTM:00/LNXSYBUS:00/PNP0A03:00/LNXVIDEO:00/input/input3
    U: Uniq=
    H: Handlers=kbd event2 
    B: PROP=0
    B: EV=3
    B: KEY=3e000b00000000 0 0 0

    I: Bus=0019 Vendor=0000 Product=0003 Version=0000
    N: Name="Sleep Button"
    P: Phys=LNXSLPBN/button/input0
    S: Sysfs=/devices/LNXSYSTM:00/LNXSLPBN:00/input/input4
    U: Uniq=
    H: Handlers=kbd event3 
    B: PROP=0
    B: EV=3
    B: KEY=4000 0 0

    I: Bus=0011 Vendor=0002 Product=0006 Version=0000
    N: Name="ImExPS/2 Generic Explorer Mouse"
    P: Phys=isa0060/serio1/input0
    S: Sysfs=/devices/platform/i8042/serio1/input/input5
    U: Uniq=
    H: Handlers=mouse0 event4 
    B: PROP=1
    B: EV=7
    B: KEY=1f0000 0 0 0 0
    B: REL=143

    I: Bus=0003 Vendor=80ee Product=0021 Version=0110
    N: Name="VirtualBox USB Tablet"
    P: Phys=usb-0000:00:06.0-1/input0
    S: Sysfs=/devices/pci0000:00/0000:00:06.0/usb1/1-1/1-1:1.0/0003:80EE:0021.0001/input/input6
    U: Uniq=
    H: Handlers=mouse1 event5 js0 
    B: PROP=0
    B: EV=1f
    B: KEY=1f0000 0 0 0 0
    B: REL=1940
    B: ABS=3
    B: MSC=10

    I: Bus=0001 Vendor=80ee Product=cafe Version=0601
    N: Name="VirtualBox mouse integration"
    P: Phys=
    S: Sysfs=/devices/pci0000:00/0000:00:04.0/input/input7
    U: Uniq=
    H: Handlers=event6 js1 
    B: PROP=0
    B: EV=b
    B: KEY=10000 0 0 0 0
    B: ABS=3

    I: Bus=0010 Vendor=001f Product=0001 Version=0100
    N: Name="PC Speaker"
    P: Phys=isa0061/input0
    S: Sysfs=/devices/platform/pcspkr/input/input8
    U: Uniq=
    H: Handlers=kbd event7 
    B: PROP=0
    B: EV=40001
    B: SND=6

V naše primeru tipkovnica pošilja dogodke na vhod `event0`. Da ugotovimo kakšne dogodke proži tipkovnica namestimo program [`evtest`](https://manpages.org/evtest) z upravljalcem paketov našega operacijskega sistema.

    apt update
    apt install evtest

    evtest

    No device specified, trying to scan all of /dev/input/event*
    Available devices:
    /dev/input/event0:	AT Translated Set 2 keyboard
    /dev/input/event1:	Power Button
    /dev/input/event2:	Video Bus
    /dev/input/event3:	Sleep Button
    /dev/input/event4:	ImExPS/2 Generic Explorer Mouse
    /dev/input/event5:	VirtualBox USB Tablet
    /dev/input/event6:	VirtualBox mouse integration
    /dev/input/event7:	PC Speaker
    Select the device event number [0-7]: 0
    Input driver version is 1.0.1
    Input device ID: bus 0x11 vendor 0x1 product 0x1 version 0xab41
    Input device name: "AT Translated Set 2 keyboard"
    Supported events:
    Event type 0 (EV_SYN)
    Event type 1 (EV_KEY)
        Event code 1 (KEY_ESC)
        Event code 2 (KEY_1)
        Event code 3 (KEY_2)
        Event code 4 (KEY_3)
        Event code 5 (KEY_4)
        Event code 6 (KEY_5)
        Event code 7 (KEY_6)
        Event code 8 (KEY_7)
        Event code 9 (KEY_8)
        Event code 10 (KEY_9)
        Event code 11 (KEY_0)
        Event code 12 (KEY_MINUS)
        Event code 13 (KEY_EQUAL)
        Event code 14 (KEY_BACKSPACE)
        Event code 15 (KEY_TAB)
        Event code 16 (KEY_Q)
        Event code 17 (KEY_W)
        Event code 18 (KEY_E)
        Event code 19 (KEY_R)
        Event code 20 (KEY_T)
        Event code 21 (KEY_Y)
        Event code 22 (KEY_U)
        Event code 23 (KEY_I)
        Event code 24 (KEY_O)
        Event code 25 (KEY_P)
        Event code 26 (KEY_LEFTBRACE)
        Event code 27 (KEY_RIGHTBRACE)
        Event code 28 (KEY_ENTER)
        Event code 29 (KEY_LEFTCTRL)
        Event code 30 (KEY_A)
        Event code 31 (KEY_S)
        Event code 32 (KEY_D)
        Event code 33 (KEY_F)
        Event code 34 (KEY_G)
        Event code 35 (KEY_H)
        Event code 36 (KEY_J)
        Event code 37 (KEY_K)
        Event code 38 (KEY_L)
        Event code 39 (KEY_SEMICOLON)
        Event code 40 (KEY_APOSTROPHE)
        Event code 41 (KEY_GRAVE)
        Event code 42 (KEY_LEFTSHIFT)
        Event code 43 (KEY_BACKSLASH)
        Event code 44 (KEY_Z)
        Event code 45 (KEY_X)
        Event code 46 (KEY_C)
        Event code 47 (KEY_V)
        Event code 48 (KEY_B)
        Event code 49 (KEY_N)
        Event code 50 (KEY_M)
        Event code 51 (KEY_COMMA)
        Event code 52 (KEY_DOT)
        Event code 53 (KEY_SLASH)
        Event code 54 (KEY_RIGHTSHIFT)
        Event code 55 (KEY_KPASTERISK)
        Event code 56 (KEY_LEFTALT)
        Event code 57 (KEY_SPACE)
        Event code 58 (KEY_CAPSLOCK)
        Event code 59 (KEY_F1)
        Event code 60 (KEY_F2)
        Event code 61 (KEY_F3)
        Event code 62 (KEY_F4)
        Event code 63 (KEY_F5)
        Event code 64 (KEY_F6)
        Event code 65 (KEY_F7)
        Event code 66 (KEY_F8)
        Event code 67 (KEY_F9)
        Event code 68 (KEY_F10)
        Event code 69 (KEY_NUMLOCK)
        Event code 70 (KEY_SCROLLLOCK)
        Event code 71 (KEY_KP7)
        Event code 72 (KEY_KP8)
        Event code 73 (KEY_KP9)
        Event code 74 (KEY_KPMINUS)
        Event code 75 (KEY_KP4)
        Event code 76 (KEY_KP5)
        Event code 77 (KEY_KP6)
        Event code 78 (KEY_KPPLUS)
        Event code 79 (KEY_KP1)
        Event code 80 (KEY_KP2)
        Event code 81 (KEY_KP3)
        Event code 82 (KEY_KP0)
        Event code 83 (KEY_KPDOT)
        Event code 85 (KEY_ZENKAKUHANKAKU)
        Event code 86 (KEY_102ND)
        Event code 87 (KEY_F11)
        Event code 88 (KEY_F12)
        Event code 89 (KEY_RO)
        Event code 90 (KEY_KATAKANA)
        Event code 91 (KEY_HIRAGANA)
        Event code 92 (KEY_HENKAN)
        Event code 93 (KEY_KATAKANAHIRAGANA)
        Event code 94 (KEY_MUHENKAN)
        Event code 95 (KEY_KPJPCOMMA)
        Event code 96 (KEY_KPENTER)
        Event code 97 (KEY_RIGHTCTRL)
        Event code 98 (KEY_KPSLASH)
        Event code 99 (KEY_SYSRQ)
        Event code 100 (KEY_RIGHTALT)
        Event code 102 (KEY_HOME)
        Event code 103 (KEY_UP)
        Event code 104 (KEY_PAGEUP)
        Event code 105 (KEY_LEFT)
        Event code 106 (KEY_RIGHT)
        Event code 107 (KEY_END)
        Event code 108 (KEY_DOWN)
        Event code 109 (KEY_PAGEDOWN)
        Event code 110 (KEY_INSERT)
        Event code 111 (KEY_DELETE)
        Event code 112 (KEY_MACRO)
        Event code 113 (KEY_MUTE)
        Event code 114 (KEY_VOLUMEDOWN)
        Event code 115 (KEY_VOLUMEUP)
        Event code 116 (KEY_POWER)
        Event code 117 (KEY_KPEQUAL)
        Event code 118 (KEY_KPPLUSMINUS)
        Event code 119 (KEY_PAUSE)
        Event code 121 (KEY_KPCOMMA)
        Event code 122 (KEY_HANGUEL)
        Event code 123 (KEY_HANJA)
        Event code 124 (KEY_YEN)
        Event code 125 (KEY_LEFTMETA)
        Event code 126 (KEY_RIGHTMETA)
        Event code 127 (KEY_COMPOSE)
        Event code 128 (KEY_STOP)
        Event code 140 (KEY_CALC)
        Event code 142 (KEY_SLEEP)
        Event code 143 (KEY_WAKEUP)
        Event code 155 (KEY_MAIL)
        Event code 156 (KEY_BOOKMARKS)
        Event code 157 (KEY_COMPUTER)
        Event code 158 (KEY_BACK)
        Event code 159 (KEY_FORWARD)
        Event code 163 (KEY_NEXTSONG)
        Event code 164 (KEY_PLAYPAUSE)
        Event code 165 (KEY_PREVIOUSSONG)
        Event code 166 (KEY_STOPCD)
        Event code 172 (KEY_HOMEPAGE)
        Event code 173 (KEY_REFRESH)
        Event code 183 (KEY_F13)
        Event code 184 (KEY_F14)
        Event code 185 (KEY_F15)
        Event code 217 (KEY_SEARCH)
        Event code 226 (KEY_MEDIA)
    Event type 4 (EV_MSC)
        Event code 4 (MSC_SCAN)
    Event type 17 (EV_LED)
        Event code 0 (LED_NUML) state 0
        Event code 1 (LED_CAPSL) state 0
        Event code 2 (LED_SCROLLL) state 0
    Key repeat handling:
    Repeat type 20 (EV_REP)
        Repeat code 0 (REP_DELAY)
        Value    250
        Repeat code 1 (REP_PERIOD)
        Value     33
    Properties:
    Testing ... (interrupt to exit)
    Event: time 1680637986.976124, type 4 (EV_MSC), code 4 (MSC_SCAN), value 1c
    Event: time 1680637986.976124, type 1 (EV_KEY), code 28 (KEY_ENTER), value 0
    Event: time 1680637986.976124, -------------- SYN_REPORT ------------
    Event: time 1680637990.155637, type 4 (EV_MSC), code 4 (MSC_SCAN), value 20
    Event: time 1680637990.155637, type 1 (EV_KEY), code 32 (KEY_D), value 1
    Event: time 1680637990.155637, -------------- SYN_REPORT ------------
    dEvent: time 1680637990.303721, type 4 (EV_MSC), code 4 (MSC_SCAN), value 20
    Event: time 1680637990.303721, type 1 (EV_KEY), code 32 (KEY_D), value 0
    Event: time 1680637990.303721, -------------- SYN_REPORT ------------
    Event: time 1680637991.193166, type 4 (EV_MSC), code 4 (MSC_SCAN), value 21
    Event: time 1680637991.193166, type 1 (EV_KEY), code 33 (KEY_F), value 1
    Event: time 1680637991.193166, -------------- SYN_REPORT ------------
    fEvent: time 1680637991.324400, type 4 (EV_MSC), code 4 (MSC_SCAN), value 21
    Event: time 1680637991.324400, type 1 (EV_KEY), code 33 (KEY_F), value 0
    Event: time 1680637991.324400, -------------- SYN_REPORT ------------
    Event: time 1680637992.015830, type 4 (EV_MSC), code 4 (MSC_SCAN), value 24
    Event: time 1680637992.015830, type 1 (EV_KEY), code 36 (KEY_J), value 1
    Event: time 1680637992.015830, -------------- SYN_REPORT ------------
    jEvent: time 1680637992.175462, type 4 (EV_MSC), code 4 (MSC_SCAN), value 24
    Event: time 1680637992.175462, type 1 (EV_KEY), code 36 (KEY_J), value 0
    Event: time 1680637992.175462, -------------- SYN_REPORT ------------

Iz `eventX` preberemo po [16 B na enkrat ki hranijo](https://www.kernel.org/doc/Documentation/input/input.txt):
- Unsigned integer (4 B) - Čas v sekundah.
- Unsigned integer (4 B) - Čas v mikro sekundah.
- Short (2 B) - Tip dogodka.
- Short (2 B) - Koda dogodka.
- Integer (4 B) - Vrednost, ki opisuje dogodek.

Sedaj napišemo naš program v programskem jeziku Python, ki bere dogodke z vhodne naprave `/dev/input/event0`. Na enkrat prebere 16 B, ki predstavljajo sekunde, mikro sekunde, tip, kodo in vrednost. Ker želimo izpisovati samo po en dogodek ob pritisku na tipkovnico potem tvorimo dogodek le v primeru, ko ima tip vrednost 1 in vrednost in prav tako vrednost 1. Sedaj pretvorimo kodo tipke v tekstovni opis, za katerega rabimo knjižnico [`evdev`](https://pypi.org/project/evdev/) in izpišemo dogodek na standardni izhod. Nato še omogočimo, da se program zažene z ukazom [`chmod`](https://www.man7.org/linux/man-pages/man1/chmod.1.html).

    apt update
    apt install python3-evdev

    nano keylogger.py
    
    #!/usr/bin/env python3

    from evdev import ecodes
    import struct
    import sys

    with open("/dev/input/event0", "rb") as f:
            while True:
                    d = f.read(16)
                    sec, usec, type, code, value = struct.unpack("IIhhi", d)
                    if type == 1 and value == 1:
                            print(ecodes.KEY[code])
                            sys.stdout.flush()    
    
    chmod +x keylogger.py

Sedaj še preizkusimo delovanje, tako da v enem oknu ukazne vrstice poženemo naš program in preusmerimo njegov izhod v program `logger`. V drugem oknu ukazne vrstice pa poženemo izpisovanje sistemskih belež `/var/log/syslog` v realnem času ter začnemo pritiskati na poljubne tipke na tipkovnic..

    ./keylogger.py | logger

    tail -f /var/log/syslog

    Apr  4 22:22:26 debian aleks: KEY_T
    Apr  4 22:22:27 debian aleks: KEY_H
    Apr  4 22:22:27 debian aleks: KEY_I
    Apr  4 22:22:28 debian aleks: KEY_S
    Apr  4 22:22:28 debian aleks: KEY_SPACE
    Apr  4 22:22:29 debian aleks: KEY_I
    Apr  4 22:22:29 debian aleks: KEY_S
    Apr  4 22:22:30 debian aleks: KEY_SPACE
    Apr  4 22:22:30 debian aleks: KEY_A
    Apr  4 22:22:30 debian aleks: KEY_SPACE
    Apr  4 22:22:31 debian aleks: KEY_T
    Apr  4 22:22:31 debian aleks: KEY_E
    Apr  4 22:22:32 debian aleks: KEY_S
    Apr  4 22:22:32 debian aleks: KEY_T
