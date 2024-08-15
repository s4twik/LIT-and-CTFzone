Okay so i 2 images, Kuwahara is used for noise reduction.

i read the encoder, i get the gist!

use claude to get better understanding

````
The module uses a set of characters (CHARACTERS) which includes spaces, underscores, curly braces, lowercase and uppercase letters, and digits.
It uses a base number system (BASE) set to 10 by default.
Key functions:
a) generate_table(): Creates encoder and decoder dictionaries for the given base and character set.
b) number_to_base(): Converts a decimal number to a list of digits in the specified base.
c) convert_from(): Converts a number from a given base back to decimal.
d) encode(): Encodes a message into a list of integers using the specified base and character set.
e) decode(): Decodes a list of integers back into the original message.
The encoding process:

Each character in the message is mapped to a unique number using the encoder table.
These numbers are then converted to the specified base.
The resulting digits are concatenated into a single list of integers.


The decoding process:

The list of integers is grouwiped into chunks.
Each chunk is converted back to decimal.
The decimal number is then mapped back to its corresponding character using the decoder table.


The module includes some tests to verify the encoding and decoding processes work correctly.
````
as far as i can understand, some sort of base encoding

okay, so it'll require bruteforcing

i am refering to a writeup atp

![image](https://github.com/user-attachments/assets/4f2b3b6b-8e80-4bd4-aed0-45de6fa427b0)

ahhh, now we decode (i regret typing this later)

with claude and some tinkering 
````
import numpy as np
from cv2 import cvtColor, COLOR_BGR2GRAY, sepFilter2D, imread, imwrite
from math import ceil, log
from string import ascii_lowercase, ascii_uppercase, digits

CHARACTERS = " _{}" + f"{ascii_lowercase}{ascii_uppercase}{digits}"
BASE = 3  # Changed to 3 as per the encoding in the main

def generate_table(base=BASE, characters=CHARACTERS):
    encoder = {}
    decoder = {}
    k = ceil(log(len(characters) / (base - 1), base)) + 1

    for i, ch in enumerate(characters):
        encoder[ch] = base ** k + i
        decoder[base ** k + i] = ch

    return encoder, decoder

def convert_from(num, base=BASE):
    s = 0
    for i, d in enumerate(num[::-1]):
        s += base**i * d
    return s

def decode(encoded, base=BASE, characters=CHARACTERS):
    _, decoder = generate_table(base, characters)
    seq = ""
    k = ceil(log(len(characters)/(base - 1), base)) + 1
    for i in range(0, len(encoded), k+1):
        num = encoded[i:i+k+1]
        conv = convert_from(num, base)
        seq += decoder[conv]
    return seq

def kuwahara(original_image, window_size):
    # This function should be implemented based on the description in the problem statement
    # For now, we'll return None, None as placeholders
    return None, None

def kuwahara_decryption(original_image, enc_image, window_size=5):
    averages, stddevs = kuwahara(original_image, window_size)
    indices = np.argmin(stddevs, axis=0)
    
    # Start at (window_size, window_size) to avoid the border
    start_x = start_y = window_size
    decoded_indices = []
    
    while start_x < enc_image.shape[0] - window_size:
        while start_y < enc_image.shape[1] - window_size:
            # Find the index of the region used in the encoded image
            encoded_index = np.argmin(np.abs(averages[:, start_x, start_y] - enc_image[start_x, start_y]))
            
            # Find the position of this index in the sorted stddevs
            sorted_indices = np.argsort(stddevs[:, start_x, start_y])
            position = np.where(sorted_indices == encoded_index)[0][0]
            
            decoded_indices.append(position)
            
            start_y += 1
        start_y = window_size
        start_x += 1
        
        # Break if we've found all the indices (assuming the message length is known)
        if len(decoded_indices) == len("FLAG"):  # Adjust this if the flag length is different
            break
    
    return decoded_indices


````

haar maan gaya sher
