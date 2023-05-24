# 13. Lab: Steganography

## Instructions

1. Hide text data in a random image and then extract it correctly.

## More information

## Detailed instructions

### 1. Steganography

Throughout history, people have tried to hide secret data in many different ways, and data can also be hidden on computers:

- Hiding data in unexpected places:
     - On free space on the disk located outside the specified partitions.
     - Free space on [virtual disks](https://github.com/polz113/virtual_disk_injector).
- Hiding data in ambiguities:
     - By changing the order of data in the file system, where we have inodes with a table which points to the blocks that make up the file. We hide the data in ambiguity by writing the data in a different order and thus we can get some space to hide the data.
     - Choice of assembly commands, as for example commands `mov ax 0` and `xor ax ax` have the same result.
- Hiding data by corrupting data:
     - Data can be hidden in formats that allow little data loss. In images, for example, we slightly change the color of individual pixels and thus can hide data.

[Steganography](https://en.wikipedia.org/wiki/Steganography) is the field concerned with hiding data within other data, while [cryptography](https://en.wikipedia.org/wiki/Cryptography) is a field that deals with changing data so that it is unreadable by third parties. In general, we combine both approaches, where we first encrypt the data and only then hide it. There is a tool [OpenPuff](https://embeddedsw.net/OpenPuff_Steganography_Home.html) that allows you to try different steganographic and cryptographic methods. One of the best-known steganographic methods is hiding in the [least significant bit](https://www.boiteaklou.fr/Steganography-Least-Significant-Bit.html) of the data carrier in a lossless format, where the data is written only in the least significant bits. With this method, on average, we corrupt only 50% of the least significant bits. We have other approaches available for lossy formats.

Steganoanalysis represents the field of discovering hidden data with the following approaches:

- Entropy evaluation.
- Statistical methods.
- Comparison of the data carrier with the original data carrier.
- Viewing data in the most variable places of the data carrier.

Now we will write a program in the `python` programming language that will be able to hide text in an image in a lossless format and will also be able to extract it from the image. An image is made up of pixels, and each pixel has a color determined based on the components [red, green, and blue (RGB)](https://en.wikipedia.org/wiki/RGB_color_model) of the color. Each component is represented by eight bits, so a pixel is represented by 24 bits. In each component, we can use the least significant bit to hide data, so we can hide 3 bits in one pixel. In general, we can store in the image for one eighth of the image size of the hidden data ($n*24/n*3 = 1/8$).

We choose an arbitrary image for the data carrier and convert it into a lossless format [`.bmp`](https://en.wikipedia.org/wiki/BMP_file_format) using the `convert` command.

    apt update
    apt install libpng-dev libjpeg-dev libtiff-dev imagemagick
    wget https://i.redd.it/urj7zxd7dt0b1.jpg
    convert image.jpg image.bmp

Now we write our program, which consists of the `hide` method, which hides the message in the images, the `extract` method, which extracts the hidden message from the image, and the `main` method, which runs the program.

The `hide` method receives a message and an image. We convert the image to bytes and create a new image with the same properties. We add the length of the secret message to the beginning of the secret message and then in a loop we write the secret message to the least significant bits of the original image and store them in the new image. The method returns a new image in which the message was hidden.

The `extract` method receives an image from which it extracts the hidden message, reading byte by byte in a loop and storing the least significant bits in the bytes of the hidden message. After reading four bytes, it extracts the length of the hidden message in bytes from them, then reads the entire hidden message in a loop, and then returns it.

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