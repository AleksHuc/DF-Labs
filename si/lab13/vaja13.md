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

[Steganografija](https://en.wikipedia.org/wiki/Steganography) je področje, ki se ukvarja s skrivanjem podatkov v drugih podatkih, medtem ko je [kriptografija](https://en.wikipedia.org/wiki/Cryptography) področje, ki se ukvarja s spreminjanjem podatkov tako, da so neberljivi tretjim osebam. V splošnem združimo oba pristopa, kjer podatke najprej zašifriramo in jih šele nato skrijemo. Na voljo je orodje [OpenPuff](https://embeddedsw.net/OpenPuff_Steganography_Home.html), ki vam omogoča preizkušanje različnih steganografskih in kriptografskih metod. Ena izmed najbolj poznanih steganografskih metod je skrivanje v [najmanj pomemben bit](https://www.boiteaklou.fr/Steganography-Least-Significant-Bit.html) nosilca podatkov v brez izgubnem (lossless) formatu, kjer podatke zapisujemo samo v najmanj pomembne bite. S to metodo v povprečju pokvarimo le 50% najmanj pomembnih bitov. Za izgubne (lossy) formate imamo na voljo druge pristope.

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

        # Convert image to bytes
        image_data = original_image.tobytes()
        # New binary variable
        new_data = b""
        # Create new image of the same type and size as original
        new_image = Image.new(original_image.mode, original_image.size)
        # Concatenate the length of the hidden message and the hidden message
        hidden_message = len(hidden_message).to_bytes(4, byteorder='big') + hidden_message

        # Iterate over the image bytes
        idx = 0
        for i, d in enumerate(image_data):

            # Current byte position of the hidden message
            byte_position = i // 8
            # Current bit position of the hidden message
            bit_position = i % 8
            # Create bit mask from the current bit position
            mask = 1 << bit_position
            # Current image byte
            next_b = d

            # Check if current byte position is still in the hidden message, otherwise break
            if byte_position < len(hidden_message):

                # Get the value a of the current bit from the current byte of the hidden message
                cb = hidden_message[byte_position] & mask

                # If value at the current bit is 0 then set the last bit of the current image byte also to zero, otherwise set it to 1
                if cb == 0:
                    next_b = next_b & 0b11111110

                else:
                    next_b = next_b | 0b00000001

            else:
                break

            # Add the modified current image byte to the binary data of the new image.
            new_data += next_b.to_bytes(1, "big")
            idx = i

        # Add all the remaining image bytes to the modified image bytes
        new_data += image_data[idx+1:]

        # Create new image from binary data and return it
        new_image.frombytes(new_data)
        return new_image


    def extract(image):

        # Convert image to bytes
        image_data = image.tobytes()
        # New binary variable
        hidden_message = b""
        # Get the length of the image in bytes
        length = len(image_data)
        # Boolean variable for switching from reading length to data of the hidden message
        check = True

        # Iterate over the image bytes
        for i, c in enumerate(image_data):

            # Every 8 bits write them as byte to hidden message variable
            if i % 8 == 0:
                if i > 0:
                    hidden_message += next_d.to_bytes(1, "big")

                # Set current byte to 0
                next_d = 0

            # Create mask for the current bit
            mask = 1 << (i % 8)

            # Check if last bit in the current image byte is 1
            if c & 0b00000001:
                # Set the current bit to 1
                next_d = next_d | mask

            # Check if we are still waiting for the length to be read and current length of the hidden>
            if check and len(hidden_message) == 4:

                # Read the length of the hidden message from 4 bytes as integer
                length = i + int.from_bytes(hidden_message, byteorder='big') * 8
                # Reset the hidden message variable
                hidden_message = b""
                # Set the boolean variable to False
                check = False

            # Check if we have come to the end of the hidden message and break
            if length == i:
                break

        # Return the hidden message
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

Sedaj primerjajte vrednosti individualnih pikslov med originalno in spremenjeno sliko s programom [Krita](https://krita.org/).