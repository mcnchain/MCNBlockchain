# MCNBlockchain

This repository does **not** contain any custom blockchain code.  
MCNBlockchain is fully based on the standard **Geth** (Go-Ethereum) client — everything works using Geth as-is.

The repository only includes configuration and service files required to run the MCN mainnet:

## 📂 Files

- **genesis-mainnet.json** — Ready-to-use mainnet genesis file.
- **geth-main.service** — Systemd service for running a first main node.
- **geth-public.service** - Systemd service for running a public node.
- **geth-additional-nodes.service** - Systemd service for running additional nodes.
- **config.toml** — List of enodes to connect to the network (static nodes).

### DFD - Duplicate Fee Deduction

**Solution:**  
Now, we operate with five real validators and no longer use `split.js`. Each validator receives its own proper fees directly, eliminating the risk of duplicate fee deduction.

### AGI - Agnostic GETH Implementation

**Solution:**  
We now use a version-pinned binary at `/opt/geth-1.13.11-stable/geth`. This ensures all deployments use a tested and verified Geth release, improving consistency and security across nodes.

### CR - Centralization Risk

**Solution:**  
We now operate with five real validators, and `split.js` is no longer used. This setup distributes responsibilities and fees directly to active validators, significantly reducing centralization risks and aligning the network with a more trustless, decentralized model.

### CFD - Concurrent Fee Distribution

**Solution:**  
Since `split.js` is no longer used, the risk of concurrent fee distribution and race conditions is completely eliminated. Fee handling now occurs directly on the active validators, ensuring consistent and reliable distribution.

### EBGL - Excessive Block Gas Limit

**Solution:**  
The `gasLimit` has been reduced to `50,000,000`, limiting the maximum computational effort per block and improving network stability and security.

### GSV - Global State Variables

**Solution:**  
Since `split.js` is no longer used, all related global state risks are eliminated. The current setup avoids naming conflicts and ensures predictable execution across validators.

### HV - Hardcoded Values

**Solution:**  
Since `split.js` is no longer used, transactions now rely on dynamic gas estimation and appropriate fee settings, ensuring reliable execution under different network conditions.

### MFIM - Missing Fee Incentivization Mechanism

**Solution:**  
The Geth service has been updated with an improved configuration, including a realistic `--miner.gasprice` and caps for RPC transaction fees (`--rpc.gascap` and `--rpc.txfeecap`). This setup encourages proper fee inclusion by users and aligns validator incentives with network economic rewards, promoting sustainable participation and network security.

### MFC - Missing Flag Configurations

**Solution:**  
The service now includes `--rpc.gascap` and `--rpc.txfeecap`, enforcing limits on gas usage and transaction fees.  
Note: `--http.idleseconds` and `--http.timeout` are not available in the current Geth version, so HTTP connection limits are implemented via Nginx, ensuring idle and slow connections are properly controlled.

### MIV - Missing Input Validation

**Solution:**  
Since `split.js` is no longer used, all validator fee handling now occurs directly and safely on active validators. This eliminates the risk of misconfigured addresses or weights, ensuring correct and reliable fee distribution.

### NPFS - Non Persistent Fee State

**Solution:**  
`split.js` is no longer used. All fee handling now occurs directly on active validators, eliminating risks of lost state and ensuring consistent, reliable fee distribution across node restarts.

### PRA - Permissive RPC Access

**Solution:**  
The public node now restricts HTTP access with:  
- `--http.vhosts="rpc.mcnchain.org"`  
- `--http.corsdomain="https://rpc.mcnchain.org"`  

HTTP and WebSocket interfaces are bound to `0.0.0.0` for the public node only, while private validator nodes are not exposed to the internet. This ensures safe public access while keeping validator nodes secure.


### PFD - Possible Funds Discrepancy

**Solution:**  
`split.js` is no longer used. All fee distribution now occurs directly on active validators, eliminating the risk of silent fund loss and ensuring reliable and accurate handling of transaction fees.

### RUC - Redundant Unlock Configuration

**Solution:**
The `--allow-insecure-unlock` flag has been removed. Accounts are now securely unlocked when running on a private server with no external access using `--unlock` and `--password`, reducing the risk of unauthorized access and improving overall node security.

### UPPA - Unrestricted P2P Port Access

**Solution:**  
For private validators, `--maxpeers` is configured to limit the number of peer connections, effectively controlling exposure. This reduces the attack surface and prevents unwanted P2P connections, ensuring more secure operation of the private network nodes.
