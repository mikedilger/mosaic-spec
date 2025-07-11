# Encrypted Secret Key

Encrypted secret keys are presented in a printable form with a `mocryptsec0` prefix.

## Secret Handling

In situations where a user could enter a password to decrypt a secret key, the secret key
should only be stored in this encrypted form and the user prompted for the password to decrypt
it. The decrypted secret key should be kept only in memory and as best possible zeroized before
memory is released.

Within this algorithm, three pieces of data are secret and SHOULD be zeroed and handled
as secret key material:  `PASSWORD`, `SYMMETRIC_KEY` and `SECRET_KEY`.

## Encryption algorithm

`PASSWORD` = Read from the user. The password should be unicode normalized to NFKC format
to ensure that the password can be entered identically on other computers/clients.

`LOG_N_BYTE` = Let the user or implementer choose one byte representing a power of 2 (e.g.
18 represents 262,144) which is used as the number of rounds for scrypt. Larger numbers take
more time and more memory, and offer better protection. 18 is a reasonable default as of
2025:

| LOG_N | MEMORY REQUIRED | APPROX TIME ON FAST COMPUTER |
|-------|-----------------|----------------------------- |
| 16    | 64 MiB          | 100 ms                       |
| 18    | 256 MiB         |                              |
| 20    | 1 GiB           | 2 seconds                    |
| 21    | 2 GiB           |                              |
| 22    | 4 GiB           |                              |

`SALT` = 16 random bytes

`SYMMETRIC_KEY` = `scrypt(password=PASSWORD, salt=SALT, log_n=LOG_N, r=8, p=1)`

The symmetric key should be 32 bytes long. This symmetric encryption key is temporary and
SHOULD be zeroed and MUST be discarded after use and not stored or reused for any other purpose.

`SECRET_KEY` = The secret key to be encrypted, as 32 bytes (not as human readable mosec0).

`XOR_OUTPUT` = `xor(SYMMETRIC_KEY, SECRET_KEY)`

`VERSION_NUMBER_BYTE` = 0x01

`CONCATENATION` = `concat( VERSION_NUMBER_BYTE, LOG_N_BYTE, SALT, XOR_OUTPUT )`

`ENCRYPTED_SECRET_KEY` = `zbase32_encode('mocryptsec', CONCATENATION)`

## Decryption algorithm

Perform the Encryption algorithm in reverse.
