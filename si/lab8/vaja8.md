# 8. Vaja: Zgodovina brskalnikov

## Navodila

1. Ugotovite, katere spletne strani je obiskal uporabnik in kdaj. Uporabljeni brskalniki so bili Firefox, Chrome in Edge.

## Dodatne informacije

## Podrobna navodila

### 1. Zgodovina brskalnikov

Prenesite [arhiv](https://ucilnica.fri.uni-lj.si/mod/resource/view.php?id=28964) uporabniških datotek s sistema z Windows 10. Odpremo paket z ukazom [tar](https://linux.die.net/man/1/tar).

    ls /home/aleks/Downloads/
    
    users.tar.xz

    cd /home/aleks/Downloads/

    tar -xf users.tar.xz

    ls

    Users users.tar.xz

    chmod -R ugoa+rwx Users/

Mozilla Firefox hrani večino zanimivih podatkov v bazah [SQLite](https://sqlite.org/index.html). [Zgodovino brskanja](https://www.foxtonforensics.com/browser-history-examiner/firefox-history-location) hrani v datoteki `places.sqlite`, ki jo najdemo v:

- Windows XP: `C:\Documents and Settings\<USERNAME>\Application Data\Mozilla\Firefox\Profiles\<PROFILE>\places.sqlite`
- Windows Vista (in od Windows 7 naprej): `C:\Users\<USERNAME>\AppData\Roaming\Mozilla\Firefox\Profiles\<PROFILE>\places.sqlite`
- Linux: `~/.mozilla/firefox/<PROFILE>/places.sqlite`
- OS X: `/Users/<USERNAME>/Library/Application Support/Firefox/Profiles/<PROFILE>/places.sqlite`

Za pregledovanje SQLite podatkovnih baz uporabite orodje za ukazno vrstico [`sqlite3`](https://linux.die.net/man/1/sqlite3) ali orodje z grafičnim vmesnikom [`sqlitebrowser`](https://manpages.debian.org/stretch/sqlitebrowser/sqlitebrowser.1).

    apt update
    apt install sqlite3 sqlitebrowser

Program odpremo tako, da pritisnemo na gumb `Activities` v zgornjem levem kotu in nato na program `DB Browser for SQLite`. Znotraj programa sedaj pritisnemo na gumb `Open Database` in odpremo želeno datoteko, na primer `Users\<USERNAME>\AppData\Roaming\Mozilla\Firefox\Profiles\<PROFILE>\places.sqlite`.

![Odprta SQLite podatkovna baza.](slike/vaja8-dbbrowser1.png)

Za pregledovanje posameznih tabel se prestavimo v zavihek `Browse Data` in v izvlečno spustnem meniju `Table:` izberemo želeno tabelo.

![Izpis posamezne SQLite tabele.](slike/vaja8-dbbrowser2.png)

Tudi Google Chrome hrani [zgodovino brskanja](https://www.foxtonforensics.com/browser-history-examiner/chrome-history-location) v bazah SQLite. Uporabniške datoteke se nahajajo v:

- Linux: `~/.config/chromium/`
- Windows Vista (in od Windows 7 naprej): `C:\Users\<USERNAME>\AppData\Local\Google\Chrome\User Data\Default\History`
- Windows XP: `C:\Documents and Settings\<USERNAME>\Local Settings\Application Data\Google\Chrome\User Data\Default\History`

Do zgodovine brskanja lahko prav tako dostopamo z orodji, kot sta `sqlite3` in `DB Browser for SQLite`.

Starejše različice Edge brskalnika so hranile [podatke o brskanju](https://www.foxtonforensics.com/browser-history-examiner/microsoft-edge-history-location) v bazi [ESE](https://en.wikipedia.org/wiki/Extensible_Storage_Engine). Datoteke se nahajajo na različnih mestih, med drugim v:

- Windows 10: `C:\Users\<USERNAME>\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat` in `C:\Users\<USERNAME>\AppData\Local\Packages\Microsoft.MicrosoftEdge_<ID>\AC\MicrosoftEdge\User\Default\`

Do podatkovnih baz ESE lahko dostopamo s knjižnico [libesedb](https://github.com/libyal/libesedb) in priloženimi programi.

Novejše različice Edge brskalnika so osnovane na Google Chrome brskalniku in hranijo podatke na enak način in sicer na mestu:

- Windows 10: `C:\Users\<USERNAME>\AppData\Local\Microsoft\Edge\User Data\Default\`

Poglejmo si še starejši spletni brskalnik Internet Explorer 5, ki hrani podatke v:

- Windows XP: `C:\Documents and Settings\<USERNAME>\Local Settings\History\History.IE5\`
- Windows 7, 8, 10: `C:\Users\<USERNAME>\AppData\Local\Microsoft\Internet Explorer\Recovery`, `C:\Users\<USERNAME>\AppData\Local\Microsoft\Windows\WebCache` in `C:\Users\<USERNAME>\Favorites`

Zgodovino lahko preberemo z orodjem [`pasco`](https://www.unix.com/man-page/debian/1/pasco).

    apt update
    apt install pasco

    cd /mnt/Documents\ and\ Settings/user/Local\ Settings/History/History.IE5/
    pasco index.dat
