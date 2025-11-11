# MCNBlockchain

This repository contains the core configuration files used for the MCN Blockchain mainnet setup.  
No application logic or custom blockchain code is included — only the files required for infrastructure and deployment.

## 📂 Files

- **genesis-mainnet.json** — Mainnet genesis configuration used to initialize the network.
- **geth.service** — Systemd service file for running the Geth node as a background process.
- **split.js** — Helper script used for splitting and processing key-related data.
- **splitter.service** — Systemd service for running the split.js script automatically.
- **README.md** — Documentation for the repository.

## ✅ Purpose

This repository exists to store and manage:
- the official mainnet genesis file,
- supporting system services,
- utility scripts related to node operations.

It does *not* contain custom consensus code, or application modules.
