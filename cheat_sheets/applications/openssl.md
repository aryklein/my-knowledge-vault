# OpenSSL basic commands

Here is a basic cheat sheet for OpenSSL commands:

### Generating a new private key and Certificate Signing Request (CSR)

```bash
openssl req -out CSR.csr -new -newkey rsa:2048 -nodes -keyout privateKey.key
```

### Generating a self-signed certificate

```bash
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout privateKey.key -out certificate.crt
```

### Generating a certificate signing request (CSR) for an existing private key

```bash
openssl req -out CSR.csr -key privateKey.key -new
```

### Checking a Certificate Signing Request (CSR)

```bash
openssl req -text -noout -verify -in CSR.csr
```

### Checking a private key

```bash
openssl rsa -in privateKey.key -check
```

### Checking a certificate

```bash
openssl x509 -in certificate.crt -text -noout
```

### Determine if a private key matches a public key

You can determine if a private key matches a public key (or a certificate) by comparing their modulus. The modulus is a part of the key and if the moduli match, the keys match. Here's how you can do that:

```bash
# extract the modulus from the private key
openssl rsa -noout -modulus -in privateKey.key | openssl md5
# extract the modulus from the public key
openssl rsa -pubin -noout -modulus -in publicKey.key | openssl md5
# or from a certificate
openssl x509 -noout -modulus -in certificate.crt | openssl md5
```

In all these commands, `openssl md5` is used to create a hash of the output, so the moduli are easier to compare. If the `md5` hashes match, it means the keys match.