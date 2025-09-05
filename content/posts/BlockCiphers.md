+++
title = "Block Ciphers - Part 1"
date = "2025-08-31"
author = "Tane Haines"
cover = ""
tags = ["", ""]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++

One day a man named Tane was bored in a plane with Wifi. What did he do? He read an academic paper on Block Ciphers.

<!--more-->

So yeah, I was in a plane today going home and was just reading on block ciphers with the intent to learn blockchain hacking. For those not well adversed in hacking, NO I AM NOT GOING TO USE THIS WITH ILL INTENT. Anyway with the disclaimer done, let's get right in.


So I'm going to use this website as an opportunity to explain what I learned while on the plane. I will cover ECB(Electronic Code Book), CBC(Cipher Block Chaining), and a tiny bit of CTR(Counter Mode) which I didn't understand very well.


# The content
## So what's a cipher?
A cipher is a method used to encrypt or decrypt messages.

## So what's a ciphertext?
A ciphertext is the result of a plaintext being passed through a cipher.

## What's a block in block cipher?
It's a 'block' of bits. Say you are encrypting a plaintext message, you would split it into blocks of x size, usually 16 bytes to mass encrypt large amounts of plaintext. If the plaintext is too short to fill in the last block, the plaintext is usually padded to be 0 when modulused by the length.

## What's an encryption key?
It's the key used for encryption. Woah! :0.

## What's a round key? 
It's a key determined from the encryption key used to encrypt the plaintext in 'rounds'.
E.G. AES128 has 10 rounds of encryption meaning each block is encrypted using 10 different round keys

## What's an IV?
This had been a great mystery for me until today. It's the initialization vector. For vectors which need a starting encryption value to xor the plaintext it is used(later discussed in CBC description if I don't forget).

## What's a SPM
A SPM, or Substitution-Permutation Cipher is the fundemental design structure for the ciphers such as aes.

## What's a S-Box
It's a box with multiple operations to encrypt/decrypt the input with a key.

## What is Electronic Codebook, ECB
ECB has a separate encryption process to decryption, but honestly, it's just reversing it. Takes up more space though.

It takes 16 bit cipher blocks and encrypts them by passing them through the S-Box.
The S-Box gets sent the cipher-key, and plaintext(or ciphertext) to decrypt, and the round key is usually generated inside the S-Box. Diagrams are online.

Vulnerabilities: Frequency analysis. AES-ECB is a Symetric Cipher so needs to be Decryptable with the same key it is Encrypted with. Therefore each plaintext always creates the same ciphertext. So for example the block b'apple_apple_appl' would always become the same ciphertext after encryption. If for example this mode is used to encrypt large volumes of real sentences, frequency analysis may be used to find the values of which each block is, or at least which ones are the same as each other. The same method can also be used to determine whether the cipher is AES-ECB mode or not, which can reveal unnecessary information. 

ECB is useful in random values such as keys when transfering keys for RSA over the internet as it is faster pased than CBC.

## What is Cipher Block Chain, CBC

In Cipher Block Chain is a ECB except with extra steps and chaining. Diagrams are online. 
What is happening is that the previous blocks ciphertext is xored with the new plaintext before it is passed into the S-Box. Thus the chaining. Since each block gets xored, you can't frequency analyse it like you can with ECB, each block relys on the output of the previous.

One well known usage of this is in Bitcoin. Each block is a AES-CBC key, literally.

So, what about the first ciphertext value used in the xor. This is what the IV(Initialization Value) is for. It is used as the first ciphertext.

## What is Counter Mode, CTR

I read about this, but at this point I was getting ready to go to sleep so I wasn't too sure. Effectively though, it's space efficient, and you can use it for encryption and decryption unlike the other two modes I mentioned which you need different functions for.

### What it took you 11+ hours to understand this? 
To this I will say no. I slept for 5-6 hours, was helping an old man next to me crumbling under air pressure, and doing my weekend CTF challenges as a side quest. 

Also yes I have learned more, this is just part one because I am writing this at night.

### How do I find the article
For people who want to check the article out [click here!](https://eprint.iacr.org/2020/1545)


