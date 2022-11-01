# Asymmtetric Cryptography ECDSA

I apologize in advance for the opprobrious content of the README file, but I wanted to describe each step in reasonable detail showing my deep understanding of the subject.

## FACILITIES

### [1.	Generating a pseudorandom number](https://github.com/maciej-mlynski/AsymetricCryptography/tree/main/entropy)

At this stage, at least several random numbers are generated by various means. At first, an n-bit string is drawn. The user can choose between a 128-bit and a 256-bit entropy. The length of the entropy also translates into the length of the seed phrase, but more on that later.

Then, using the Xoroshiro128 encryption method, another number is drawn. This number is the mixing instruction of the first entropy. This instruction is very similar to the method of generating a public key in the DH (Diffie Helman) system, where the bits (0-1) decide whether to perform one or 2 transformations. In my case, the number generated by Xoroshiro128 will determine the mixing of the entropy generated earlier. If the bit of the number Xoroshiro128 will be 0 then we do not mix, if 1 then we mix. There may be several bits, so mixing may occur even several times. By mentioning mixing I generally mean the shufle method.

On the issue of entropy generation, which is crucial for cryptographic asymmetry, I still have a bit to work out. Even though I would mix and blend the number generated by the algorithm, it will still only be a PSEUDO random number, which, if enough numbers are generated, can be predicted in some way. Improving this step will be an attempt to create a real random number. Usually used for this is the tracking of the mouse, which the user is asked to move. However, I do not want to go into traditional solutions, so I think that in the future I would like users to tell a little about themselves, maybe choose some colors, favorite premiods, sports etc.

### [2.	Hashing method](https://github.com/maciej-mlynski/AsymetricCryptography/tree/main/hash_method)

The hashing method is used in many important places in my application. The hash will primarily be the private key and any other entropy. I have chosen the SHA 256 hash method. Any string passed through this algorithm will create a unique 256-bit hash. Is this enough? For the sake of comparison, let me add that in terms of security, the 256-bit key in the ECDSA model is comparable to the 3072-bit key for the RSA model. Below is an interesting comparison:

 Symetrci Key Size (bits) | RSA and DH Key Size (bits) | Eliptic Curve Key Size (bits) | 
| ------------------------|----------------------------|-------------------------------|
| 80                      |	1024                     	 | 160                           |
| 112         	           | 2048                       | 224                         	 |
| 128                   	 | 3072                       | 256	                          |
| 192                     | 7680                       | 384                           |
| 256                     | 15360                      | 521                           |


### [3. Seed Phrase](https://github.com/maciej-mlynski/AsymetricCryptography/blob/main/key_generators/seedPhrase.py)

The seed phrase is nothing more than a representation of the generated entropy in a user-friendly way. Let's start with why do we need it at all? The seed phrase in my project finds its use in private key recovery. This is exactly the same as it is in metamask wallets. What is worth adding at this point is that, in fact, the seed phrase will help to find the entropy even before the mixing process passes (SHA - 256). So the phrase is generated between the entropy process and the hashing method. how does it work?

1) Entropy is represented in binary form. (128 bits or 256 bits)
2) Looking for a checksum. For a 128 bit key it will be the first 4 bits, for 256 bits -> 8 bits. *The bits mentioned in the case of the checksum are strictly for the final version of the private key. (after sha-256 hashing method)
3) We add the checksum to the entropy from pt.1 represented in bits.
4) We group the bits so that there are 11 bits in each group.
5) By substituting the individual into the appropriate equation, we are able to obtain specific numbers from 1 to 2048.
6) Each number has its representation in the form of a word, which is mapped with words in the BIP-39 standard.
7) The user receives a seed phrase of a length that it has predetermined. 128 bit entropy presents 12 words, and 256 bit entropy presents as many as 24 words.
8) The words should be written by the user on a piece of paper.

### [4.	Secret and Public Keys](https://github.com/maciej-mlynski/AsymetricCryptography/tree/main/key_generators)

This is where all the magic of ECDSA (Elliptic Curve Digital Signature Algorithm) happens. Let me add that I use a particular epiliptic curve called secp256k1. Modules for ECDSA you can find [here](https://github.com/maciej-mlynski/AsymetricCryptography/tree/main/ecdsa)

The private key is nothing more than the entropy represented as a hash in the SHA-256 model. The public key is created from the private key. It is, in a sense, an instruction for its creation. In the case of ECDSA, we make use of the epiliptic curve, on which the certain transformations will be performed. The starting point is G, which describes the shape of the epiliptic curve. The number is then transformed based on the private key.

As known, the key in the form of hex, or int we can also represent in bit form. The bits of the key will determine whether our G-point will be doubled, or added and doubled. Of course, this is not simply doubling or adding, but scalar operations. We start from the right side of the private key’s bit.

If the first bit from the right is 0 then point G is doubled. If the next bit equals 1 then at this point we also perform the doubling method on the previously modified G point. Only when the next 1 occure in the bit representation of private key then we perform the scalar addition / point addition method.

After that, we again perform the scalar doubling method. We repeat the process until the last 1 from the left. As a result of scalar multiplications in the result we get the coordinates of the public key from which we easily get the public key.

What advantages does it have? The process is one-sided, that is, it is easy to generate a public key from a private key, but reversing the process requires a huge amount of computing power.  Of course, the longer the key, the harder the task for hackers. The principle is simple: longer key = more multiplications = harder to break.

### [5.	Signing transaction](https://github.com/maciej-mlynski/AsymetricCryptography/blob/main/message/signEncMessage.py)

Here, too, we use the power of the epiliptic curve and pseudorandom numbers. Let's take a look at this process one by one:
1) Generate a random entropy. It is important that the number be different from each other each time. (NONCE - Number Used Only Once)
2) The entropy is passed through the SHA-256 hashing method and then used in the ECDSA model as a G-point transformation instruction. (Just like with a private key).
3) From the generated point, we only take the X coordinate.
4) Generate another entropy and also hash it using the sha-256 method.
5) Generate the cryptographic signature by substituting the variables into the appropriate formula:
 - first mixed entropy
 - second mixed entropy
 - private key of the person sending the message
 - coordinate x
 - number n, which is the PRIME number

The transaction signature method returns 3 values: Signature, second hashed entropy and coordinates of the point created by the first mixed entropy. These values can be public as long as the user is willing to reveal his real face and the receiver does not want to verify the sender. These values must not lead third parties to the private key.

* The signature of the transaction is an add-on in my project, but when it comes to transactions based on blockchain technologies, it is a fundamantal issue, as it is a proof of transparencies.

### [6.	Verification of transactions](https://github.com/maciej-mlynski/AsymetricCryptography/blob/main/message/verifyDecMessage.py)

Here the recipient of the transaction can check whether the sender is who he claims to be. To check, you will need all the result values of the transaction signing method and the the public key of the person sending the message. It is these elements substituted into the subsequent ECDSA multiplications and formulas that will lead us to the generation of the next point.

If the public key of the person sending the message is as to be correct then the coordinates of the point as a result of the verification process should be identical to the point sent by the sender.

### [7.	Encryption and decryption of messages](https://github.com/maciej-mlynski/AsymetricCryptography/tree/main/message)

To encrypt the text, I used the AES library. It allows the content to be encrypted against a certain key. This key, of course, must not be known to third parties.


The same key is used to encrypt and decrypt the message, which proves its symmetry. What is worth noting, however, is that it is never sent between users and is different every time. What users send between each other is, of course, the encrypted message and instructions on how to find this key based on the recipient's private key.

Briefly about the process of sending instructions to find the key that decrypts the message:
1) As before, a random entropy mixed by SHA-256 is generated.Let's call this number K.
2) Exactly as before K in bitwise form is the instruction to transform point G. Let's call it the R point.
3) Create the secret value by scalar multiplication of the number K by the coordinates of the recipient's public key (Not by the point G as it was in each previous case).
4) We send the encrypted message together with the point R.

The message along with the R-point does not need to be specially protected, because only the recipient has a private key, so that by doing some transformations on the R-point he can decrypt the message. The method to get to the secret key is, as before, scalar multiplication. The difference is that the instruction is, as I mentioned before, its private key, and the the object to be transformed is not the point G, but the point R. The secret key is the X coordinate of the generated point.

* At this point I would like to share one of the weaknesses of my algorithm. Just as the generation of keys and the signing and verification of transactions is relatively transparent and thus  hard to break by hackers, the encryption of messages is not quite so. Admittedly, the AES method is considered quite strong security, but in any case I did not do it myself. I took advantage of the available library. So I have to trust that the developers did not leave any wicket for themselves or hackers. This is definitely a part I would like to do my own way in the near future.

### [8.	Private key recovery](https://github.com/maciej-mlynski/AsymetricCryptography/blob/main/key_generators/recoverKeys.py)

As I mentioned earlier, the seed phrase is a reflection of the entropy from which the private key is formed. Thus, by making some transformations we can enable users to recall the secret key. Since the private key is needed by the user in several places, the user can use the seed phrase directly instead of the secret key.

### 9.	Improvement plans
1) Writing my own script to encrypt messages
2) Connecting to a database, where the hashes of transactions (nonce) will be stored along with the public key so that there is no repetition (The chance of this is like 1/2**256, however, still exists).
3) Writing a script to create a real random number, not a pseudo random number as is currently the case.

### 10. A puzzle for hackers

Below I leave the details of a certain transaction. User A sends a message to user B. As I mentioned some information can be public, and anyway no one should break the cipher.

The task for those willing to do so is to decrypt the sent message, without knowing the private key of the users, or the escrypt key decrypting the content of the message. If you succeed it only means that my code is weak and I still need to work on it. If, in addition, you are able to guess the private key of one or both of the users it will be a clear signal to me that I probably shouldn't take up such complicated topics as asymmetric cryptography. Good luck

#### DATA

Sender public key: 02c4e16c2f817821ba34e51a9590e9f92222ec078a53eb217794d13e657bcd5080

Receiver public key: 03848a642cfd57b7e6d37df7fc4a0e67dfbbc61d027acfdd2a54dfe527ce520696

R point coordinates: 
x = 99580057917327851106966939903015676517289578513513259442518168252597044173260, 
y = 9372175617998258043462139118118171645045943673997511838361674329883109577939

Encrypted message:
(b"\x7f*\x9fp\xc2\x12\x9c@\x10\x07GI\xc2=\x12\xb3\x13V\xca\xb0\xc3(<c\x9f\x8ag\xb2$\xe4\xadk\xbd$\x16\x84\xc3\xfa\xf4\xa9\x9b]E\xd7\xde\x95q[p%\x8a\x8b\x9e\xd1\x1b\xacv\xdb\x84\x7f\x7f\xda\xa6\xe9\x85\\7t\xb2\x931L\xb5\xef\x14\xa9\xe7\xed\x8e\xab\x8e\x8fv\xfb\xa3\xbaX\xd4\xf0?w\xcb:\xcb\xf3\xe8H0\xdfbN\xa2\x84\x04A,\x00\xe4d\xc1b\xfa^\xe4\x1aQL\x8b\x89\x8d\x0eJ\xf0qZ\xb1]\xea\xe6\xdbK\xf1\x0b\xf6%z\xb9\x03z\xbb\xa2\xd7\xa5\xd7k\xbfju\xa7`\x0f\xc7\xecg%R\xc8\xae\xfcXc\n\x19\xbd9y\x88\xd0\xdb\xb2\xc5\xa4e\xad[^I\xfc\xf5\x1c\xc7gh\xb4\x93\xbc\xbbC\xe3\xc4\xd6\xe8 W\x1d\xe5\xea\x86\xa7\xe3\x81\xaa\x01@\xc1\x0b\xce\xcc\xbb2N\xb6z\xae\xa5\xe6\x895\xed\x80\x9a\xbd\xb2n\xd5\x94V!P\xfb\xac\xe9B\xfa'F\xbd\x14\xf1M\x7f\x0e_Y,s\xae\x8dx\x88r\x1f.\x18\xfb\xe3\xfa\xc99\x06\x88\xd5jc.\xd8\xd2\xa4\x99>\xb3\xdf\x90\x18\x825\xf0\xe4\xec![\xa0\xc1A\x0c\x9b\xd1\xfbT\xad\xc7,\tE\xee\x8fH\xcd\x85", b'\x05\xf9\xf2C\xdf(\xe0.B\xd3\t\xfa\x92w\xb9\x8b', b'\xc2\xbadb\xe4/\x80o\xed\xac\x0e\xe4\x8d\xba\x01\x9b')




##### In addition, you can verify for yourself whether the sender is who he claims to be based on these values:

Signature:
x = 61804390252209550729716183329621277690057643953601791546373572698090826822114,
y = 25199922369180434650770836400268117027362420901334425256642247210555879960953

Encrypted entropy (k2): 86978483577432511642995949475920788319515783826597287028628526562044294013655

K point coordinates:

x = 61804390252209550729716183329621277690057643953601791546373572698090826822114,
y = 108410082898792334435153827026831082586755225925849725110772806954982724812810

*If the point K, will be equal to the generated point formed from the signature, entropy k2 and the public key of the sender, it will mean that the public key provided is correct.


