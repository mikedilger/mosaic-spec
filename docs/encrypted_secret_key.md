# Encrypted Secret Key

Encrypted secret keys are presented in a printable form with a `mocryptsec0` prefix.

## Secret Handling

In situations where a user could enter a password to decrypt a secret key, the secret key
should only be stored in this encrypted form and the user prompted for the password to decrypt
it. The decrypted secret key should be kept only in memory and as best possible zeroized before
memory is released.

Within this algorithm, three pieces of data are secret and SHOULD be zeroed and handled
as secret key material:  `PASSWORD`, `SYMMETRIC_KEY`,  `SECRET_KEY`, and `CHECKED_SECRET_KEY`.

## Encryption algorithm

### Inputs

`SECRET_KEY` = The secret key to be encrypted, as 32 bytes (not as human readable mosec0).

`PASSWORD` = Read from the user. The password should be unicode normalized to NFKC format
to ensure that the password can be entered identically on other computers/clients.
The password is temporary and SHOULD be zeroed and MUST be discarded after use and not stored
or reused for any other purpose.

`LOG_N_BYTE` = Let the user or implementer choose one byte representing a power of 2 (e.g.
18 represents 262,144) which is used as the number of rounds for scrypt. Larger numbers take
more time and more memory, and offer better protection. 18 is a reasonable default as of
2025. Implementations MAY fail if this value is greater than 22:

| LOG_N | MEMORY REQUIRED | APPROX TIME ON FAST COMPUTER |
|-------|-----------------|----------------------------- |
| 16    | 64 MiB          | 100 ms                       |
| 18    | 256 MiB         |                              |
| 20    | 1 GiB           | 2 seconds                    |
| 21    | 2 GiB           |                              |
| 22    | 4 GiB           |                              |

### Algorithm

`SALT` = 16 random bytes

`SYMMETRIC_KEY` = `scrypt(password=PASSWORD, salt=SALT, log_n=LOG_N_BYTE, r=8, p=1, dkLen=36)`

The symmetric key output should be 36 bytes long (the `dkLen`). This symmetric encryption key is
temporary and SHOULD be zeroed and MUST be discarded after use and not stored or reused for any
other purpose.

`CHECKBYTES` = `[0xb9, 0x60, 0xa1, 0xe2]`

`CHECKED_SECRET_KEY` = `CONCAT(SECRET_KEY, CHECKBYTES)`

This value is temporary and SHOULD be zero and MUST be discarded after use and not stored or reused
for any other purpose.

`XOR_OUTPUT` = `xor(SYMMETRIC_KEY, CHECKED_SECRET_KEY)` [<sup>rat</sup>](rationale.md#xor)

`VERSION_NUMBER_BYTE` = 0x01

`CONCATENATION` = `concat( VERSION_NUMBER_BYTE, LOG_N_BYTE, SALT, XOR_OUTPUT )`

`ENCRYPTED_SECRET_KEY` = `CONCAT('mocryptsec0', zbase32_encode(CONCATENATION))`

## Decryption algorithm

### Inputs

`ENCRYPTED_SECRET_KEY` = Read as input

`PASSWORD` = Read from the user. The password should be unicode normalized to NFKC format
to ensure that the password can be entered identically on other computers/clients.
The password is temporary and SHOULD be zeroed and MUST be discarded after use and not stored
or reused for any other purpose.

### Algorithm

`ZBASE32_ENCODED` = `strip_and_check_prefix(ENCRYPTED_SECRET_KEY, "mocryptsec0")`

`CONCATENATION` = `zbase32_decode(ZBASE32_ENCODED)`

`VERSION_NUMBER_BYTE` = `CONCATENATION[0]`

Check that the `VERSION_NUMBER_BYTE` is 0x01 else fail.

`LOG_N_BYTE` = `CONCATENATION[1]`

Consider failing if this value is greater than 22 (which is compute excessive).

`SALT` = `CONCATENATION[2..18]`

`XOR_OUTPUT` = `CONCATENATION[18..54]`

`SYMMETRIC_KEY` = `scrypt(password=PASSWORD, salt=SALT, log_n=LOG_N_BYTE, r=8, p=1)`

The symmetric key output should be 36 bytes long (the `dkLen`). This symmetric encryption key is
temporary and SHOULD be zeroed and MUST be discarded after use and not stored or reused for any
other purpose.

`CHECKED_SECRET_KEY` = `xor(SYMMETRIC_KEY, XOR_OUTPUT)`

This value is temporary and SHOULD be zero and MUST be discarded after use and not stored or reused
for any other purpose.

`CHECKBYTES` = `CHECKED_SECRET_KEY[32..36]`

Verify these equal `[0xb9, 0x60, 0xa1, 0xe2]`. If not, presume the password was incorrect.

`SECRET_KEY` = `CHECKED_SECRET_KEY[0..32]`
