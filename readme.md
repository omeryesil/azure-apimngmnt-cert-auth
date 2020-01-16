
Generate certificate 

https://gist.github.com/mtigas/952344

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
