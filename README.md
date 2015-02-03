# SSL certificate chain resolver

This script downloads all intermediate CA certificates for a given SSL server certificate. It can help you fix the *incomplete certificate chain* issue, also reported as *Extra download* by [Qualys SSL Server Test](https://www.ssllabs.com/ssltest/).

## Usage

Dependencies:

- curl or wget
- OpenSSL

```
./resolve.sh input.pem output.pem
```

All certificates are in Base64-encoded PEM format.

Example:

```
$ grep 'CERTIFICATE' input.pem
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----

$ ./resolve.sh input.pem output.pem
1: OU=Domain Control Validated, OU=PositiveSSL Wildcard, CN=*.xxx.com
2: C=GB, ST=Greater Manchester, L=Salford, O=COMODO CA Limited, CN=COMODO RSA Domain Validation Secure Server CA
3: C=GB, ST=Greater Manchester, L=Salford, O=COMODO CA Limited, CN=COMODO RSA Certification Authority

Certificate chain complete.
Total 3 certificate(s) written.
Verified successfully.

$ grep 'CERTIFICATE' output.pem
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
```

## Description

![Incomplete certificate chain](incomplete-chain.png)

All operating systems contain a set of default trusted root certificates. But CAs usually don't use their root certificate to sign customer certificates. Instead of they use so called intermediate certificates, because they can be rotated more frequently.

A certificate can contain a special *Authority Information Access* extension ([RFC-3280](http://tools.ietf.org/html/rfc3280)) with URL to issuer's certificate. Most browsers can use the AIA extension to download missing intermediate certificate to complete the certificate chain. This is the exact meaning of the *Extra download* message. But some clients (mobile browsers, OpenSSL) don't support this extension, so they report such certificate as untrusted.

A server should always send a complete chain, which means concatenated all certificates from the certificate to the trusted root certificate (exclusive, in this order), to prevent such issues. Note, the trusted root certificate should not be there, as it is already included in the system’s root certificate store.

You should be able to fetch intermediate certificates from the issuer and concat them together by yourself, this tool helps you automatize it by looping over certificate's AIA extension field.

## Other implementations

- [freekmurze/ssl-certificate-chain-resolver](https://github.com/freekmurze/ssl-certificate-chain-resolver) (PHP)

## Licence

[MIT](LICENCE). Appreciations welcome.
