# MCNBlockchain

This repository does **not** contain any custom blockchain code.  
MCNBlockchain is fully based on the standard **Geth** (Go-Ethereum) client — everything works using Geth as-is.

The repository only includes configuration and service files required to run the MCN mainnet:

## 📂 Files

- **genesis-mainnet.json** — Ready-to-use mainnet genesis file.
- **geth.service** — Systemd service for running a Geth node.
- **split.js** — Small helper script (not blockchain code).
- **splitter.service** — Service for running the helper script.
- **README.md** — Documentation.

## ✅ Summary

There is **no custom protocol**,  
no smart contracts,  
no modifications to Geth,  
no application logic,  
no blockchain code.

This repo only holds the **configuration files** used to run MCN mainnet on top of the standard Geth client.
