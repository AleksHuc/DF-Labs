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
    convert urj7zxd7dt0b1.jpg image.bmp

Now we write our program, which consists of the `hide` method, which hides the message in the images, the `extract` method, which extracts the hidden message from the image, and the `main` method, which runs the program.

The `hide` method receives a message and an image. We convert the image to bytes and create a new image with the same properties. We add the length of the secret message to the beginning of the secret message and then in a loop we write the secret message to the least significant bits of the original image and store them in the new image. The method returns a new image in which the message was hidden.

The `extract` method receives an image from which it extracts the hidden message, reading byte by byte in a loop and storing the least significant bits in the bytes of the hidden message. After reading four bytes, it extracts the length of the hidden message in bytes from them, then reads the entire hidden message in a loop, and then returns it.

    apt install python3-pillow

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

Now compare the values of the individual pixels of the original and modified image using the program [Krita](https://krita.org/).