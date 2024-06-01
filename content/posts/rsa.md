---
title: "RSA"
date: 2024-06-01T14:15:17+07:00
draft: false
toc: true
images:
tags:
  - encryption
  - rsa
  - golang
---

RSA (Rivest-Shamir-Adlema) is asymmetric encryption algorithm it works by encrypt plain message using public key, and decrypt using private key. public key can be shared amongs parties/services/client but the private key must be kept for server/message processor because encrypted message only can be decrypted by right pair private key, decryption will be failed if you try to decrypt message using wrong private key.

![RSA](../images/rsa.png)

Below is example of private and public key
```pem
-----BEGIN RSA PRIVATE KEY-----
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
-----END RSA PRIVATE KEY-----
```

```pem
-----BEGIN RSA PUBLIC KEY-----
MIICCgKCAgEA09r9ouQfbGtKQ8izKcADVHoKs7e3wL0qpXazidQT/qF6kgdU/GIF
+beFVn4ZoVurxt7gow7BVLulyepUS/r6XmFXViBu44WKdcOL9wAtZbTNHmLbo9SW
y6zbsbfuJJPdA3KhTeE0HgagyTcU/icNLjqTZEDmE6B7huk/4u+TzU2AzLxvh6yW
fTj1RDeyc6AOy8hZxB6QAOW5Vr+7VahYUzIYgFZ/vmCS9Kp7B4OblVX7O+nxEelf
7s4NZOftwAqdLjgg4rOyVZp6m0C03IqtYAJ5L5vhEsa8GeQscJ1+Q6lQ0W6MGZ/8
QG2hZmFtHpytq2wlUovYqs3u62VtMRqQRVxg2XuWSfV/wh4FZmH/y0EiD2PJ0GyI
V9eGc1wsKl3fyYxTayC7CU8sfh6DD9V7dzwmXdo/ZRmWmjrjfZg8dzo9rvtegwUd
83IqZcGaSrEjOrNyZ1ai+KRTzbT2QMQiDSJTYZjQ4ht8u93tTt3dtbDyVVUHFs9o
qoA0J9Fuj4/Or75sP39y64Y+U9FozKfBKPdkVcvHwHuI3qhIgv2Q1/zBepLEYpLd
yuinusfFE7mEYVzkWqDMhaCpCk87HiHZJm4XbeaaFX4wTQYIZDgP38qFO43ttqio
j0dhDSDZrBbCTErq8cfhe2X6jsNJdd/BKjiZdM8cg0MUriohpLmhlysCAwEAAQ==
-----END RSA PUBLIC KEY-----
```

RSA can be used in any programming language, below is example using Go programming language

```go
package main

import (
	"crypto/rand"
	"crypto/rsa"
	"crypto/sha256"
	"crypto/x509"
	"encoding/pem"
	"fmt"
	"log"
	"os"
)

func generatePrivateKey() (*rsa.PrivateKey, error) {
	pk, err := rsa.GenerateKey(rand.Reader, 4096)
	if err != nil {
		log.Fatalf("err generate pk: %w", err)
	}

	privateKeyPem := pem.EncodeToMemory(&pem.Block{
		Type:  "RSA PRIVATE KEY",
		Bytes: x509.MarshalPKCS1PrivateKey(pk),
	})

	err = os.WriteFile("private.pem", privateKeyPem, 0644)
	if err != nil {
		return nil, err
	}

	return pk, nil
}

func generatePublicKey(priv *rsa.PrivateKey) (*rsa.PublicKey, error) {
	publicKeyPem := pem.EncodeToMemory(&pem.Block{
		Type:  "RSA PUBLIC KEY",
		Bytes: x509.MarshalPKCS1PublicKey(&priv.PublicKey),
	})

	err := os.WriteFile("public.pem", publicKeyPem, 0644)
	if err != nil {
		return nil, err
	}

	return &priv.PublicKey, nil
}

func main() {
	priv, err := generatePrivateKey()
	if err != nil {
		log.Fatalf("Err failed generate private key: %w", err)
	}

	pub, err := generatePublicKey(priv)
	if err != nil {
		log.Fatal("Err failed generate public key: %w", err)
	}

	msg := []byte("plain message")
	hash := sha256.New()
	rands := rand.Reader

	encMsg, err := rsa.EncryptOAEP(hash, rands, pub, msg, nil)
	if err != nil {
		log.Fatalf("Failed encrypt message: %w", err)
	}

	fmt.Printf("Enc message: %x \n", encMsg)

	msg, err = rsa.DecryptOAEP(hash, rands, priv, encMsg, nil)
	if err != nil {
		log.Fatalf("Failed descrypt message: %s", err.Error())
	}

	fmt.Printf("Dec message: %s \n", msg)
}
```