# MCNBlockchain

This repository does **not** contain any custom blockchain code.  
MCNBlockchain is fully based on the standard **Geth** (Go-Ethereum) client — everything works using Geth as-is.

The repository only includes configuration and service files required to run the MCN mainnet:

---

## 📂 Files

- **genesis-mainnet.json** — Ready-to-use mainnet genesis file.
- **geth-main.service** — Systemd service for running a first main node.
- **geth-public.service** - Systemd service for running a public node.
- **geth-additional-nodes.service** - Systemd service for running additional nodes (validators).
- **geth-explorer-node.service** - Systemd service for running node for explorer.
- **config.toml** — List of enodes to connect to the network (static nodes).

---

## Node Types

### 1. Main Validator Node


### 2. Additional Validator Nodes


### 3. Public RPC Nodes


### 4. Explorer RPC Node


---

## P2P Static Peering

Used to ensure stable validator connectivity and prevent network isolation.

---

## Security:

* No unlocked accounts on validator nodes
* CORS restricted
* VHosts restricted
