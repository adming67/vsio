### Check OpenSSL version

```shell
openssl version -a
OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)
built on: Wed May 24 17:12:55 2023 UTC
platform: debian-amd64
options:  bn(64,64)
compiler: gcc -fPIC -pthread -m64 -Wa,--noexecstack -Wall -Wa,--noexecstack -g -O2 -ffile-prefix-map=/build/openssl-Z1YLmC/openssl-3.0.2=. -flto=auto -ffat-lto-objects -flto=auto -ffat-lto-objects -fstack-protector-strong -Wformat -Werror=format-security -DOPENSSL_TLS_SECURITY_LEVEL=2 -DOPENSSL_USE_NODELETE -DL_ENDIAN -DOPENSSL_PIC -DOPENSSL_BUILDING_OPENSSL -DNDEBUG -Wdate-time -D_FORTIFY_SOURCE=2
OPENSSLDIR: "/usr/lib/ssl"
ENGINESDIR: "/usr/lib/x86_64-linux-gnu/engines-3"
MODULESDIR: "/usr/lib/x86_64-linux-gnu/ossl-modules"
Seeding source: os-specific
CPUINFO: OPENSSL_ia32cap=0xfeda32035f8bffff:0x9c2fb9
```
##### How fast it runs on the system using four CPU cores and testing RSA algorithm
```
sudo openssl speed -multi 4 rsa
```
### Base64 Encoding/Decoding
##### Encoding a file using base64
```
openssl base64 -in file.data
```
##### Encoding some text using Base64
```shell
echo -n "some text" | openssl base64
```
##### Base64 decode a file with output to another file
```shell
openssl base64 -d -in encoded.data -out decoded.data
```
### Symmetric encryption/decryption [Powerful]
##### Encrypt (-a switch is equivalent to -base64)
```shell
openssl enc -e -a -aes-256-cbc -pbkdf2 -iter xx27014 -kfile pascode -in file.txt -out file.txt.enc
```
##### Decrypt files
```shell
openssl enc -d -a -aes-256-cbc -pbkdf2 -iter xx27014 -kfile pascode -in file.txt.enc -out file.txt
```
###### Babyxxxxesmamaeatingxxxxxnomama@169
### Weaker Encryption
##### Encrypt
```shell
openssl enc cipher-type format -in filename.txt -out filename.txt.enc
	OR
openssl enc -aes-256-cbc -base64 -in filename.txt -out filename.txt.enc
```
##### Decrypt
```shell
openssl enc -d cipher-type format -in filename.txt -out filename.txt.enc
	OR
openssl end -d -aes-256-cbc -base64 -in filename.txt -out filename.txt
```
### Working with Hashes
##### List digest algorithms available
```shell
openssl list -digest-algorithms
```

##### Hash a file using -sha256 (or -sha512)
```shell
openssl dgst -sha256 file.data
```
##### Hash a file using SHA256 with its output in binary form (no output hex encoding)
####### No ASCII or encoded characters will be printed out to the console, just pure bytes. You can append ' | xxd'

```shell
openssl dgst -binary -sha256 file.data
```
##### Hash text using SHA3-512
```shell
echo -n "some text" | openssl dgst -sha3-512
```
##### Create HMAC - SHA384 of a file using a specific key in bytes
```shell
openssl dgst -SHA384 -mac HMAC -macopt hexkey:369bd7d655 file.data
```
##### Create HMAC - SHA512 of some text
```shell
echo -n "some text" | openssl dgst -mac HMAC -macopt hexkey:369bd7d655 -sha512
```
### x509 Certificates (Asymmetric encryption/decryption)
##### If you want to create a self-signed certificate without root ca. Skip -nodes to use a password with the key
```shell
sudo openssl req -x509 -nodes -days 3650 -newkey rsa:4096 -keyout domain.kstar.us.key -out domain.kstar.us.crt
```
##### Remove a passphrase from a private key
```shell
openssl rsa -in privateKey.pem -out newPrivateKey.pem
```
##### Display detailed private key information
```shell
openssl rsa -text -in pub_priv.key -noout
```
##### This command will generate private key and a certificate [rootCA here] in a single shot.
```shell
openssl req -x509 -nodes -sha256 -days 7300 -newkey rsa:4096 -subj "/CN=G/C=PK/ST=Punjab/L=Faisalabad/O=G/OU=I" -keyout rootCA.key -out rootCA.crt
```
##### Generate a Certificate Signing Request (CSR) for an Existing Private Key [if you want to use a config file like below]
```shell
openssl req -new -out CSR.csr -key privateKey.key [-config csr.conf]
```

##### Create CSR File [Include Detail]
```shell
cat > csr.conf <<EOF
[ req ]
default_bits = 4096
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = PK
ST = Punjab
L = Faisalabad
O = G
OU = I
CN = port.us.to

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = port.us.to
#DNS.2 = obsvmgc.us.to
#IP.1 = 172.18.1.7

EOF
```
### Checking Using OpenSSL
##### Check/Verify a Certificate Signing Request (CSR)
```shell
openssl req -text -noout -verify -in CSR.csr
```
##### Check a private key
```shell
openssl rsa -check -in privateKey.key
```
##### Check a certificate
```shell
openssl x509 -text -noout -in certificate.crt
```
##### Check a PKCS#12 file (.pfx or .p12)
```shell
openssl pkcs12 -info -in keyStore.p12
```
### Check Expiration of a Certificate
###### Prints something like 'notAfter=Nov  3 22:23:50 2014 GMT'
```shell
openssl x509 -enddate -noout -in file.pem
```
###### Gives exitcode 0 if not expired withint 86400 seconds (1yr)
```shell
openssl x509 -checkend 86400 -noout -in file.pem
```
### Debugging Using OpenSSL
##### Check an MD5 hash of the public key to ensure that it matches with what is in a CSR or private key
```shell
openssl x509 -noout -modulus -in certificate.crt | openssl md5
openssl rsa -noout -modulus -in privateKey.key | openssl md5
openssl req -noout -modulus -in CSR.csr | openssl md5
```

### Check an SSL connection.
###### All the certificates (including Intermediates) should be displayed
```shell
openssl s_client -connect www.paypal.com:443
```
### Conversion Using OpenSSL
###### Convert a DER file (.crt .cer .der) to PEM
```shell
openssl x509 -inform der -outform pem -in certificate.cer -out certificate.pem
```
###### Convert a PEM file to DER
```shell
openssl x509 -outform der -in certificate.pem -out certificate.der
```
###### Convert a PKCS#12 file (.pfx .p12) containing a private key and certificates to PEM
You can add -nocerts to only output the private key or add -nokeys to only output the certificates.
###### Skip -nodes to incorporate the use of a password
```shell
openssl pkcs12 -in keyStore.pfx -out keyStore.pem -nodes
```
###### Convert PEM TO PKCS#12 (PEM certificate and a private key to PKCS#12 [.pfx .p12])
```shell
openssl pkcs12 -export -inkey privateKey.key -in certificate.crt -certfile CACert.crt -out certificate.pfx
```
###### Convert PEM to PFX
```shell
openssl pkcs12 -export -inkey privateKey.key -in certificate.crt -certfile CACert.crt -out certificate.pfx
```
###### Convert PFX to PEM
```shell
openssl pkcs12 -nodes -in certificate.pfx -out certificate.cer
```
###### Convert PEM to P7B
```shell
openssl crl2pkcs7 -nocrl -certfile certificate.cer -certfile CACert.cert -out certificate.p7b
```
###### Convert P7B to PEM
```shell
openssl pkcs7 -print_certs -in certificate.p7b -out certificate.cer
```
###### Create a CSR from existing certificate and private key:
```shell
openssl x509 -x509toreq -signkey example.key -in cert.pem -out example.csr
```

