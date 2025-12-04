# MCNBlockchain

This repository does **not** contain any custom blockchain code.  
MCNBlockchain is fully based on the standard **Geth** (Go-Ethereum) client — everything works using Geth as-is.

The repository only includes configuration and service files required to run the MCN mainnet:

## 📂 Files

- **genesis-mainnet.json** — Ready-to-use mainnet genesis file.
- **geth.service** — Systemd service for running a Geth node.
- **README.md** — Documentation.

### DFD - Duplicate Fee Deduction

**Criticality:** Critical  
**Location:** `split.js#L42`  
**Status:** Resolved  

**Description:**  
Originally, `split.js` was used to distribute transaction fees to multiple addresses. The script operated in `'total'` gas share mode, which caused double-counting of the base fee under EIP-1559, leading to potential unintended loss for the miner. At that time, we had only one validator, so this redistribution was unnecessary and risky.

**Solution:**  
Now, we operate with five real validators and no longer use `split.js`. Each validator receives its own proper fees directly, eliminating the risk of duplicate fee deduction.

### AGI - Agnostic GETH Implementation

**Criticality:** Minor / Informative  
**Location:** `geth.service#L13`  
**Status:** Resolved  

**Description:**  
The systemd service for running the Geth client on the MCN chain previously referenced the binary directly at `/usr/local/bin/geth` without specifying a version. This setup relied on the locally installed Geth, which could be updated or replaced without notice, potentially causing inconsistencies, unexpected behavior, or security risks if an untested binary was executed.

**Solution:**  
We now use a version-pinned binary at `/opt/geth-1.13.11-stable/geth`. This ensures all deployments use a tested and verified Geth release, improving consistency and security across nodes.

### CR - Centralization Risk

**Criticality:** Minor / Informative  
**Location:** `geth.service#L29`, `split.js#L9`  
**Status:** Resolved  

**Description:**  
Initially, the chain had a centralized setup with a single validator responsible for sealing blocks. This created several risks:  
- **Single Point of Control:** The chain's operation depended entirely on one entity, increasing the risk of downtime or manipulation.  
- **Centralized Fee Distribution:** The `split.js` script distributed transaction fees to a predefined list of validators, who were not all active in consensus, causing discrepancies.  
- **Decentralization Erosion:** Without adding new validators promptly, the chain remained centralized longer than intended, undermining its trustless design.  

**Solution:**  
We now operate with five real validators, and `split.js` is no longer used. This setup distributes responsibilities and fees directly to active validators, significantly reducing centralization risks and aligning the network with a more trustless, decentralized model.

### CFD - Concurrent Fee Distribution

**Criticality:** Minor / Informative  
**Location:** `split.js`  
**Status:** Resolved  

**Description:**  
The `_isSplitBusy` flag in `split.js` was meant to act as a lock to prevent reentrant execution of `processBlocksAndDistribute()`. However, it was not implemented with atomic or thread-safe operations. This could lead to race conditions where multiple instances run concurrently, causing inconsistent state updates or duplicate fund distributions.

**Solution:**  
Since `split.js` is no longer used, the risk of concurrent fee distribution and race conditions is completely eliminated. Fee handling now occurs directly on the active validators, ensuring consistent and reliable distribution.

### EBGL - Excessive Block Gas Limit

**Criticality:** Minor / Informative  
**Location:** `genesis-mainnet.json#L20`  
**Status:** Resolved  

**Description:**  
The genesis block originally set the `gasLimit` to `900,000,000`. Such a high limit allowed single transactions to consume excessive computational resources, risking performance degradation or denial-of-service attacks on node hardware.

**Solution:**  
The `gasLimit` has been reduced to `50,000,000`, limiting the maximum computational effort per block and improving network stability and security.

### GSV - Global State Variables

**Criticality:** Minor / Informative  
**Location:** `split.js`  
**Status:** Resolved  

**Description:**  
Global variables such as `_feePool`, `_lastProcessedBlk`, and `_isSplitBusy` were defined in the global scope of `split.js` without namespacing. This increased the risk of variable collisions if multiple scripts ran in the same runtime, potentially causing unpredictable behavior or logic errors.

**Solution:**  
Since `split.js` is no longer used, all related global state risks are eliminated. The current setup avoids naming conflicts and ensures predictable execution across validators.

### HV - Hardcoded Values

**Criticality:** Minor / Informative  
**Location:** `split.js`  
**Status:** Resolved  

**Description:**  
The script used a hardcoded gas limit of 21,000 and a fixed priority fee of 1 gwei for all transactions. This setup did not account for varying network conditions or more complex transactions, potentially causing failures if actual gas requirements exceeded these fixed values.

**Solution:**  
Since `split.js` is no longer used, transactions now rely on dynamic gas estimation and appropriate fee settings, ensuring reliable execution under different network conditions.

### MFIM - Missing Fee Incentivization Mechanism

**Criticality:** Minor / Informative  
**Location:** `geth.service`  
**Status:** Resolved  

**Description:**  
Previously, the system directed mining rewards and transaction fees to a predefined `etherbase` address, incentivizing only a specific set of validators. Users were not motivated to include priority fees in their transactions, which could reduce the effectiveness of fee-based incentives, especially in low-congestion conditions.

**Solution:**  
The Geth service has been updated with an improved configuration, including a realistic `--miner.gasprice` and caps for RPC transaction fees (`--rpc.gascap` and `--rpc.txfeecap`). This setup encourages proper fee inclusion by users and aligns validator incentives with network economic rewards, promoting sustainable participation and network security.

### MFC - Missing Flag Configurations

**Criticality:** Minor / Informative  
**Location:** `geth.service`  
**Status:** Partially Resolved  

**Description:**  
Geth provides runtime flags to safeguard node performance and prevent misuse of its JSON-RPC interface. Key flags include:  
- `--rpc.gascap`: limits maximum gas for read-only calls (`eth_call`, `eth_estimateGas`) to prevent excessive CPU usage.  
- `--rpc.txfeecap`: limits the maximum transaction fee accepted via `eth_sendTransaction`, protecting against accidental high-fee transactions.  
- `--http.idleseconds` and `--http.timeout`: control idle HTTP connections and request processing duration.  

Previously, these flags were not configured, leaving nodes vulnerable to resource exhaustion and potential abuse.

**Solution:**  
The service now includes `--rpc.gascap` and `--rpc.txfeecap`, enforcing limits on gas usage and transaction fees.  
Note: `--http.idleseconds` and `--http.timeout` are not available in the current Geth version, so HTTP connection limits are implemented via Nginx, ensuring idle and slow connections are properly controlled.

### MIV - Missing Input Validation

**Criticality:** Minor / Informative  
**Location:** `split.js`  
**Status:** Resolved  

**Description:**  
The `split.js` distribution script previously did not validate the `VALIDATORS` array or the corresponding `WEIGHTS`. This lack of input validation could lead to unpredictable behavior, failed transactions, or incorrect allocation of funds.

**Solution:**  
Since `split.js` is no longer used, all validator fee handling now occurs directly and safely on active validators. This eliminates the risk of misconfigured addresses or weights, ensuring correct and reliable fee distribution.

### NPFS - Non Persistent Fee State

**Criticality:** Minor / Informative  
**Location:** `split.js`  
**Status:** Resolved  

**Description:**  
The `split.js` script previously relied on a temporary filter (`eth.filter("latest")`) to trigger fee distribution when new blocks were mined. Since this filter was not persistent, any restart of Geth or the script would reset internal state variables like `feePool` and `lastProcessedBlk`, resulting in permanent loss of unprocessed fee data and inconsistent fee distribution.

**Solution:**  
`split.js` is no longer used. All fee handling now occurs directly on active validators, eliminating risks of lost state and ensuring consistent, reliable fee distribution across node restarts.

### PRA - Permissive RPC Access

**Criticality:** Minor / Informative  
**Location:** `geth.service`  
**Status:** Resolved  

**Description:**  
Previously, Geth was configured with `--http.vhosts "*"` and `--http.corsdomain "*"`, allowing requests from any origin or host. If the node's HTTP port were exposed publicly, this would enable malicious websites to send arbitrary JSON-RPC requests, including transaction submissions, especially dangerous because `--allow-insecure-unlock` and unlocked accounts were in use.

**Solution:**  
The public node now restricts HTTP access with:  
- `--http.vhosts="rpc.mcnchain.org"`  
- `--http.corsdomain="https://rpc.mcnchain.org"`  

HTTP and WebSocket interfaces are bound to `0.0.0.0` for the public node only, while private validator nodes are not exposed to the internet. This ensures safe public access while keeping validator nodes secure.


### PFD - Possible Funds Discrepancy

**Criticality:** Minor / Informative  
**Location:** `split.js#L91`  
**Status:** Resolved  

**Description:**  
The `split.js` script previously used a `safeSend` function to distribute ETH, including to the burn address (`0x000000000000000000000000000000000000dEaD`). This function did not verify whether transactions were successfully mined, potentially causing silent loss of funds, as amounts were deducted from `feePool` regardless of transaction success.

**Solution:**  
`split.js` is no longer used. All fee distribution now occurs directly on active validators, eliminating the risk of silent fund loss and ensuring reliable and accurate handling of transaction fees.

### RUC - Redundant Unlock Configuration

**Criticality:** Minor / Informative  
**Status:** Resolved  

**Description:**  
The Geth node was previously configured with `--allow-insecure-unlock`, allowing accounts to be unlocked via HTTP or WebSocket RPC. This is intended for development and poses a security risk in production. Even though the validator key is unlocked at startup using `--unlock` and `--password`, `--allow-insecure-unlock` unnecessarily expands the unlock surface, risking unauthorized access if the RPC interface is exposed.

**Solution:**
The `--allow-insecure-unlock` flag has been removed. Accounts are now securely unlocked when running on a private server with no external access using `--unlock` and `--password`, reducing the risk of unauthorized access and improving overall node security.

### UPPA - Unrestricted P2P Port Access

**Criticality:** Minor / Informative  
**Location:** `geth.service`  
**Status:** Resolved  

**Description:**  
The Geth node by default listens on P2P port 30303 for TCP and UDP traffic and enables public peer discovery. This exposes the node to unsolicited connections from the internet, which could be exploited with malformed blocks, oversized transaction pools, or other resource-intensive traffic, potentially causing performance degradation or denial-of-service.

**Solution:**  
For private validators, `--maxpeers` is configured to limit the number of peer connections, effectively controlling exposure. This reduces the attack surface and prevents unwanted P2P connections, ensuring more secure operation of the private network nodes.
