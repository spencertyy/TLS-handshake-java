# TLS Handshake Implementation in Java

A from-scratch implementation of the TLS handshake protocol in Java, featuring mutual certificate authentication, Diffie-Hellman key exchange, and AES-encrypted messaging.

![result](TLS%20result.png)

## What This Project Implements

- **Mutual TLS Authentication** — both client and server verify each other's certificates against a shared CA
- **Diffie-Hellman Key Exchange** — 2048-bit MODP Group (RFC 3526) for forward-secure shared secret negotiation
- **AES/CBC Encryption** — symmetric encryption of messages using the derived shared secret
- **HMAC-SHA256** — message integrity verification

## Handshake Flow

```
Client                                Server
  |                                     |
  |--- Client Certificate + DH PubKey ->|
  |                                     | (verify client cert against CA)
  |<-- Server Certificate + DH PubKey --|
  | (verify server cert against CA)     |
  |                                     |
  |    Both compute shared secret via DH key agreement
  |                                     |
  |<--------- Encrypted Messages ------>|
```

## Project Structure

```
src/
├── Client.java           # TLS client entry point
├── Server.java           # TLS server entry point
├── TlsHandshake.java     # Handshake logic (client & server)
├── DiffieHellmanUtil.java# DH key pair generation and shared secret computation
├── MessageCrypto.java    # AES encryption/decryption and HMAC
└── KeyLoader.java        # Certificate and private key loading

resources/
├── CAcertificate.pem             # Self-signed CA root certificate
├── CASignedClientCertificate.pem # CA-signed client certificate
└── CASignedServerCertificate.pem # CA-signed server certificate
```

## Prerequisites

- Java 8 or above
- OpenSSL (to generate your own certificates)

## Setup

### 1. Generate your own certificates

```bash
# Generate CA key and certificate
openssl genrsa -out CAprivateKey.pem 2048
openssl req -new -x509 -key CAprivateKey.pem -out CAcertificate.pem -days 365

# Generate server key and certificate signed by CA
openssl genrsa -out serverPrivate.key 2048
openssl req -new -key serverPrivate.key -out server.csr
openssl x509 -req -in server.csr -CA CAcertificate.pem -CAkey CAprivateKey.pem -out CASignedServerCertificate.pem -days 365

# Generate client key and certificate signed by CA
openssl genrsa -out clientPrivate.key 2048
openssl req -new -key clientPrivate.key -out client.csr
openssl x509 -req -in client.csr -CA CAcertificate.pem -CAkey CAprivateKey.pem -out CASignedClientCertificate.pem -days 365

# Convert private keys to DER format
openssl pkcs8 -topk8 -inform PEM -outform DER -in serverPrivate.key -out serverPrivateKey.der -nocrypt
openssl pkcs8 -topk8 -inform PEM -outform DER -in clientPrivate.key -out clientPrivateKey.der -nocrypt
```

### 2. Update certificate paths

In `Client.java` and `Server.java`, update the resource paths to point to your local `resources/` directory.

### 3. Compile and run

```bash
# Compile
javac -d bin src/*.java

# Start server (terminal 1)
java -cp bin Server

# Start client (terminal 2)
java -cp bin Client
```

## Key Design Decisions

| Component | Choice | Reason |
|---|---|---|
| Key Exchange | Diffie-Hellman 2048-bit | Forward secrecy; RFC 3526 standard parameters |
| Encryption | AES/CBC/PKCS5Padding | Symmetric encryption after shared secret derived |
| Integrity | HMAC-SHA256 | Message authentication code |
| Auth | X.509 certificates signed by custom CA | Mirrors real-world PKI infrastructure |
