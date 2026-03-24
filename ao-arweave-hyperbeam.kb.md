# HyperBEAM, AO, & Arweave Development Knowledge Base

This knowledge base provides a complete reference for developing on the HyperBEAM / AO / Arweave technology stack. It covers the underlying blockchain mechanics, the Actor-Oriented (AO) messaging protocols, the HyperBEAM Erlang implementation, configuration, API interactions, and strict coding guidelines.

## 1. Core Architecture & Foundations

### The Hardware Metaphor
The Permaweb stack is best understood as a decentralized supercomputer where architecture is separated into specialized, infinitely scalable layers:
*   **The Hard Drive (Arweave):** Permanent, immutable memory.
*   **The CPU (AO):** A hyper-parallel processor enabling simultaneous, independent processes.
*   **The Motherboard/OS (HyperBEAM):** The routing environment connecting compute to memory.
*   **The Electricity ($AR & $AO):** The economic security and incentive layer.

### The Software Philosophy
HyperBEAM adopts a Unix-like "Everything is a Message" philosophy. Just as Unix treats every resource as a file to enable piping, AO treats every program state as a message to enable infinite remixing and "piping" between decentralized devices.

### 1.1 Arweave (The Storage Layer)
Arweave is a decentralized, permanent information storage protocol. It provides an arbitrarily scalable ledger of data by utilizing a novel consensus mechanism powered by Succinct Proofs of Random Access (SPORA). 
*   **Arweave 2.6+ Consensus & SPORA:** The network's mining mechanism was upgraded in the 2.6 hard fork, optimizing SPORA to force nodes to maintain the entire historical dataset to earn rewards. This drastically lowers the barrier to entry for miners, caps energy expenditure costs, and greatly increases overall decentralization.
*   **The Blockweave:** Blocks flexibly link to the previous block and a randomly selected historical block to ensure absolute data integrity. The top-level data structure is a merkelized list of 3-tuples (block hash, weave size, transaction root), allowing nodes to perform Simplified Payment Verification (SPV) without downloading intermediate headers.
*   **Verifiable Delay Function (VDF):** A chained hashing technique (SHA2-256) evaluates the passage of time. It outputs a checkpoint approximately once every second, unlocking challenges within the dataset.
*   **The Endowment Model:** Users pay a one-time upfront fee for permanent storage. Mathematically, 85% of this fee enters an endowment pool to guarantee continuous miner compensation independent of current network activity.
*   **Storage Economics (GiB Token):** Nau Finance's GiB stablecoin acts as the standard primitive for calculating long-term storage costs, ensuring developers price resources against a stable metric of byte-storage rather than fluctuating fiat or native AR values. 
*   **Node Operations:** To fulfill SPORA, the Node OS must undergo "Workload Tuning" optimizing for constant read-access of historical data chunks. Furthermore, wallet integration featuring a strict mining key is required to actively receive $AR rewards.

### 1.2 AO-Core Protocol (Hyper-Parallel Compute)
AO-Core is a protocol for **hyper-parallel compute**, replacing traditional "distributed computing" paradigms. It represents program states as HTTP documents and utilizes hashpaths to ensure verifiability across its unlimited parallel architecture. 

*   **The Three Pillars (MU, SU, CU):** The AO network categorically divides infrastructure node responsibilities into three distinct pillars:
    1. **Messenger Units (MUs):** Handle message ingress, routing, and delivery across the network.
    2. **Scheduler Units (SUs):** Assign verifiable cryptographic sequence numbers to messages (hashpaths), ensuring deterministic execution ordering.
    3. **Compute Units (CUs):** The stateless processing nodes that perform the actual WebAssembly executions and calculate state transitions.
*   **HyperBEAM:** An Erlang-based (OTP 27) reference implementation covering the node duties of the AO-Core protocol. Erlang was chosen precisely for its "Telecom DNA"—originally engineered for switches that could never go offline.
    *   *Concurrency:* Erlang naturally manages millions of lightweight micro-processes, obliterating the heavy OS threads or single-threaded event loops of V8/JVM/Python.
    *   *Garbage Collection:* Operates via per-process local heap isolation, eliminating "Stop-the-World" global collection spikes.
    *   *Uptime (Hot Code Reloading):* Allows live structural updates without halting the underlying framework (no restarts/downtime required).
*   **Holographic State:** A process "state" is *never* stored in a traditional database. Instead, it is "holographic": a reproducible, deterministic result derived entirely from replaying the permanent, sequential message logs stored on Arweave. Holographic state is location-independent "projected" state. It is not tied to any physical hardware; it is a mathematical certainty derived from permanent inputs.
*   **Autonomous "Cron" Capabilities:** AO processes are not purely reactive. Developers must leverage autonomous "Cron" capabilities to set up scheduled evaluations, allowing processes to actively "wake up" based on node subscriptions and network clock ticks rather than waiting for an external user trigger.
*   **Messages:** Every piece of data is a "message", interpreted as a binary term or a map of functions, strictly using HTTP semantics (RFC 9110). This foundation enables advanced native primitives like **x402 (HTTP 402 Payment Required)** for decentralized metering.
*   **Hashpaths:** The raw, unnamed schedules of inputs that define a specific point in the global compute state space.
*   **Processes:** Immutable references (names) that point to a specific, evolving Hashpath over time.

### 1.3 Nau Finance & The GiB Stablecoin
Instead of being pegged to a fiat currency like the US Dollar, the **GiB token** is a "data stablecoin" pegged exactly to the cost of 1 GiB of permanent storage on Arweave.
1. **Data Capacity vs. Monetary Value:** Because the token is pegged to storage capacity rather than dollars, its fiat value actually *decreases* over time due to the Kryder+ rate (the declining cost of physical storage hardware). This makes it highly predictable for developers buying storage and an attractive option for hedging or leverage trading.
2. **Predictable Storage Endowments:** Developers and users mint GiB by collateralizing `$AR`. They can then use GiB to pay for their permanent storage endowments. This ensures developers have a highly stable, predictable metric for their infrastructure costs rather than dealing with the extreme volatility of native `$AR` token swings.
3. **The HyperBEAM Oracle:** To guarantee the peg's accuracy, Nau Finance relies on a HyperBEAM Oracle running on the AO hyper-parallel compute network. It calculates a 6-week time-weighted moving average for the real-time price of a single byte on the Arweave network.
4. **DeFi Mechanics (No Interest & Auto-Liquidation):** Unlike traditional lending protocols like Aave, borrowers who mint GiB pay a flat, one-time fee instead of accruing compounding interest. Furthermore, because it's built on AO, Nau Finance utilizes AO's native "cron" capabilities to execute automatic liquidations without needing external keepers.

### 1.4 The Dual-Token Economy & Wallet Funding
To natively interact with the Arweave and AO ecosystems, developers and users must maintain a cryptographic wallet (typically a JSON keyfile like `wallet.json`). Separating storage from compute allows for radically targeted economic incentives:

*   **$AR (Arweave Token):** The native utility token of the base-layer Arweave network, required strictly to pay the one-time endowment for permanent data storage. It has a strict maximum total supply of `< 66 Million` tokens.
*   **$AO (AO Token):** The native token securing message passing within the AO hyper-parallel computer. Modeled on a Bitcoin-style 4-year halving cycle, it has a strictly capped maximum supply of `21 Million`. Interestingly, it features a `100% community distribution` model with absolutely zero pre-sale. 
*   **Earning $AO:** Users actively accrue $AO either by holding AR (36% of the supply is distributed to $AR holders over time) or by bridging assets like stETH or DAI to receive derivatives (e.g., aoETH).
*   **The Armstrong:** The smallest calculable unit of the AO ecosystem is "The Armstrong" (named after Joe Armstrong). Mathematically, `$10^9$ Armstrongs = 1 $AO`.
*   **The Wander Extension (ArConnect):** For frontend interaction and securing keyfiles, developers and users can use the Wander browser extension (formerly ArConnect). Rather than dangerously hardcoding raw private keys or pasting `wallet.json` contents into browser UIs, Wander acts as a secure, non-custodial signing bridge. It allows the Vanilla JS frontend to securely execute `window.arweaveWallet.sign()` transactions and interact with AO gateways natively.
*   **Onboarding Constraints:** A critical barrier for transitioning Web2 developers is the absence of a pre-funded keyfile. AI Agents generating initialization scripts **MUST** script automated checks to look for `./wallet.json` on startup. If missing or unfunded, the scaffolding must pause and explicitly interact with the developer on how to securely generate the wallet locally (e.g., via `arweave-js` or `ao-connect`) and route them to an official faucet or bridge to acquire testnet/mainnet `$AR` and `$AO` balances before attempting deployment.
* **Managed TEE Wallets:** By using the `~auth-hook@1.0` device, applications can gate trust-minimized wallets using standard Web2 authentication like cookies, OAuth, or social logins.

### 1.5 Decentralized Gateway Mesh & Trustless Verification
The network has evolved from a centralized enterprise-scale infrastructure (arweave.net) to a decentralized mesh powered by HyperBEAM.
*  **Trust-Minimized Reads:** HyperBEAM allows any node to function as a full Arweave gateway, including services for indexing, bundling, and unbundling.
*  **Holographic Verifiability:** Unlike legacy gateways that required external trust, HyperBEAM gateways serve data that is trustlessly verifiable due to the underlying properties of the AO network.
*  **Signed Contextual Metadata:** For ANS-104 DataItems, gateways now return full signed metadata with responses. This allows clients to validate the integrity of served data items directly for the first time, rather than simply trusting the serving endpoint.
*  **Egress Efficiency:** This decentralized architecture is designed to handle massive scale, noted at approximately 300 PB of monthly egress demand across the network.
* **Proof of Exclusion:** HyperBEAM gateways provide more than just inclusion proofs; because they operate within TEEs, they provide Proof of Exclusion. This ensures that a node has not censored or skipped any results in a GraphQL query, solving the 'censored indexer' problem found in legacy peer-to-peer networks.

### 1.6 Advanced Routing & Multi-Endpoint Logic
Gateway behavior has shifted from single-endpoint dependency to a configurable multi-endpoint routing system.
*  **Multicast-like Execution:** HyperBEAM supports parallel requests sent to multiple peers. Responses are then selected or aggregated based on specific operator policies.
*  **Routing Strategies:** Developers can utilize "Shuffled-Range" strategies for chunked data or multi-response semantics for transaction posts.
*  **Failover Mastery:** Node operators can now configure upstream Arweave peers and routing policies, defining exactly how many peers to query and when to stop based on performance or reliability needs.
*  **Global Deduplication:** If multiple users request a computation that shares a prior state (e.g., a specific token balance at a specific block), the system can unify these states and avoid redundant execution, drastically increasing global efficiency.

---

## 2. AI Agent Guidelines & Contribution Rules

When developing for HyperBEAM, **AI agents and developers must strictly adhere to the following rules:**

### 2.1 "The HyperBEAM Canon" (Style & Formatting)
1.  **Surgical Edits:** Always be surgical. Minimize line-of-code changes during every single edit.
2.  **Blend In:** Write code as if you were the original author.
3.  **Strict RFC 9110 Compliance:** Always use hyphens (`-`) over underscores (`_`) in binary key names (e.g., `<<"My-Header">>`, not `<<"My_Header">>`) to strictly align with HTTP Semantics.
4.  **No Waterfalls:** Avoid deeply nested `case` expressions.
5.  **Enhanced Metadata:** Add a `%%% @doc moduledoc` to every new Erlang module, and `%% @doc Description` above every new function. This explicit metadata mandate helps AI agents navigate and understand the codebase more efficiently.
6.  **Standardized Citations:** Always use a structured, consistent citation format (using bracketed standard cross-references like `[#]`) within documentation to allow LLMs to verify the source of truth immediately without hallucination.

### 2.2 AI Economic & Benchmark Standards
1.  **"Sixth Entity" AI Benchmarks:** For all autonomous agents and AI-driven processes, developers MUST implement **Consciousness Logs**. An agent's entire decision-making history, prompt evaluations, and action logic must be written as a permanent, verifiable audit trail on-chain to ensure accountability.
2.  **Do Not Break Tests:** Never submit code as 'complete' unless validated against the full test suite. 

---

## 3. Installation & System Requirements

### 3.1 System Requirements
*   **OS:** Ubuntu 22.04 LTS recommended.
*   **Hardware:** 4+ CPU cores, 8GB RAM minimum (16GB recommended for CUs), 10GB free disk space.

### 3.2 Software Dependencies
1.  **Base Tools:** `build-essential`, `cmake`, `git`, `pkg-config`, `ncurses-dev`, `libssl-dev`.
2.  **Erlang/OTP 27:** HyperBEAM strictly requires Erlang version 27.
3.  **Rebar3:** The Erlang build tool.
4.  **Rust & Cargo:** Required for certain component compilation parameters.

---

## 4. Configuration System

### 4.1 Decentralized Gateway Infrastructure
HyperBEAM utilizes a prioritized configuration system: **Environment Variables > Runtime HTTP Signatures > CLI Arguments (`erl_opts`) > Configuration File > Defaults**.

**Decentralize the Gateway:** Historically, configurations relied on centralized URLs (like `arweave.net`). Gateway logic is now hosted dynamically on the AO compute layer itself to ensure trust-minimized, decentralized data retrieval. Avoid hardcoding legacy Web2 HTTP gateways; prefer resolving gateway endpoints dynamically through the MU/CU infrastructure.

### 4.2 Configuration Parameters
*   **`config.flat`**: Only supports simple atoms, strings, integers, and booleans.
*   **`hb:start_mainnet/1`:** For complex configurations (routing, customized storage arrays), you **must** use the Erlang API `hb:start_mainnet(#{port => 10001})`.

```erlang
% Example: Utilizing a decentralized AO gateway resolution
hb:start_mainnet(#{
    http_extra_opts => #{
        store => [
            {hb_store_fs, #{dir => <<"local-storage">>}},
            {hb_store_ao_gateway, #{resolution_strategy => trust_minimized}}
        ]
    }
}).
```
### 4.3 Optimized Indexing Layer
A first-class indexing layer exists within HyperBEAM to reduce the hardware burden on gateway operators.

*  **Binary Offset Encoding:** Introduced in PR675, a new compressed storage format for Arweave data item indexes uses binary encoding (via hb_store_arweave_offset.erl).
*  **Storage Reduction:** Each index key-value pair has been reduced from approximately 90 bytes of encoded ASCII to a compact $32+\sim11$ bytes.
*  **Stateless Gateways:** Operators can run "effectively stateless" nodes by using an offset index as the primary persistent state and streaming data on demand. This reduces total storage requirements for operators by approximately 50%.
*  **Header Exclusion:** Indexers may choose to discard Arweave block headers after the indexing process to further optimize disk usage.
---

## 6. HyperBEAM Architecture: Devices & HyperPATHs

HyperBEAM exposes an HTTP API (HyperPATHs) utilizing RFC-9421 HTTP Message Signatures for authentication. Every interaction is done by sending HTTP requests to specific device endpoints.

*   **`~relay@1.0`**: Acts as the ingress router from the wider HTTP network into the native AO ecosystem.
*   **`~scheduler@1.0`**: Assigns a linear hashpath to an execution for deterministic ordering (SU logic).
*   **`~process@1.0`**: Creates and manages holographic shared execution states where users add inputs to a hashpath.
*   **`~wasm64@1.0`**: Executes WebAssembly code via WAMR.
*   **`~patch@1.0`**: Exposes process state directly via GET requests, permanently replacing legacy "dryrun" simulations.
*   **`p4@1.0` / `~simple-pay@1.0`**: Underpinning the x402 standards, these frameworks allow node operators to price compute usage. They handle the cryptographic token-handshakes and integrate natively with the GiB token standard.
*  **`~copycat@1.0/arweave`**: A specialized device for indexing transactions and bundled items into the offset index. It can be triggered to index from newest to oldest records.
*  **`GET /~arweave@2.9/tx=TX_ID`**: returns the full message including parsed fields.
*  **`GET /~arweave@2.9/raw=TX_ID`**: provides a legacy-compatible endpoint for raw data retrieval.
*  **`~cron@1.0`:** Used to trigger one-time or scheduled indexing tasks (e.g., `/once?cron-path=~copycat@1.0/arweave`).
*  **`~query@1.0`**: The interface for executing trust-minimized GraphQL queries.
*  **`~snip@1.0`**: The "Secure Nested Paging" device used to verify the hardware and firmware integrity of a remote TEE node.
*  **`auth-hook@1.0`**: Used for mapping Web2 authentication (cookies/OAuth) to AO process interactions. In the context of payments, this device can be used to verify that a user has a valid payment session or "allowance" before routing the request to the underlying compute unit.
* **`p4@1.0`**: This is the primary framework that underpins the x402 payment protocol standards. It allows node operators to price compute usage and handle the coordination of payments.
* **`~simple-pay@1.0`**: This device works alongside p4 to handle the actual economic transactions. It integrates natively with the GiB token standard and manages the cryptographic handshakes required to unlock an endpoint.

---

## 7. Testing, Debugging, & Profiling

### 7.1 Running Tests
*   **Run all tests:** `rebar3 eunit`.

### 7.2 Debug Logging
Use `?event(term())` in code to write debug prints to the CLI, visible by setting the `HB_PRINT` environment variable.

### 7.3 Troubleshooting & CU Memory Tuning
*   **Memory Tuning for CUs:** When operating Compute Units (CUs) dedicated to complex executions, especially intensive AI inference tasks required by "Sixth Entity" operations, developers must proactively configure WebAssembly container limits. Adjust the `PROCESS_WASM_MEMORY_MAX_LIMIT` environment variable in the node configuration to allocate sufficient RAM headroom preventing out-of-memory (OOM) crashes during peak parallel or LLM processing loads.

### 7.4 Performance Tuning: Chunk Retrieval
*  **Direct Serving:** HyperBEAM is optimized to serve chunked data directly from Arweave nodes, eliminating the need for large, expensive local caches.
*  **Parallel Gap-Filling:** The system utilizes parallel chunk range fetching and "gap-fill" logic to maintain high performance.
*  **Logic Location:** These operations are handled within dev_arweave.erl (specifically around lines 188 and 251).

---

## 8. AOS SQLite module
The AOS SQLite module provides a full SQLite database that is natively embedded directly into AO contracts

*  **In-WASM Execution:** Because AO processes operate within WebAssembly (WASM), the SQLite database is compiled and runs directly inside an isolated AO ~process@1.0 environment.
*  **On-Chain State Memory:** It functions as a live, on-chain memory snapshot for applications, enabling structured data management and decentralized, trustless querying.
*  **Message-Driven Execution:** When a Compute Unit (CU) receives an interaction message, it executes standard SQL commands (such as INSERT or SELECT) directly within its isolated memory space to process state changes.
* **Cost-Efficient Querying:** Users can interact with the SQLite state using "dry-run" messages via Compute Units. This allows them to execute queries and read data without writing to the chain, meaning they do not incur permanent Arweave storage fees.
* **Simple Implementation:** It is ideal for straightforward application logic that requires structured data. For example, you can deploy a script that accepts a user's name and score, inserts it into a single embedded SQLite table, and returns the top entries.

### 8.1 Using the AOS SQLite module
To use the embedded AOS SQLite capability, you must interact with the AO process hosting it. In the HyperBEAM architecture, every interaction is treated as an HTTP request sent to specific endpoints called "devices" via HyperPATHs.

Because the SQLite database runs inside an isolated WebAssembly process, you do not use traditional SQL connection strings. Instead, you send messages (which contain your SQL commands or application logic) to its devices and the Compute Unit (CU) executes them.

*  **Writing Data (State Changes):** To insert or update data in your SQLite database, you send a message to the process through a Messenger Unit (MU). This message is assigned a slot by the `~scheduler@1.0` device and permanently stored on Arweave, ensuring your database state is deterministic and reproducible.
*  **Reading Data (Cost-Free Queries):** To execute SELECT queries without paying Arweave storage fees, you use the CU's `dry-run` endpoint. The CU will load the process's memory snapshot, execute your read query against the SQLite database within its memory, and return the result directly to you without logging the transaction on-chain.
*  **Frontend Integration:** If you are building a user interface for your application, you can easily trigger these device calls using the aoconnect SDK. Use the `dryrun` function to execute cost-free SQLite queries through your local or preferred CU. Use the `message` or `spawn` functions to write new data to the database. The SDK will automatically utilize the default MU to save the message to Arweave.
