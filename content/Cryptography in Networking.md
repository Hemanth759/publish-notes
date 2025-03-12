---
tags:
  - networking
  - cryptography
  - computer-science/cryptography
site: https://dev.to/hemanthvanam/cryptography-in-networking-3o8i
alternateSite: https://w.amazon.com/bin/view/Users/vanamh/SecureCommunicationsOverInternet/
---
# Introduction
## Why data needs to be secure and encrypted?
A lot of critical informations such as Credit card details, your net banking credentials, confidential documents are continuously transmitted over the internet and keeping these data encrypted is crucial for the Internet to work the way it works today. You wouldn't want to propose to your crush over internet if you knew that everybody can see your messages right?

## Types of Encryptions
Widely there are two ways to implement encryptions. 
1. **[[Symmetric key Encryption]]:**  
    Symmetric-key algorithms are algorithms for cryptography that use the same cryptographic keys for both the encryption of plaintext and the decryption of cipher text. Examples include:
    - [Data Encryption Standard](https://www.techslang.com/definition/what-is-symmetric-encryption/#:~:text=1.-,Data%20Encryption%20Standard,-Data%20Encryption%20Standard)
    - [Advanced Encryption Standard](https://www.techslang.com/definition/what-is-symmetric-encryption/#:~:text=2.-,Advanced%20Encryption%20Standard,-One%20of%20the)
    - [International Data Encryption Algorithm](https://www.techslang.com/definition/what-is-symmetric-encryption/#:~:text=3.-,International%20Data%20Encryption%20Algorithm,-The%20International%20Data)
2. **[[Asymmetric key Encryption]]:**  
    Public-key cryptography, or asymmetric cryptography, is the field of cryptographic systems that use pairs of related keys. Each key pair consists of a [[Public key]] and a corresponding [[Private key]]. Key pairs are generated with cryptographic algorithms based on mathematical problems termed one-way functions. Public key is mainly (but not necessarily) be used to encrypt while private key is used to decrypt. Encryption with private key and decryption with public key is also possible. Examples include:
    - [RSA algorithm](https://www.geeksforgeeks.org/rsa-algorithm-cryptography/).
    - [Elliptical curve cryptography](https://www.techtarget.com/searchsecurity/definition/elliptical-curve-cryptography).

----
# Establishing encryption between two parties

> [!INFO]- Why initial encryption establishment is difficult problem
> Imagine you want to communicate with your friend without anyone hearing your conversations. You could talk in private where nobody is present but that is not possible in the internet as the every byte of data transmitting over internet can be interrupted and read by anybody. You could establish a symmetric encryption secret key between your friend and you and share the data after encrypting the message which looks like a garbage text for a person who can't decrypt it. Since your friend has the shared secret key they could decrypt the message using it and send a reply to it after encrypting to you. But how are you going to share this secret key in the first place? 
> **Solutions**:
> 1. Meet your friend in real world and share the key. (It is not feasible to travel every time you want to connect with new person)
> 2. Send the secret key in the post card. (And hope that postman doesn't read take advantage of it?)
> 3. Send the secret key over internet in such a way that anyone listening to your communications can't figure out the actual secret key.

## Secret key exchange - [[Diffie Hellman Algorithm]]
Initial state: 
Alice and Bob want to communicate and establish a secure connection. So they want to create a shared secret key over the internet without anyone else knowing the actual shared secret key. Initially Alice and Bob each have their own secret keys which are private and **won't ever** be shared across the internet. Alice and Bob will agree on values of 'g' and 'n' initially.
![](https://i.imgur.com/6oo7kMm.png)

At the end of this communication Alice and Bob will generate a shared secret key which is created using each of their secret keys but notice that Alice and Bob never shared their actual secret key. If anyone in the public domain wants to generate this key they would only have values 'ga', 'gb', 'g', 'n' and there is no equation to create 'gab' value using these values in maths.

## Man in the middle attack (MITM)
There is a major problem with the Diffie Hellman algorithm as any actor can interrupt the data flow and act as disguise as 'Bob' to the 'Alice' network and disguise as 'Alice' to the 'Bob' network as shown in the following flow.

![](https://i.imgur.com/NOXQife.png)

At the end of this flow Hacker would have established shared secret keys with Alice and Bob. If Alice sends any message to Bob then Hacker can decrypt that message and record it in his/her storage or modify that message and again encrypt it using the shared secret key created with 'Bob'. Alice and Bob could never knew that there is a man in the middle listening to their every conversations by decrypting their messages.

## Using Asymmetric keys to tackle MITM
Asymmetric key encryption could be used to avoid the man in the middle attack problem. But first lets look at the properties of Asymmetric keys:

**Asymmetric key properties:**
1. Any content encrypted by private key can only be decrypted by public key and vice versa.
2. Asymmetric keys can also be used to digitally sign the content.

> [!INFO] Digital Signature
> The message content is first hashed using a standard hashing algorithm like SHA256 and then encrypted using the private key of the server. This means everyone can decrypt the encrypted message content with public key of server which means that the message must be encrypted by the server as only it has the private key. Then the actual message is hashed and compared to decrypted message content to verify that the message has not been tampered by anyone other than the server.

The flow now becomes:
![](https://i.imgur.com/yNXxRVp.png)

> [!IMPORTANT]
> Using the Asymmetric key encryptions we first verify the authenticity of the server (Verify that server says who the server is) and then establish a shared secret symmetric key between server and the client to make the future calls. This is all take care in the initial TLS handshake for any TCP session. Remember that Symmetric keys are only **short lived** and new shared symmetric key has to be created every time a new  connection needs to be created or if old symmetric keys are expired. But the Asymmetric keys **don't change** ever once created so they are super secret.

## Why not use Asymmetric key encryption everywhere?

Technically, we can use Asymmetric encryption everywhere possible. In this flow client and server has their own set of public and private keys. The general flow of HTTPS calls would look something like this:
- In every HTTPS calls the sender has to encrypt the message using public of the receiver and private key of the sender.
- Now the receiver can decrypt the message using the public key of sender which makes sure that message is sent by sender but not anyone else since only sender can encrypt using their private key.
- Receiver can decrypt the message again using his private key. Since only receiver has the his/her private key they can be sure that this message hasn't been decrypted by any one.

This seems like a fine approach. But there are some problems with only using Asymmetric keys for every HTTP call. Some of the major problems:
- Asymmetric key encryption mainly works since finding Prime factors of given number is computationally hard problem. But in the future if quantum computers were able to solve this problem in few secs the whole internet communications becomes a plain textbook as in every TCP call can be recorded, decrypted by anyone who has a quantum machine.
- Encrypting huge payloads using Asymmetric keys is very time consuming process and this will increase the latency.

So it is standardised that Asymmetric key encryptions like RSA would be used to verify the authenticity of at least one of the parties (generally server) and after that create a shared symmetric encryption key which can be used in future http calls. Note that symmetric keys are created per TCP connection every time while Asymmetric keys are only created once and never changed again. (Asymmetric keys can be rotated but any message encrypted using old keys becomes invalid and also there is the problem of Certificate authority needing to update keys at their end)

----
# Certificate Authority
If you notice in the Diffie-hellman with RSA flow MITM attack can still be possible since no one is verifying if the public key of server is actually the correct public key that said server. If that is case then your Internet service provider can redirect the traffic to different server and provide that malicious server's public key as the actual public key of that server. Every one using the same ISP would have no way to verify the authenticity of this Malicious server now.

![](https://i.imgur.com/GTvEd1B.png)

The solution to this problem is the Certificate authorities which verify the authenticity of a public key to its owners. For understanding how Certificate authorities work we need to understand how Digital certificate and Digital signatures work.

## [[Digital Signature]]

![](https://i.imgur.com/VFEBlAd.png)

The message content is first hashed using a standard hashing algorithm like SHA256 and then encrypted using the private key of the server. This means everyone can decrypt the encrypted message content with public key of server which means that the message must be encrypted by the server as only it has the private key. Then the actual message is hashed and compared to decrypted message content to verify that the message has not been tampered by anyone other than the server.

Hashing functions like [SHA256](https://emn178.github.io/online-tools/sha256.html) can be used since encrypting the whole document is time consuming.

**Properties of Hashing algorithms:**
- Hashing any message is always a one way process. Nobody can efficiently get back the original message given the hash of that message.
- Changing the original message even slightly the hash of that message changes drastically.
## [[Digital Certificate]]
![](https://i.imgur.com/cV7rEXj.png)

Digital certificate is a digitally signed document which says that "this person is the owner of this public key and I can vouch for that". And the digital certificate also has the second part of the message which gives details of the public key of certifier of that certificate and digital certificate which vouches for that certifier's public key.

![](https://i.imgur.com/2zmJEIF.png)

----
## Vulnerabilities of current system
### 1. Quantum machines:
The whole Asymmetric encryption keys concepts work only because the prime factorisation of a given number is really hard to find even for computers. But the quantum machines can change that because of the way quantum machines work. Thankfully quantum machines also solve this problem. Quantum machines can also be used to encrypt and decrypt data which would replace the Asymmetric key encryption. Watch this [video](https://www.youtube.com/watch?v=6H_9l9N3IXU) if you want to learn how quantum machines can be used to solve this.

### 2. Injection of unknown root Certificates in devices:
**Lenovo superfish Case study:**

Lenovo installed by default a software called superfish in all their devices as they wanted to replace the ads of popular websites with their own ads essentially earning free advertisements. The major issue was that this superfish application was running on each device creating its own certificates and signing them off. Since this application is supposed to run on each devices private and public keys of this unknown root certifier is present in the superfish application itself. If any developer wanted to use this private key, which they can, and issue certificates for their own servers every lenovo device which has this root certificate will be vulnerable to attacks.

### 3. Root certificates issues wrong certificates:
It could happen that root certificates authorities can issue a certificate for a malicious server and there would be no way to know that this is happening. This is what exactly happened with a Dutch company certificate authority which went bankrupt after this exploit. They certified some malicious servers with their public keys for the domain of google.com. So this malicious servers listened, recorded and decrypted all the HTTPS calls to google.com.

> [!FAIL] Dutch company certificate authority
> People saw that public keys of google.com was being certified by a Dutch company certificate authority and found out this issue is present.

# References
- Symmetric key encryption: [https://www.techslang.com/definition/what-is-symmetric-encryption](https://www.techslang.com/definition/what-is-symmetric-encryption/).
- Asymmetric key encryption: [https://www.techtarget.com/searchsecurity/definition/asymmetric-cryptography](https://www.techtarget.com/searchsecurity/definition/asymmetric-cryptography).
- Diffie Hellman Algorithm: [https://www.youtube.com/watch?v=NmM9HA2MQGI](https://www.youtube.com/watch?v=NmM9HA2MQGI), maths behind it: [https://www.youtube.com/watch?v=Yjrfm_oRO0w](https://www.youtube.com/watch?v=Yjrfm_oRO0w).
- Man in the middle attack: [https://www.imperva.com/learn/application-security/man-in-the-middle-attack-mitm](https://www.imperva.com/learn/application-security/man-in-the-middle-attack-mitm/).
- RSA algorithm: [https://www.youtube.com/watch?v=wXB-V_Keiu8](https://www.youtube.com/watch?v=wXB-V_Keiu8).
- Digital signature: [https://www.docusign.com/how-it-works/electronic-signature/digital-signature/digital-signature-faq](https://www.docusign.com/how-it-works/electronic-signature/digital-signature/digital-signature-faq).
- Digital certificates: [https://www.youtube.com/watch?v=stsWa9A3sOM](https://www.youtube.com/watch?v=stsWa9A3sOM).
- Lenovo superfish malware: [https://slate.com/technology/2015/02/lenovo-superfish-scandal-why-its-one-of-the-worst-consumer-computing-screw-ups-ever.html](https://slate.com/technology/2015/02/lenovo-superfish-scandal-why-its-one-of-the-worst-consumer-computing-screw-ups-ever.html).