# 6. Vaja: Beleženje pod operacijskim sistemom Linux

## Navodila

1. 

## Dodatne informacije

## Podrobna navodila

### Beleženje pod operacijskim sistemom Linux

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