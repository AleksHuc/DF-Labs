# 8. Vaja: Zgodovina brskalnikov

## Navodila

1. Ugotovite, katere spletne strani je obiskal uporabnik in kdaj. Uporabljeni brskalniki so bili Firefox, Chrome in Edge. Začnite s prostima programoma.

## Dodatne informacije

## Podrobna navodila

### 1. Zgodovina brskalnikov

Prenesite [arhiv](https://ucilnica.fri.uni-lj.si/mod/resource/view.php?id=28964) uporabniških datotek s sistema z Windows 10. 

Mozilla Firefox hrani večino zanimivih podatkov v bazah SQLite. Zgodovino brskanja hrani v datoteki places.sqlite, ki jo najdemo v:

- Windows XP: `C:\Documents and Settings\<USERNAME>\Application Data\Mozilla\Firefox\Profiles\<PROFILE>\places.sqlite`
- Windows Vista: `C:\Users\<USERNAME>\AppData\Roaming\Mozilla\Firefox\Profiles\<PROFILE>\places.sqlite`
- Linux: `~/.mozilla/firefox/<PROFILE>/places.sqlite`
- OS X: `/Users/<USERNAME>/Library/Application Support/Firefox/Profiles/<PROFILE>/places.sqlite`

Za lažje delo z bazami SQLite lahko uporabite program sqlitebrowser.


Tudi Google Chrome hrani podatke v bazah SQLite. Uporabniške datoteke se nahajajo v:

- Linux: `~/.config/chromium/`
- Windows Vista (in Windows 7): `C:\Users\<USERNAME>\AppData\Local\Google\Chrome\`
- Windows XP: `C:\Documents and Settings\<USERNAME>\Local Settings\Application Data\Google\Chrome\`

Microsoft Edge

Edge hrani podatke o brskanju v bazi ESE. Datoteke se nahajajo na različnih mestih, med drugim v:

- Windows 10: `C:\Users\<USERNAME>\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat` in `C:\Users\<USERNAME>\AppData\Local\Packages\Microsoft.MicrosoftEdge_<ID>\AC\MicrosoftEdge\User\Default\`

Za delo z bazami ESE lahko uporabite knjižnico libesedb in priložene programe.
