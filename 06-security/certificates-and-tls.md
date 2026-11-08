# Certificates and TLS

TLS protects data in transit and lets clients verify the identity of a server. This guide covers certificate inspection and connection testing with OpenSSL; see the [OpenSSL command documentation](https://docs.openssl.org/).

## Important certificate files

| File | Purpose |
|---|---|
| Certificate (`.crt`, `.pem`, `.cer`) | Public certificate, often including a server identity and public key. |
| Private key (`.key`, `.pem`) | Secret key corresponding to the certificate's public key. |
| CA bundle / chain | Certificates used to build a trust chain to a trusted certificate authority. |
| CSR (`.csr`) | Certificate Signing Request sent to a certificate authority. |

File extensions do not guarantee encoding or content. Use `file` and OpenSSL to inspect a file.

## Inspect a certificate

```bash
openssl x509 -in server.crt -noout -subject -issuer -dates
openssl x509 -in server.crt -noout -fingerprint -sha256
openssl x509 -in server.crt -noout -text | less
```

| Option | Meaning |
|---|---|
| `-in <file>` | Read the certificate from a file. |
| `-noout` | Do not print the encoded certificate itself. |
| `-subject` | Show the certificate subject. |
| `-issuer` | Show the issuing CA. |
| `-dates` | Show validity start and expiry dates. |
| `-fingerprint -sha256` | Show the SHA-256 certificate fingerprint. |
| `-text` | Print certificate details in readable form. |

Check the Subject Alternative Name (SAN) extension when verifying that a certificate covers a hostname. Modern hostname validation relies on SAN, not just the Common Name:

```bash
openssl x509 -in server.crt -noout -ext subjectAltName
openssl x509 -in server.crt -noout -checkhost example.com
```

`-checkhost` checks whether the certificate matches the supplied hostname.

## Check a certificate served by a remote host

```bash
openssl s_client -connect example.com:443 -servername example.com -verify_return_error </dev/null
```

- `-connect` selects the host and port.
- `-servername` sends the TLS Server Name Indication (SNI) hostname, which is important when a server hosts multiple certificates.
- `-verify_return_error` makes verification failures affect the command result instead of merely displaying them.

To display the presented certificate's subject, issuer, and dates:

```bash
openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -subject -issuer -dates
```

This second command is useful for inspection but does not by itself prove that the certificate chain is trusted or that hostname validation succeeds. Use the first command for a verification-oriented connection test.

## Inspect a private key

```bash
openssl pkey -in server.key -noout -check
openssl pkey -in server.key -pubout
```

Do not paste private keys into chat, tickets, public repositories, or online certificate checkers. Restrict the key file to the service account or administrators that need it.

## Inspect a CSR

```bash
openssl req -in server.csr -noout -subject -text
```

Verify the requested SAN entries, subject details, and public-key algorithm before submitting the CSR to a certificate authority.

## Verify certificate and key match

Compare the public key derived from the certificate and private key:

```bash
openssl x509 -in server.crt -pubkey -noout | openssl pkey -pubin -outform DER | sha256sum
openssl pkey -in server.key -pubout -outform DER | sha256sum
```

The hashes should match. This works for common RSA and elliptic-curve key types supported by OpenSSL.

## Renewal and deployment

- Use a trusted certificate authority and automate renewal where practical.
- Keep private keys readable only by the service and administrators that need them.
- Install the complete certificate chain in the format expected by the service.
- Validate service configuration before reload or restart.
- Reload the service after renewal when supported, then check the certificate served over the network.
- Monitor expiry and renewal failures.

Certificate issuance tools such as Certbot have service- and CA-specific procedures.
