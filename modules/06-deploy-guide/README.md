# Module 06 — Deploy Guide

> Step-by-step deployment to Stellar Futurenet and Testnet.

## Overview

```
Write Rust → Build WASM → Deploy to Network → Invoke Functions → Verify on Explorer
```

## Step 1: Build Your Contract

```bash
cd your-contract-directory

# Build
stellar contract build
# This runs: cargo build --target wasm32-unknown-unknown --release

# Output file:
# target/wasm32-unknown-unknown/release/your_contract.wasm
```

### Optimize WASM (recommended for production)
```bash
stellar contract optimize \
  --wasm target/wasm32-unknown-unknown/release/your_contract.wasm
# Output: target/wasm32-unknown-unknown/release/your_contract.optimized.wasm
```

> **Size limit**: WASM must be under 64KB. Optimization typically reduces size by 40-60%.

## Step 2: Create & Fund an Identity

### Option A: CLI-managed identity (recommended for students)
```bash
# Generate identity + auto-fund on futurenet
stellar keys generate student --network futurenet --fund

# Check your address
stellar keys address student
```

### Option B: Existing secret key
```bash
# Import existing key
stellar keys add student --secret-key
# Paste your secret key when prompted

# Fund via Friendbot
curl "https://friendbot-futurenet.stellar.org/?addr=$(stellar keys address student)"
```

### Option C: For testnet
```bash
stellar keys generate student --network testnet --fund
```

## Step 3: Add Network (one-time setup)

```bash
# Futurenet
stellar network add futurenet \
  --rpc-url https://rpc-futurenet.stellar.org \
  --network-passphrase "Test SDF Future Network ; October 2022"

# Testnet
stellar network add testnet \
  --rpc-url https://soroban-testnet.stellar.org \
  --network-passphrase "Test SDF Network ; September 2015"
```

## Step 4: Deploy

```bash
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/your_contract.wasm \
  --source-account student \
  --network futurenet

# Output: CAAAAAAA... (your CONTRACT_ID)
# SAVE THIS! You need it for every invocation.
```

### Deploy with an alias (easier to remember)
```bash
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/your_contract.wasm \
  --source-account student \
  --network futurenet \
  --alias my-contract
```

## Step 5: Invoke Functions

```bash
# Call a function
stellar contract invoke \
  --id <CONTRACT_ID> \
  --source-account student \
  --network futurenet \
  -- \
  function_name \
  --arg1 value1 \
  --arg2 value2
```

> **Important**: The `--` separator is required. Everything after it is contract arguments.

### Examples

```bash
# Hello world
stellar contract invoke \
  --id CABC123... \
  --source-account student \
  --network futurenet \
  -- hello --to World

# Set a value
stellar contract invoke \
  --id CABC123... \
  --source-account student \
  --network futurenet \
  -- set_value --key "test" --value 42

# Read a value (no source needed for read-only)
stellar contract invoke \
  --id CABC123... \
  --network futurenet \
  -- get_value --key "test"
```

## Step 6: Verify on Block Explorer

1. Go to [Stellar Expert](https://stellar.expert/explorer/futurenet)
2. Paste your contract ID or transaction hash
3. You should see your contract and its transactions

For testnet: [stellar.expert/explorer/testnet](https://stellar.expert/explorer/testnet)

## Quick Reference: Complete Flow

```bash
# 1. Build
stellar contract build

# 2. Generate + fund identity
stellar keys generate student --network futurenet --fund

# 3. Deploy
CONTRACT_ID=$(stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/your_contract.wasm \
  --source-account student \
  --network futurenet)

echo "Deployed: $CONTRACT_ID"

# 4. Invoke
stellar contract invoke \
  --id $CONTRACT_ID \
  --source-account student \
  --network futurenet \
  -- your_function --your_arg "value"

# 5. Verify
echo "View on explorer: https://stellar.expert/explorer/futurenet/contract/$CONTRACT_ID"
```

## Network Comparison

| | Futurenet | Testnet | Mainnet |
|---|-----------|---------|---------|
| **Purpose** | Experimental / learning | Pre-production testing | Real money |
| **Stability** | May reset | Stable | Permanent |
| **Free tokens** | Yes (Friendbot) | Yes (Friendbot) | No |
| **Use for** | Bootcamp, learning | Staging, testing | Production |
| **RPC URL** | `rpc-futurenet.stellar.org` | `soroban-testnet.stellar.org` | `mainnet.sorobanrpc.com` |
| **Explorer** | stellar.expert/futurenet | stellar.expert/testnet | stellar.expert/public |

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `Account not found` | Identity not funded | Run Friendbot again |
| `HostError: Error(Storage, MissingValue)` | Wrong contract ID | Double-check CONTRACT_ID |
| `Transaction simulation failed` | Bug in contract logic | Test locally with `cargo test` first |
| `Wasm too large` | WASM > 64KB | Run `stellar contract optimize` |
| `op_underfunded` | Not enough XLM for fees | Fund your account again |
| `tx_bad_seq` | Sequence number mismatch | Wait a moment, retry |
| `connection refused` | RPC server down | Try again or use different RPC |

## Next Steps

- [Module 07 — Common Errors](../07-common-errors/) for a comprehensive error guide
- [Module 09 — Tools & Inspection](../09-tools-and-inspection/) to debug deployed contracts
