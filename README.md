# YubiHSM2 → AWS KMS Key Import Guide

This project walks through the process of generating an ECC key using a YubiHSM2 and importing it into AWS KMS using the "external key" flow. It demonstrates a real-world use case where a hardware-rooted key is generated securely on-prem, then wrapped and transferred to AWS for cloud signing operations.

## 🔐 Features

- ECC P-256 key generation on YubiHSM2
- PKCS#11 wrapping and export of private key
- AWS KMS import using RSA-OAEP key wrap
- Message signing using AWS CLI
- Signature verification using OpenSSL

## 📂 Files

- `yubi-import-guide.md` – Full step-by-step CLI walkthrough
- `README.md` – Overview of purpose and contents

## 🧪 Requirements

- YubiHSM2 with firmware 2.4+
- AWS KMS permissions for external key import
- OpenSSL
- AWS CLI
- jq (optional, for parsing CLI JSON)

## ✅ What You'll Do

1. Generate an ECC key in YubiHSM2
2. Export the public key
3. Create an external key in AWS KMS
4. Convert and load AWS wrapping key into YubiHSM2
5. Wrap and upload the private key to AWS
6. Sign and verify a message using KMS

---

This guide is based on successful testing with Vault + YubiHSM and is intended as a standalone integration reference.
