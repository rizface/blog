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
PRIVATE KEY EXAMPLE
-----END RSA PRIVATE KEY-----
```

```pem
-----BEGIN RSA PUBLIC KEY-----
PUBLIC KEY EXAMPLE
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