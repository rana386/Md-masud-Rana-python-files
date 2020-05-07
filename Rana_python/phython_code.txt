import numpy as np
import pandas as pd
from skimage.io import imread, imshow
from skimage.transform import rescale
import skimage
import bitarray
import matplotlib.pyplot as plt

import os
print(os.listdir("../input/script-and-image"))





Scripts = ""

def TextToString(txt):
    with open (txt, "r", encoding='utf-16') as file:
        data=file.readlines()
        script = ""
        for x in data[1:]:
            x = x.replace('"','').replace("\n"," \n ").split(' ')
            x[1] += ":"
            script += " ".join(x[1:-1]).replace("\n"," \n ")
        return script
    
Scripts += TextToString("../input/script-and-image/secret_script.txt")

print(Scripts[:1000])



deathstar_img = imread("../input/script-and-image/download.png")

plt.figure(figsize=(10, 10))
plt.imshow(deathstar_img)

print("Image is "+str(deathstar_img.shape[0])+" by "+str(deathstar_img.shape[1])+" pixels with "+str(deathstar_img.shape[2])+" color channels")





def MessageToBits(message):
    #tag message (and pad w/ spaces till 10 characters)
    tag = "{:<10}".format(str(len(message)*8))
    message = tag+message
    #convert to bits
    code = bitarray.bitarray()
    code.frombytes(message.encode('utf-8'))
    code = "".join(['1' if x == True else '0' for x in code.tolist()])
    return code
	
	
	



def CheckBitSize(img, message):
    h = img.shape[0]
    w = img.shape[1]
    try:
        c = img.shape[2]
    except:
        c = 1
    image_max_size = h*w*c*2
    string_size = len(message)
    print("Message is "+str(string_size/8000)+" KB and image can fit "+str(image_max_size/8000)+" KB of data")
    if string_size > image_max_size:
        print("Message is too big to be encoded in image")
        return False
    else:
        print("Image can be encoded with message. Proceed")
        return True
    





def EncodeImage(img, message):
    code = MessageToBits(message)
    if CheckBitSize(img, code):
        shape = img.shape
        img = img.flatten()
        code = list(code)
        code_len = len(code)
        for i,x in enumerate(img):
            if i*2 <code_len:
                zbits = list('{0:08b}'.format(x))[:6]+code[i*2:i*2+2]
                img[i] = int("".join(zbits), 2)
            else:
                return img.reshape(shape)
        return img.reshape(shape)

encoded_img = EncodeImage(deathstar_img, Scripts)






def CompareTwoImages(img1,img2):
    fig=plt.figure(figsize=(20, 20))

    fig.add_subplot(2, 2, 1)
    plt.imshow(img1)
    fig.add_subplot(2, 2, 2)
    plt.imshow(img2)

    plt.show()
CompareTwoImages(deathstar_img, encoded_img)




print(deathstar_img[200][200])
print(encoded_img[200][200])





def DecodeImage(img):
    bit_message = ""
    bit_count = 0
    bit_length = 200
    for i,x in enumerate(img):
        for j,y in enumerate(x):
            for k,z in enumerate(y):
                zbits = '{0:08b}'.format(z)
                bit_message += zbits[-2:]
                bit_count += 2
                if bit_count == 80:
                    try:
                        decoded_tag = bitarray.bitarray(bit_message).tobytes().decode('utf-8')
                        bit_length = int(decoded_tag)+80
                        bit_message = ""
                    except:
                        print("Image does not have decode tag. Image is either not encoded or, at least, not encoded in a way this decoder recognizes")
                        return
                elif bit_count >= bit_length:
                    return bitarray.bitarray(bit_message).tobytes().decode('utf-8')

decoded_message = DecodeImage(encoded_img)
print(decoded_message[:1000])




skimage.io.imsave("Download_Image_With_Secret_Message.png", encoded_img)    	