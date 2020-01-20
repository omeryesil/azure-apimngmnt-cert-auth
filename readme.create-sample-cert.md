
Generate certificate 
https://gist.github.com/mtigas/952344

- **CER files:** CER file is used to store X.509 certificate. Normally used for SSL certification to verify and identify web servers security. The file contains information about certificate owner and public key. A CER file can be in binary (ASN.1 DER) or encoded with Base-64 with header and footer included (PEM), Windows will recognize either of these layout.

- **PVK files:** Stands for Private Key. Windows uses PVK files to store private keys for code signing in various Microsoft products. PVK is proprietary format.

- **PFX files:** Personal Exchange Format, is a PKCS12 file. This contains a variety of cryptographic information, such as certificates, root authority certificates, certificate chains and private keys. It’s cryptographically protected with passwords to keep private keys private and preserve the integrity of the root certificates. The PFX file is also used in various Microsoft products, such as IIS.

## Step 1 - Create Certificate Authority Root

```shell
CERT_PASS=abcd1234
```

- Generate ca.pass.key and ca.key files

  ```shell
  openssl genrsa -aes256 -passout pass:${CERT_PASS} -out ca.pass.key 4096
  openssl rsa -passin pass:${CERT_PASS} -in ca.pass.key -out ca.key
  rm ca.pass.key
  ```

- Generate the CA root cert
  when prompted, use whatever you'd like, but i'd recommend some human-readable Organization and Common Name.

  ```shell
  openssl req -new -x509 -days 3650 -key ca.key -out ca.pem
  ```

- Create Cer file from Pem (Azure API Management requries Cer)

  ```shell
  openssl x509 -inform PEM -in ca.pem -outform DER -out ca.cer
  ```


## Create Client Key and CSR

```shell
CLIENT_ID="01-mydevice"
CLIENT_SERIAL=01

openssl genrsa -aes256 -passout pass:${CERT_PASS} -out ${CLIENT_ID}.pass.key 4096
openssl rsa -passin pass:${CERT_PASS} -in ${CLIENT_ID}.pass.key -out ${CLIENT_ID}.key 
rm ${CLIENT_ID}.pass.key

# generate the CSR
# i think the Common Name is the only important thing here. think of it like
# a display name or login.
openssl req -new -key ${CLIENT_ID}.key -out ${CLIENT_ID}.csr

# issue this certificate, signed by the CA root we made in the previous section
openssl x509 -req -days 3650 -in ${CLIENT_ID}.csr -CA ca.pem -CAkey ca.key -set_serial ${CLIENT_SERIAL} -out ${CLIENT_ID}.pem
```

## Bundle the private key & cert for end-user client use

```shell
cat ${CLIENT_ID}.key ${CLIENT_ID}.pem ca.pem > ${CLIENT_ID}.full.pem
```

## Bundle Client Key into PFX File

```shell
openssl pkcs12 -export -out ${CLIENT_ID}.full.pfx -inkey ${CLIENT_ID}.key -in ${CLIENT_ID}.pem -certfile ca.pem
```
