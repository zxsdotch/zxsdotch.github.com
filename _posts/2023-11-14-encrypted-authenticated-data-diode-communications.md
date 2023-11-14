---
title: Encrypted & authenticated data diode communications
categories:
  - network security
  - cryptography
---
Data diodes provide physically guaranteed one-way communications in computer
networks. Data diodes can be built using commercial off the shelf components,
such as by disconnecting the receive end of a fiber transceiver. Several
dedicated appliances are also available. Traditionally, data diodes have been
used in high security settings, such as military or industrial control systems.
Data diodes might also have a place in other industries, such as
health care or finance, especially to transmit logs, analytics, usage, or
billing information.

# Given a data diode, how would you encrypt the data you are transmitting?
Modern cryptographic protocols, such as TLS, require an initial
handshake to establish session keys (and gain properties such as
perfect forward secrecy). If you are dealing with one-way communications,
you'll have to either use existing file encryption protocols (which will
result in lots of bytes of overhead) or design your cryptographic protocol.

A simple architecture can be implemented with minimal
engineering effort:
- an emitter-proxy which encrypts data inside the secure network.
- a data diode which connects the emitter-proxy to the receiver-proxy.
- a receiver-proxy which decrypts the data, outside the secure network.

![diagram](/images/encrypted-authenticated-data-diode-communications.svg)

Each proxy gets a X25519 key pair and knows the public key of the other proxy.

The emitter-proxy implements the following:
1. derives a shared key by performing a Diffie-Hellman key exchange, followed
  by a KDF. This step only needs to be performed once, it is therefore possible
  to use more expensive constructs with possibly better security margins.
2. listens for incoming data packets.
3. encrypts each packet with AES-GCM or AES-GCM-SIV using the shared key derived
  above.
4. sends the encrypted packet to the data diode.

The receiver-proxy is similar:
1. derives the same shared key by performing its own Diffie-Hellman key exchange
  and KDF operation.
2. listens for incoming data packets.
3. decrypts each packet with AES-GCM or AES-GCM-SIV using the shared key.
4. forwards the decrypted packet to wherever the data needs to go.

The result is an encrypted & authenticated data diode communication. The
encryption overhead can be very minimal: 12 bytes for the IV and 16 bytes for
the TAG.

# Interested to implement this?
There are a few details I didn't cover, such as heartbeats and ensuring packets
aren't lost in transit, as well as making key rotations easier -- all tractable
problems. As a co-author of [Ghostunnel](https://github.com/ghostunnel/ghostunnel),
an encryption/decryption proxy widely used in a production setting,
I am confident I can help you -- feel free to [contact](/contact) me.
