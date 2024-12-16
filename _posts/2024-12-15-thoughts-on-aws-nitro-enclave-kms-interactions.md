---
title: Thoughts on AWS Nitro Enclave &#x21d4; AWS KMS interactions
categories:
  - cryptography
  - AWS Nitro Enclave
---
At Zxs, we have been involved in several AWS Nitro Enclave-related projects. We have noticed a few areas where the AWS documentation doesn’t clearly mention some cryptographic considerations. We have discussed these issues with the security team at AWS and we understand that their hands are tied: they can’t easily change their existing design/APIs and they don’t want to risk confusing their developers by appending their documentation.

This post describes several issues you should take into account when designing AWS Nitro Enclaves which communicate with AWS KMS for persistent key operations. We’ll assume you are familiar with the following pieces of documentation:
- [AWS Nitro Enclaves User Guide](https://docs.aws.amazon.com/enclaves/latest/user/nitro-enclave.html)
- [How AWS Nitro Enclaves uses AWS KMS - AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/services-nitro-enclaves.html)

Nitro Enclaves enable running code in a confined environment. Communication is limited to the parent instance using a vsock. If code running in an enclave needs to communicate with AWS KMS, it might seem like there are two setups:
- setup 1: create an ephemeral RSA keypair and generate an attestation in the enclave. Pass the attestation to the parent instance. Let the parent instance call AWS KMS. The response is encrypted to the ephemeral RSA keypair. The parent instance then forwards the encrypted response to the enclave.
- setup 2: have the parent instance provide its IAM credentials to the enclave and setup a TCP vsock ⇔ AWS KMS bridge (you can call this a proxy or a port forward if you prefer). The enclave then establishes a TLS connection to AWS KMS. The enclave still needs to generate an ephemeral RSA keypair and an attestation in order to use AWS KMS key policies tied to a specific enclave.

## Observation 1: AWS KMS responses aren’t authenticated
In the first setup, a man-in-the-middle (MITM) attacker on the enclave’s parent instance sees the public RSA key in the attestation. The attacker can therefore generate any encrypted response. This implies, it is impossible to trust most of AWS KMS’ responses in this setup. The only exceptions are Decrypt and DeriveSharedSecret when the response is meant to be an encrypted content-encryption-key (CEK), in which case an attacker cannot forge responses. Assuming the CEK is used with an authenticated cipher.

## Observation 2: Encrypted KMS responses are malleable
When AWS KMS returns an encrypted response, the encryption algorithm is currently fixed to RSA with AES-CBC (see [cms.c in aws-nitro-enclaves-sdk-c](https://github.com/aws/aws-nitro-enclaves-sdk-c/blob/main/source/cms.c#L48)). AES-CBC is an unauthenticated encryption mode and there is no additional HMAC in the response. An attacker can flip bits in the ciphertext and corresponding bits will flip in the plaintext. We now have two reasons for using the second setup and performing the TLS handshake from within the enclave. 

## Observation 3: GenerateRandom is probably never useful
The only safe way to request random bytes (with the intention of increasing one’s entropy pool) from AWS KMS is to first establish a TLS connection, which itself requires having a good source of entropy, a catch-22. What is going on with [Generate random using kmstool-enclave-cli](https://github.com/aws/aws-nitro-enclaves-sdk-c/issues/131)?

In any case, the Enclave has mutiple sources of entropy and can receive additional
entropy on a case-by-csae basis (e.g. while processing requests).

## Observation 4: The downside of initiating the TLS handshake from enclaves
The major downside of initiating the TLS handshake from within enclaves is that any changes to the TLS configuration (e.g. changes in which root certificates are trusted) will be akin to changing your enclave’s code and can result in different PCR values.

The footprint of the additional code to handle TLS + HTTP can also be significant. It's code that enters the
enclave's trust boundary and must therefore be trusted and/or carefully audited.

We recommend that you configure the TLS client to only use TLS 1.3, secure ciphers, and only trust [Amazon's root CAs](https://www.amazontrust.com/repository/).

## Show don’t tell
[aws-nitro-enclave-foobar-service](https://github.com/zxsdotch/aws-nitro-enclave-foobar-service) is a piece of Go code we implemented to demonstrate how to securely create and use an AWS KMS-backed key. The code uses both setups (with and without TLS handshake from within the enclave) in a secure way.

AWS’ code examples [kmstool-enclave-cli](https://github.com/aws/aws-nitro-enclaves-sdk-c/tree/main/bin/kmstool-enclave-cli) and [vsock_proxy](https://github.com/aws/aws-nitro-enclaves-cli/blob/main/vsock_proxy) are equally good references when designing Nitro Enclaves.
