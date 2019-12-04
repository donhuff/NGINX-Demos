
# NGINX webserver that serves a simple page containing its hostname, IP address and port as wells as the request URI and the local time of the webserver.

The images are uploaded to Docker Hub -- https://hub.docker.com/r/nginxdemos/hello/.

How to run:
```
$ docker run -P -d nginxdemos/hello
```

Now, assuming we found out the IP address and the port that mapped to port 80 on the container, in a browser we can make a request to the webserver and get the page below: ![hello](hello.png)

A plain text version of the image is available as `nginxdemos/hello:plain-text`. This version returns the same information in the plain text format:
```
$ curl <ip>:<port>
Server address: 172.17.0.2:80
Server name: 22becba5323d
Date: 07/Feb/2018:16:05:05 +0000
URI: /
Request ID: 48ba0db334a6ed165e783469c2af868f
```

The images were created to be used as simple backends for various load balancing demos.

## TLS Certificate

Adapted from [this post](https://www.markbrilman.nl/2011/08/howto-convert-a-pfx-to-a-seperate-key-crt-file/).

1. In PowerShell create server certificate: `New-SelfSignedCertificate -DnsName "argus.15x10.com" -CertStoreLocation "cert:\CurrentUser\My"`
2. Open MMC.exe. Add the certificates snap-in for the current user. Navigate to Personal\Certificates. Export the generated certificate with private key to server.pfx with password.
3. Open Command Prompt. Change directory to the folder with server.pfx. Start and OpenSSL session.
4. Extract the encrypted private key: `pkcs12 -in server.pfx -nocerts -out server-encrypted.key`
5. Extract the certificate: `pkcs12 -in server.pfx -clcerts -nokeys -out server.crt`
6. Extract the decrypted key: `rsa -in server-encrypted.key -out server-decrypted.key`

## Certificate Authority

1. Start an OpenSSL session.
2. Generate the CA key: `genrsa -des3 -out ca.key 4096`
3. Create a CA certificate: `req -new -x509 -days 365 -key ca.key -out ca.crt -config openssl-ca.conf`
4. Create a client certificate
    1. Create an RSA key: `genrsa -des3 -out user.key 4096`
    2. Create a Certificate Signing Request (CSR): `req -new -key user.key -out user.csr -config openssl-user.conf`
    3. Sign the CSR: `x509 -req -days 365 -in user.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out user.crt`
    4. Create a PKCS #12 package with certificate and private key: `pkcs12 -export -out user.pfx -inkey user.key -in user.crt -certfile ca.crt`

## Docker

Build: `docker build -t hello .`
Run: `docker run -d -it -p 443:443 hello`
