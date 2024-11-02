# Client Certificate test-certificates

## Server

```bash
openssl req \
	-x509 \
	-newkey rsa:4096 \
	-keyout server/server_key.pem \
	-out server/server_cert.pem \
	-nodes \
	-days 365 \
	-subj "/CN=localhost/O=Client\ Certificate\ Demo" \
	-addext "subjectAltName=DNS:localhost,DNS:local.playwright"
```

## Trusted client-certificate (server signed/valid)

```
mkdir -p client/trusted
# generate server-signed (valid) certifcate
openssl req \
	-newkey rsa:4096 \
	-keyout client/trusted/key.pem \
	-out client/trusted/csr.pem \
	-nodes \
	-days 365 \
	-subj "/CN=Alice"

# sign with server_cert.pem
openssl x509 \
	-req \
	-in client/trusted/csr.pem \
	-CA server/server_cert.pem \
	-CAkey server/server_key.pem \
	-out client/trusted/cert.pem \
	-set_serial 01 \
	-days 365
# create pfx
openssl pkcs12 -export -out client/trusted/cert.pfx -inkey client/trusted/key.pem -in client/trusted/cert.pem -passout pass:secure
```

## Self-signed certificate (invalid)

```
mkdir -p client/self-signed
openssl req \
	-newkey rsa:4096 \
	-keyout client/self-signed/key.pem \
	-out client/self-signed/csr.pem \
	-nodes \
	-days 365 \
	-subj "/CN=Bob"

# sign with self-signed/key.pem
openssl x509 \
	-req \
	-in client/self-signed/csr.pem \
	-signkey client/self-signed/key.pem \
	-out client/self-signed/cert.pem \
	-days 365
```

## Private Certificate Authority (CA)

```bash
mkdir -p ca
openssl req \
	-x509 \
	-newkey rsa:4096 \
	-keyout ca/key.pem \
	-out ca/cert.pem \
	-nodes \
	-days 365 \
	-subj "/CN=ca/O=Client\ Certificate\ Demo"
```

## CA-signed certificate (trusted if CA provided)

```
mkdir -p client/ca-signed
# generate (valid) certifcate signed with private certificate authority
openssl req \
	-newkey rsa:4096 \
	-keyout client/ca-signed/key.pem \
	-out client/ca-signed/csr.pem \
	-nodes \
	-days 365 \
	-subj "/CN=Carlos"

# sign with certificate authority
openssl x509 \
	-req \
	-in client/ca-signed/csr.pem \
	-CA ca/cert.pem \
	-CAkey ca/key.pem \
	-out client/ca-signed/cert.pem \
	-set_serial 01 \
	-days 365

# create pfx
openssl pkcs12 \
	-export \
	-out client/ca-signed/cert.pfx \
	-inkey client/ca-signed/key.pem \
	-in client/ca-signed/cert.pem \
	-passout pass:secure
```
