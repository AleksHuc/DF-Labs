# 2. Vaja: Diski in diskovni sistemi

## Navodila

1. Kako podatke shranjujemo na računalnikih in kako so urejeni?

## Dodatne informacije

## Podrobna navodila

### 1. Naloga

#### Kako so sestavljeni diski

Prvi diski na IBM kompatibilnih osebnih računalnikih so bili na računalnik priključeni tako, da je računalnik povsem nadziral disk. Računalnik je torej povedal, kam naj se premakne glava, preštel sektorje in prebral podatke. Do povezovanje diskov se je uporabljal na primer vmesnik [ST506/ST412](https://en.wikipedia.org/wiki/ST506/ST412#Connector_pinouts) ter podatki so se zapisovalni na disk s kodiranjem osnovanim na modificirani frekvenčni modulaciji ([Modified frequency modulation - MFM](https://en.wikipedia.org/wiki/Modified_frequency_modulation)).

Zaradi enostavnosti je bilo potrebno za branje oz. pisanje podatkov podati sled (Cyllinder), glavo na disku (Head) in sektor (Sector), ki kažejo na željen sektor s podatki. Temo procesu rečemo CHS naslavljanje ([CHS Addressing](https://en.wikipedia.org/wiki/Cylinder-head-sector)), danes pa naslavljamo podatke na disku tako da podamo samo številko sektorja na katerem so zapisani.

- C: sled, ki ima vrednosti med 0 in 1023. Sled je koncentrični krog na diskovni glavi.
- H: glava, Ki ima vrednosti med 0 in 254. Glava nam določi stran in število diskovnih plošč v disku. 
- S: sektor, ki ima vrednosti med 1 in 63. Vsaka sled je razdeljene na sektorje.

Prvi sektor je namenjen za sistemski nalagalnik in tabelo razdelkov. Sektor je imel privzeto velikost $512B$, novejši diski pa imajo sektorje velikosti $4096B$. Če predpostavimo velikost sektorja $512B$, potem lahko za CHS naslovimo maksimalno $1024*255*63*512 = 8,422,686,720B$.

