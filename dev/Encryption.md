WFB-NG encrypts data stream using [libsodium](https://download.libsodium.org/doc/).

When TX starts, it generates new session key, encrypts it using public key authenticated encryption (``cryptobox``) and announce it every ``SESSION_KEY_ANNOUNCE_MSEC`` (default 1s).  Data packets encrypted by ``crypto_aead_chacha20poly1305_encrypt`` using session key and packet index as nonce.  

To initialize TX->RX keypair run ``keygen`` utility. It will make ``drone.key`` and ``gs.key``.
