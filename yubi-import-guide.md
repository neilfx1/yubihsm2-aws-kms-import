# YubiHSM2 → AWS KMS Key Import (Full Guide)

This guide outlines how to generate an ECC NIST P-256 key pair in a YubiHSM2 (firmware 2.4+), wrap and export the private key, and import it into AWS KMS as externally generated material.

---

## 🛠 Step 1: Generate Key in YubiHSM2

```bash
# Generate an ECC P-256 key on the HSM
YubiHSM> generate asymmetric 0 0x100 "YubiHSM Signing Key" 1 exportable-under-wrap,sign-ecdsa ecp256
```

```bash
# Export the public key
YubiHSM> get pubkey 0 0x100 asymmetric-key key0x100.pub
```

---

## ☁️ Step 2: Create an Importable Key in AWS KMS

Go to AWS KMS Console and create a new key with the following options:

- **Key Type**: Asymmetric  
- **Key Usage**: Sign & Verify  
- **Key Spec**: ECC_NIST_P256  
- **Key Material Origin**: External
- **Regionality**: Single-Region-Key  
- **Alias**: key0x100
- **Administrative Permissions**: OrganizationAccountAccessRole
- **Key Users**: OrganizationAccountAccessRole

Select the Wrapping Key Options:

- **Wrapping Key Spec**: RSA_4096  
- **Wrapping Algorithm**: RSA_AES_KEY_WRAP_SHA256

Download the Wrapping Public Key and Import Token. 

---

## 🔁 Step 3: Prepare the Wrapping Key for YubiHSM2

```bash
unzip ~/Downloads/Import_Parameters_*.zip
openssl rsa -pubin -inform DER -in WrappingPublicKey.bin -out wrappingkey-aws.pem
```

---

## 🔐 Step 4: Import Wrapping Key into YubiHSM2

```bash
YubiHSM> put wrapkey-public 0 0x101 "AWS KMS Wrap Key" all export-wrapped exportable-under-wrap,sign-ecdsa,encrypt-cbc,decrypt-cbc wrappingkey-aws.pem
```

---

## 📦 Step 5: Wrap the Private Key

```bash
YubiHSM> get wrapped-asym-raw 0 0x101 asymmetric-key 0x100 aes256 rsa-oaep-sha256 key0x100-wrapped-aws.bin
```

Upload the wrapped key and metadata to AWS KMS.

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
cat message-aws.signature.b64 | base64 -d > message-aws.signature
cat key0x100.pub
cat message-aws.txt
openssl dgst -sha256 -verify key0x100.pub -signature message-aws.signature message-aws.txt
```

---
