# SEED Labs: Secret-Key Encryption Report

This report contains detailed tasks and results from the SEED Lab on symmetric encryption using tools like OpenSSL and Python.

---

## Task 1: Frequency Analysis

### Mapping

- Original list: `nyvxuqmhtipaczlgbredfskjow`
- Most frequent letters: `etaoinshrdlcumwfgypbvkjxqz`
- Mapping result: `vgapnbrtmwsicuxeohqyzflkdj` → `abcdefghijklmnopqrstuvwxyz`

### Command
```bash
tr 'vgapnbrtmwsicuxeohqyzflkdj' 'abcdefghijklmnopqrstuvwxyz' < ciphertext.txt > hadioutput.txt
```

### Output
![Frequency Analysis Input](images/frequency_input.png)
![Frequency Analysis Output](images/frequency_output.png)

---

## Task 2: Encryption with Different Ciphers

### 1. AES-128-CBC

```bash
openssl enc -aes-128-cbc -e -in plain.txt -out cipher.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

> 128-bit key and 64-bit IV used

![AES Cipher Text](images/aes_output.png)
![AES Decryption](images/aes_decryption.png)

### 2. DES-CBC

```bash
openssl enc -des-cbc -e -in plain.txt -out cipher2.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

![DES Output](images/des_output.png)

### 3. Blowfish (BF-CBC)

```bash
openssl enc -bf-cbc -e -in plain.txt -out cipher3.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

---

##  Task 3: ECB vs CBC (Image Encryption)

###  Original Image
![Original Image](images/original_image.png)

### Commands
```bash
openssl enc -aes-128-ecb -e -in pic_original.bmp -out enc_ecb.bmp -K 00112233445566778889aabbccddeeff
openssl enc -aes-128-cbc -e -in pic_original.bmp -out enc_cbc.bmp -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

### Results
- ECB Encrypted: ![ECB](images/ecb_encrypted.png)
- CBC Encrypted: ![CBC](images/cbc_encrypted.png)

> **Observation**: ECB leaks image patterns, CBC hides them.

---

## Task 4: Padding

### Description
ECB and CBC need padding. OFB and CFB do not.

![File Sizes](images/padding_files.png)
![Hexdump](images/padding_hexdump.png)

> 6 bytes of padding (`0x06`) added when block size is not aligned

---

##  Task 5: Error Propagation

### Commands
```bash
openssl enc -aes-128-ecb -e -in fish.txt -out output.bin -K 00112233445566778889aabbccddeeff
bless output.bin  # hex editor to corrupt 55th bit
openssl enc -aes-128-ecb -d -in output.bin -out hadidec.txt -K 00112233445566778889aabbccddeeff
```

![Bit Corruption](images/error_propagation.png)

### Observations

- **ECB**: Only the corrupted block is broken.
- **CBC**: Two blocks are affected.
- **CFB/OFB**: Small localized corruption.

---

## Task 6: Initialization Vector (IV) Mistakes

### Different IVs
```bash
openssl enc -aes-128-cbc -e -in plaintxt.txt -out enc1.bin -K ... -iv ...
openssl enc -aes-128-cbc -e -in plaintxt.txt -out enc2.bin -K ... -iv ...
```

![Diff IV](images/iv_diff.png)

### Same IVs
```bash
openssl enc -aes-128-cbc -e -in plaintxt.txt -out enc4.bin -K ... -iv ...
```

![Same IV](images/iv_same.png)

> Same IV = same ciphertext = insecure

---

## Task 6.2: Keystream Reuse in CFB

### Equation
```
K = P1 ⊕ C1
P2 = C2 ⊕ K
```

> Same IV reuse allows full recovery of new plaintext (P2)

---

## Task 6.3: Oracle Attack via Docker

```bash
dcbuild
dcup
nc 10.9.0.80 3000
```

![Docker Run](images/docker_run.png)
![Oracle Access](images/oracle_access.png)

> Python script modifies IV to generate valid ciphertext.

---

## Task 7: Crypto Programming (Python)

### Given
- Plaintext: `This is a top secret.`
- Ciphertext: `764aa26b55a4da...`
- IV: `aabbccddeeff00998877665544332211`

### Python Command
```bash
python3 mhadi.py "This is a top secret." <cipher_hex> <iv_hex>
```

![Key Extraction](images/key_found.png)

> **Recovered Key**: `Syracuse########`

---

## Tools Used

- OpenSSL
- Python 3
- Docker
- Netcat (`nc`)
- Hex Editor (`bless`)

---

## Credits

Based on the [SEED Labs – Crypto Encryption Lab](https://seedsecuritylabs.org/Labs_20.04/Crypto/Crypto_Encryption/)

Report by *Zehra Qureshi*
