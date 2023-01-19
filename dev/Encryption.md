# Encryption schematic

WFB-NG provides unswitchable encryption: it is impossible to establish connection without key agreement between drone and ground station.

Drone (TX part) keeps his TX private key and GS (RX) public key.
Ground stations holds RX private key and drone TX public key.

To initialize TX<->RX keypairs run ``keygen`` utility. It will make ``drone.key`` and ``gs.key``.

Encryption is applied to content of wifi packets, so that third party doesn't see any contents. Structure of data is following:

```
    radiotap_header:
       ieee_80211_header:
         1. Data packet:
            wblock_hdr_t   { packet_type = 1, nonce = (block_idx << 8) + fragment_idx }
              wpacket_hdr_t  { flags, packet_size }  #
                data                                 #
                                                     +-- encrypted and authenticated by session key
         2. Session packet:
            wsession_hdr_t { packet_type = 2, nonce = random() }
              wsession_data_t { epoch, channel_id, fec_type, fec_k, fec_n, session_key } # -- encrypted and signed using crypto_box_easy(rx_publickey, tx_secretkey)

    data nonce:  56bit block_idx + 8bit fragment_idx
    session nonce: crypto_box_NONCEBYTES of random bytes
```

Where `data` is payload of UDP packet. No IP/UDP headers transmitted.


# Encryption process

When TX starts, it generates new session key, encrypts it using public key authenticated encryption (``cryptobox``) and announce it every ``SESSION_KEY_ANNOUNCE_MSEC`` (default 1s). 

Take a look at `src/tx.cpp`  `make_session_key()` and `send_session_key`.

Data packets encrypted by ``crypto_aead_chacha20poly1305_encrypt`` using session key and packet index as nonce.

Data packets and key packets are distinguished by `packet_type` field send on wifi layer:
`WFB_PACKET_KEY` vs `WFB_PACKET_DATA`


# Key renewal

TX<->RX keypairs are not transferred on air and do not leave the drone/GS.
New session key is regenerated each ``MAX_BLOCK_IDX`` packets (blocks).


# Encryption tools

* WFB-NG encrypts data stream using [libsodium](https://download.libsodium.org/doc/).
* ``keygen`` utility is used on GS to create two keypairs.

# Debugging

Debugging encryption is always not easy because it is designed to be not very easy for intruder to read data. However, here are hints:

``Unable to decrypt session key`` message (src/rx.cpp, around `WFB_PACKET_KEY`) means that
key pairs do not match. Usually it means that you need to regenerate keypairs and put one of them to drone, second one to ground station and restart daemons on both sides.

Be careful with `scripts/install_gs.sh` script because it will regenerate your GS keypair.
If you run it again after drone was installed and configured, you will get unmatched keypairs.

