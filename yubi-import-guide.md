# YubiHSM2 → AWS KMS Key Import (Full Guide)

This guide outlines how to generate an ECC NIST P-256 key pair in a YubiHSM2 (firmware 2.4+), wrap and export the private key, and import it into AWS KMS as externally generated material.

---

## 🛠 Step 1: Generate Key in YubiHSM2

```bash
# Create ECC P-256 key
YubiHSM> generate asymmetric 0 0x100 "YubiHSM Signing Key" 1 exportable-under-wrap,sign-ecdsa ecp256
```

```bash
# Export the public key
YubiHSM> get pubkey 0 0x100 asymmetric-key key0x100.pub
```

---

## ☁️ Step 2: Create External Key in AWS KMS

Use AWS Console:

- **Key Type**: Asymmetric  
- **Key Usage**: Sign & Verify  
- **Key Spec**: ECC_NIST_P256  
- **Key Material Origin**: External  
- **Alias**: key0x100

Then:

- **Wrapping Key Spec**: RSA_4096  
- **Wrapping Algorithm**: RSA_AES_KEY_WRAP_SHA256  
- **Download**: WrappingPublicKey + ImportToken

---

## 🔁 Step 3: Convert Wrapping Key Format

```bash
unzip ~/Downloads/Import_Parameters_*.zip
openssl rsa -pubin -inform DER -in WrappingPublicKey.bin -out wrappingkey-aws.pem
```

---

## 🔐 Step 4: Import Wrapping Key to YubiHSM2

```bash
YubiHSM> put wrapkey-public 0 0x101 "AWS KMS Wrap Key" all export-wrapped exportable-under-wrap,sign-ecdsa,encrypt-cbc,decrypt-cbc wrappingkey-aws.pem
```

---

## 📦 Step 5: Wrap the Private Key

```bash
YubiHSM> get wrapped-asym-raw 0 0x101 asymmetric-key 0x100 aes256 rsa-oaep-sha256 key0x100-wrapped-aws.bin
```

Upload both the wrapped private key and the import token to AWS KMS as part of the external key material upload.

---

## ✍️ Step 6: Sign a Message with KMS

```bash
cat message-aws.txt | base64

aws kms sign \
  --key-id alias/key0x100 \
  --message "TWVzc2FnZSBzaWduZWQgYnkgQVdTIEtNUwo=" \
  --signing-algorithm ECDSA_SHA_256 \
  | jq -r .Signature | tee message-aws.signature.b64
```

---

## 🧪 Step 7: Verify Signature

```bash
base64 -d message-aws.signature.b64 > message-aws.signature
openssl dgst -sha256 -verify key0x100.pub -signature message-aws.signature message-aws.txt
```

---

© Yubico 2025. Adapted for GitHub presentation by Neil.
