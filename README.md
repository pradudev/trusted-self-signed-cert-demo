# Sample code to demo trusted self-signed cert in IIS

Self-signed certs are not trusted by the browsers because the browser cannot validate its authenticaity as it is not signed by any valid CAs

## Pre-requisites
1. OpenSSL

## Steps
### Generate self-signed Root CA Cert
1. Generate a private key "root-prado-ca.key"
```bash
openssl genrsa -des3 -out root-prado-ca.key 2048
```

2. Generate a public cert "root-prado-ca.pem" using the above private key
```bash
openssl req -x509 -new -nodes -key root-prado-ca.key -sha256 -days 1460 -out root-prado-ca.pem
```

3. (Optional) Generate a "root-prado-ca.pfx" file from the above key and cert file
```bash
openssl pkcs12 -export -out root-prado-ca.pfx -inkey root-prado-ca.key -in root-prado-ca.pem
```

4. Install the above root cert in the "Trusted Root Certification Authorities"
```bash
certutil -addstore -f "ROOT" root-prado-ca.pem
```

### Generate a custom cert signed by a self-signed Root CA for the domain "www.prado.com"
1. Create the file "www-prado-com.csr.cnf" and add the below content in it
```bash
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn

[dn]
C=AU
ST=VIC
L=MEL
O=Prado Pty Ltd
OU=Marketing
emailAddress=admin@techforum.com
CN = www.prado.com
```

2. Create the file "www-prado-com.v3.ext" and add the below content in it
```bash
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1=prado.com
DNS.2=www.prado.com
```

3. Generate the private key "www-prado-com.key" and the Certificate Signing Request(CSR) file "www-prado-com.crt"
```bash
openssl req -new -sha256 -nodes -out www-prado-com.csr -newkey rsa:2048 -keyout www-prado-com.key -config www-prado-com.csr.cnf
```

3. Generate the cert "www-prado-com.crt" signed by our root CA
```bash
openssl x509 -req -in www-prado-com.csr -CA root-prado-ca.pem -CAkey root-prado-ca.key -CAcreateserial -out www-prado-com.crt -days 500 -sha256 -extfile www-prado-com.v3.ext
```

4. (Optional) Generate the "www-prado-com.pfx"
```bash
openssl pkcs12 -export -out www-prado-com.pfx -inkey www-prado-com.key -in www-prado-com.crt
```

