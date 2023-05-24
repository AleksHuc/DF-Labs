# 13. Vaja: Steganografija

## Navodila

1. Skrijte tekstoven podatke v naključno sliko in jih nato tudi pravilno izluščite.

## Dodatne informacije

## Podrobna navodila

### 1. Steganografija

Tajne podatke so ljudje že v zgodovini poskusili na več različnih načinov, prav tako lahko podatke skrivamo na računalnikih:

- Skrivanje podatkov na nepričakovanih mestih:
    - Na prostem prostoru na disku, ki se nahaja izven določenih razdelkov.
    - Na prostem prostoru na [navideznih diskih](https://github.com/polz113/virtual_disk_injector).
- Skrivanje podatkov v dvoumnostih:
    - S spreminjanjem vrstnega reda zapisa podatkov v datotečnem sistemu, kjer imamo vozlišča (inode) z tabelo, ki kaže na bloke, ki sestavljajo datoteko. Podatke skrivamo v dvoumnosti tako, da podatke zapišemo podatke v drugem vrstnem redu in tako lahko dobimo nekaj prostora za skrivanje podatkov.
    - Izbira strojnih ukazov, saj na primer imata ukaza `mov ax 0` in `xor ax ax` imata enak rezultat.  
- Skrivanje podatkov s kvarjenjem podatkov:
    - V formatih, ki omogočajo majhno izgubo podatkov lahko skrijemo podatke. V slikah, na primer, rahlo spremenimo barvo posameznih pikslov in tako lahko skrijemo podatke.

[Steganografija](https://en.wikipedia.org/wiki/Steganography) je področje, ki se ukvarja s skrivanjem podatkov v drugih podatkih, medtem ko je [kriptografija](https://en.wikipedia.org/wiki/Cryptography) področje, ki se ukvarja s spreminjanjem podatkov tako, da so neberljivi tretjim osebam. V splošnem združimo oba pristopa, kjer podatke najprej zašifriramo in jih šele nato skrijemo. Na voljo je orodje [OpenPuff](https://embeddedsw.net/OpenPuff_Steganography_Home.html), ki vam omogoča preizkušanje različnih steganografskih in kriptografskih metod. Ena izmed najbolj poznanih steganografskih metod je skrivanje v [najmanj pomemben bit](https://www.boiteaklou.fr/Steganography-Least-Significant-Bit.html) nosilca podatkovv brez izgubnem (lossless) formatu, kjer podatke zapisujemo samo v najmanj pomembne bite. S to metodo v povprečju pokvarimo le 50% najmanj pomembnih bitov. Za izgubne (lossy) formate imamo na voljo druge pristope.

Steganoanaliza predstavlja področje, odkrivanja skritih podatkov s pristopi:

- Pregledovanja entropije.
- Statističnimi metodami.
- Primerjava nosilca podatkov z originalnim nosilcem podatkov.
- Pregledovanje podatkov na najbolj spremenljivih mestih nosilca podatkov.

Sedaj bomo v programskem jeziku `python` napisali program, ki bo znal skriti besedilo v sliko v brez izgubnem formatu in ga bo znal iz slike tudi izluščiti. Slike je sestavljena iz pikslov, in vsak piksel in barvo določeno na podlagi komponent [rdeče, zelene in modre (RGB)](https://en.wikipedia.org/wiki/RGB_color_model) barve. Vsaka komponenta je prestavljena z osmimi biti, torej piksel predstavimo z 24 biti. V vsaki komponenti lahko uporabimo najmanj pomemben bit za skrivanje podatkov, torej lahko v en piksel skrijemo 3 bite. V splošnem lahko shranimo v sliko za eno osmino velikosti slike skritih podatkov ($n*24/n*3 = 1/8$).

Za podatkovni nosilec izberemo poljubno sliko in jo pretvorimo v brez izgubni format [`.bmp`](https://en.wikipedia.org/wiki/BMP_file_format) z ukazom `convert`.

    apt update
    apt install libpng-dev libjpeg-dev libtiff-dev imagemagick
    wget https://i.redd.it/urj7zxd7dt0b1.jpg
    convert image.jpg image.bmp

Sedaj napišemo naš program, ki je sestavljen iz metode `hide`, ki skrije sporočilo v slike, metode `extract`, ki iz slike izlušči skrito sporočilo in metode `main`, ki požene program.

Metoda `hide` prejme sporočilo in sliko. Sliko pretvorimo v bajte in ustvarimo novo sliko z enakimi lastnostmi. Skrivnemu sporočilu dodamo na začetek dolžino skrivnega sporočila in nato v zanki zapisujemo skrivno sporočilo v najmanj pomembne bite originalne slike in jih shranjujemo v novo sliko. Metoda na koncu vrne novo sliko v kateri je bilo skrito sporočilo.

Metoda `extract` prejme sliko iz katere izlušči skrito sporočilo, tako da v zanki bere bajt po bajt in shranjuje najmanj pomembne bite v bajte skritega sporočila. Ko prebere štiri bajte iz njih izlušči dolžino skritega sporočila v bajtih, nato v zanki prebere še celotno skrito sporočilo, ki ga nato vrne.

    apt install python3-pip
    pip install Pillow

    nano steganography.py

    #!/bin/env python3

    from PIL import Image

    def hide(hidden_message, original_image):

        image_data = original_image.tobytes()
        new_data = b""
        new_image = Image.new(original_image.mode, original_image.size)
        hidden_message = len(hidden_message).to_bytes(4, byteorder='big') + hidden_message
        idx = 0
        for i, d in enumerate(image_data):

            byte_position = i // 8
            bit_position = i % 8
            mask = 1 << bit_position
            next_b = d
            if byte_position < len(hidden_message):
                cb = hidden_message[byte_position] & mask
                if cb == 0:
                    next_b = next_b & 0b11111110
                else:
                    next_b = next_b | 0b00000001
            else:
                break
            new_data += next_b.to_bytes(1, "big")
            idx = i
        new_data += image_data[idx+1:]
        new_image.frombytes(new_data)
        return new_image


    def extract(image):

        image_data = image.tobytes()
        hidden_message = b""
        length = len(image_data)
        check = True
        for i, c in enumerate(image_data):
            if i % 8 == 0:
                if i > 0:
                    hidden_message += next_d.to_bytes(1, "big")
                next_d = 0
            mask = 1 << (i % 8)
            if c & 0b00000001:
                next_d = next_d | mask
            if check and len(hidden_message) == 4:
                print(hidden_message)
                length = i + int.from_bytes(hidden_message, byteorder='big') * 8
                hidden_message = b""
                check = False
            if length == i:
                break
        return hidden_message


    if __name__ == "__main__":
        print("Steganography")
        message = "This is the hidden message that I want to hide inside of a picture!"
        print("Message to hide: ", message)
        original = Image.open("image.bmp")
        # original.show()
        print("Hiding the message")
        new = hide(message.encode("utf-8"), original)
        # new.show()
        new.save("new.bmp")
        print("Extracting the hidden message")
        print("Extracted message: ", extract(new))

    chmod +x steganography.py

    ./steganography.py