# Certificate Authority Service

This README provides an abstract, generic guide for setting up a Certificate Authority (CA) service and issuing signed certificates using OpenSSL. The steps below can be adapted to various environments.

## Overview

The process involves:
1. Generating a secure CA private key and self-signed certificate.
2. Creating a private key and Certificate Signing Request (CSR) for a server.
3. Configuring certificate extensions in the OpenSSL configuration file.
4. Signing the server's CSR with the CA to produce a certificate.
5. Deploying the signed certificate and corresponding private key to the target application.

## 1. Generate the CA Key and Self-Signed Certificate

Create a secure private key for the CA and generate a self-signed certificate. The certificate is valid for 10 years (3650 days).

```bash
openssl genrsa -aes256 -out ./private/ca.key.pem 4096
chmod 400 ./private/ca.key.pem

openssl req -config ./openssl.cnf -key ./private/ca.key.pem \
  -new -x509 -days 3650 -sha256 -extensions v3_ca \
  -out ./certs/ca.cert.pem
```

## 2. Issue Certificates for a Server

Generate a private key and a Certificate Signing Request (CSR) for the server. Replace `{server_name}` and `{server_ip}` with the appropriate values.

```bash
openssl genrsa -out {server_name}.key.pem 2048
openssl req -new -key {server_name}.key.pem -out {server_name}.csr.pem -subj "/CN={server_name} or {server_ip}"
```

## 3. Configure Certificate Extensions

In your OpenSSL configuration file (`openssl.cnf`), set up the certificate extensions for server certificates. For example, add the following sections to define generic server certificate settings and alternative names:

```ini
[ server_cert_generic ]
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names_generic

[ alt_names_generic ]
DNS.1 = example.com          # Replace with the desired domain
IP.1 = 192.168.1.101         # Replace with the appropriate IP address
```

## 4. Sign the Server Certificate

Use the CA to sign the server’s CSR, producing a certificate valid for 375 days. Adjust the extension name and file paths as needed.

```bash
sudo openssl ca -batch -config ./openssl.cnf -extensions server_cert_generic -days 375 -notext -md sha256 -in ./servers-req/{server_name}.csr.pem -out ./certs/{server_name}.cert.pem
```

Example:

```bash
sudo openssl ca -batch -config ./openssl.cnf -extensions server_cert_generic -days 375 -notext -md sha256 -in ./servers-req/main_server.csr.pem -out ./certs/main_server.cert.pem
```

## 5. Deploy the Certificate and Private Key

Copy the signed certificate and its corresponding private key to the target application’s certificate directory. For example:

```bash
cp ./certs/{server_name}.cert.pem /path/to/app_data/cert/
cp {server_name}.key.pem /path/to/app_data/cert/
```

Replace `/path/to/app_data/cert/` with the actual directory path used by your application.

---

This guide serves as a high-level abstraction for creating and managing a Certificate Authority using OpenSSL. Customize file paths, names, and configuration settings to fit your specific environment.
