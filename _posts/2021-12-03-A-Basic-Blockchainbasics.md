---
layout: post
title: "Blockchain prerequisites (PART 1)"
subtitle: "Glimpsing the world of blockchain"
date: 2021-12-03
author: "Hitesh Kumar"
header-img: "img/bs1-bg.jpg"
tags: [golang, blockchain, publickeycryptography, RSA, scratch]
---


# Blockchain prerequisites (PART 1) :

## Public Key Cryptography :

If you stumbled on the term for the first time. You have my sympathy ~ Lol no. Kidding. So without anymore pain lets get started. In this blog, we will discuss about 
-  Message encryption dilemma
- What is public key Cryptography
- What's the use and why
- Code in golang 


## Message encryption dilemma ~ 

As our world is progressing more and more toward digitalization. The way of living and operating things is also changing. We are paying more online, ordering online, watching movie online, sending message online AND WHAT NOT !

One of the events that catch more headlines nowdays is message encryption. The reason why people started using more signal instead of whatsapp. ~ Although whatsapp claims they have end to end encryption... does it matter ? the way people learn and act matters at the end. Anyways, unencrypted connection will easily allow any third party to intercept and manipulate message. But the thing is, how does the encryption works ? If one day i have to sit and think, create my own message encryption method. The first thing i will be thinking of is :
-  If i have to send message to other person. I can obfuscate message using some method and make it unreadable for any attacker in between and the person who will recieve my message can unobfuscate it. Suppose, i obfuscate it with a key, means it can only be unobfuscated at the reciever's end with that particular only. Problem here is, key needs to be known by both party. And that raises little security risk. Because, how do you think you are going to let other person send or know about the key. That indeed adds some security risk. 


So what would be better way. IT IS LITERALLY THE TITLE. 

##  What is this, actually ?

According to wikipedia :

```
 Public-key cryptography, or asymmetric cryptography, is a
 cryptographic system that uses pairs of keys. Each pair
 consists of a public key (which may be known to others) 
 and a private key (which may not be known by anyone except
 the owner).
```

So basically, two keys  
- Public Key  -- out there in open domain
- Private Key  -- private to an individual only

The Dataflow is as follows :

[![Screenshot-2021-12-12-at-14-01-20-FC1sae-Nr-Keli-Sketchboard.png](https://i.postimg.cc/J4ZMjrPN/Screenshot-2021-12-12-at-14-01-20-FC1sae-Nr-Keli-Sketchboard.png)](https://postimg.cc/Czh3ggy5)


- Whatever encrypted with public key can only be decrypted by private key

So, if i am using public key of my friend to send some message. My friend can only decrypt it using his private key. Even anyone attacks in middle and tries to decrypt message. They wont be succesfull because :
- Private key is unique to an individual.
- encrypted message is highly obfuscated. They cannot brute force it


This method is highly popular in blockchain and digital signatures and gives promising results.


### Coding up :>

RSA (Rivest–Shamir–Adleman) encryption is one of the most widely used algorithms for secure data encryption.

Let's generate private and public key of Sender.

[![carbon-1.png](https://i.postimg.cc/zBw1GPQY/carbon-1.png)](https://postimg.cc/Ppq76KhS)

Now for reciever's :

[![carbon-2.png](https://i.postimg.cc/L6p3PQ00/carbon-2.png)](https://postimg.cc/G9XDwKRk)


Now, lets bind it and encrypt some sample message. You will see that we are going to use recvPublicKey. 

[![carbon-3.png](https://i.postimg.cc/RZ18fnRw/carbon-3.png)](https://postimg.cc/mPD8WkKr)

Output is gonna be like this :

[![carbon-4.png](https://i.postimg.cc/wvbTprRx/carbon-4.png)](https://postimg.cc/PpYkb2M0)

Now, we are pretty sure that our message is converted into some non-understandable terms. And it is ready to be sent over the internet without any concern of man in the middle attack.

But for reciever. They need to be able to understood what is that. For that our decrypting code would look something like this :

[![carbon-5.png](https://i.postimg.cc/T24q7YwF/carbon-5.png)](https://postimg.cc/pyKFyvKJ)


[![carbon-6.png](https://i.postimg.cc/CKcfRmnX/carbon-6.png)](https://postimg.cc/TyWPSj9J)

Pretty magical right ?!

Now i wanna just try something new. What if i do some changes at `ciphertext` (The text we get after encryption). Let's see what we get.
Doing simple :
We manipulated the message. Now when receiver gets it and tries decrypting. Let's see how it goes.

```go
	hackermessage := []byte("Neo, you are hacked!")
    // Decrypt Message at RECV. end_ -> Using private key
	plainText, err := rsa.DecryptOAEP(hash, rand.Reader, recvPrivateKey, hackermessage, label)

	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
	
```


We get output like :
```
crypto/rsa: decryption error
exit status 1

```

We can see, anything we send after it gets encrypted. Doesn't matter what we do. The person who holds the private key gets assured that they will get what they seek.



#### Full Code :

[![carbon.png](https://i.postimg.cc/2y5SWDhf/carbon.png)](https://postimg.cc/HcfmGF6S)


```go
package main

import (
	"crypto/rand"
	"crypto/rsa"
	"crypto/sha256"
	"fmt"
	"os"
)

func main() {

	// Generate RSA Keys for Sender !!

	// ~~~~~~~~~~~~~ SENDER KEY : ~~~~~~~~~~~~~~~

	// Private key
	senderPrivateKey, err := rsa.GenerateKey(rand.Reader, 2048)

	if err != nil {
		fmt.Println(err.Error)
		os.Exit(1)
	}
	// Public key
	senderPublicKey := &senderPrivateKey.PublicKey

	// ~~~~~~~~~~~~~ RECIEVER's KEY : ~~~~~~~~~~~~~~~

	recvPrivateKey, err := rsa.GenerateKey(rand.Reader, 2048)

	if err != nil {
		fmt.Println(err.Error)
		os.Exit(1)
	}

	recvPublicKey := &recvPrivateKey.PublicKey

	fmt.Println("Private Key : ", senderPrivateKey)
	fmt.Println("Public key ", senderPublicKey)
	fmt.Println("Private Key : ", recvPrivateKey)
	fmt.Println("Public key ", recvPublicKey)

	// ~~~~~~~~~~~~~ Encryption : ~~~~~~~~~~~~~~~

	//Encrypt Sender's Message
	message := []byte(`Neo, sooner or later you're going to realize just as i did. 
	There's a difference between knowing the path and walking the path`)
	label := []byte("")
	hash := sha256.New()

	// Encrypting with recvPublicKey !
	ciphertext, err := rsa.EncryptOAEP(hash, rand.Reader, recvPublicKey, message, label)

	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	// Let's see how the encrypted message looks like :
	fmt.Printf("OAEP encrypted [%s] to \n[%x]\n", string(message), ciphertext)
	fmt.Println()

	// Decrypt Message at RECV. end_ -> Using private key
	plainText, err := rsa.DecryptOAEP(hash, rand.Reader, recvPrivateKey, ciphertext, label)

	if err != nil {
		fmt.Println(err)
		os.Exit(1)
	}

	fmt.Printf("OAEP decrypted [%x] to \n[%s]\n", ciphertext, plainText)

}
```


## Conclusion :

This is awesome stuff. Try some out by yourself. Try tampering with private and public key and see whats the result. Play around.

I hope it was some useful to you. and my future self.
