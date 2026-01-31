# EVM Wallet Skill

**Status:** ✅ Built (Phase 1 - Core + Read-only)

Self-sovereign EVM wallet for Clawdbot. Private key stored locally, no external API dependencies for core operations.

## Overview

This skill provides a complete EVM wallet implementation with support for:
- ✅ Wallet generation and secure storage
- ✅ Balance checking (ETH + ERC20 tokens)
- ✅ Native token transfers (ETH)
- ✅ ERC20 token transfers
- ✅ Generic contract interactions
- ✅ Token swaps via Odos aggregator (best-route DEX aggregation)
- ✅ Smart gas estimation (EIP-1559)
- ✅ Multi-chain support (Base, Ethereum, Polygon, Arbitrum, Optimism)

## Supported Chains

| Chain    | ChainId | Native Token | Gas Fees |
|----------|---------|------------- |----------|
| Base     | 8453    | ETH          | Very Low |
| Ethereum | 1       | ETH          | High     |
| Polygon  | 137     | POL          | Low      |
| Arbitrum | 42161   | ETH          | Low      |
| Optimism | 10      | ETH          | Low      |

**Recommendation:** Start with Base for testing - lowest gas fees!

## Quick Start

```bash
# 1. Generate wallet
node src/setup.js

# 2. Check balance (should be 0)
node src/balance.js base

# 3. Fund your wallet by sending ETH to the address
# 4. Check balance again
node src/balance.js base
```

## Scripts

### Setup
```bash
node src/setup.js                    # Generate new wallet
node src/setup.js --force            # Overwrite existing wallet
node src/setup.js --json             # JSON output
```

### Balance Checking
```bash
node src/balance.js base                           # ETH balance on Base
node src/balance.js ethereum 0x833589fcd6edb...    # USDC balance on Ethereum
node src/balance.js --all                          # All chains, native tokens
node src/balance.js --all --json                   # JSON output
```

### Transfers
```bash
# Native ETH transfers
node src/transfer.js base 0x123... 0.01

# ERC20 token transfers
node src/transfer.js base 0x123... 100 0x833589fcd6edb...

# Skip confirmation
node src/transfer.js base 0x123... 0.01 --yes
```

### Contract Interactions
```bash
# Read operations (view/pure functions)
node src/contract.js base 0x833589fcd... "balanceOf(address)" 0x123...
node src/contract.js ethereum 0x123... "symbol()"

# Write operations (state-changing)
node src/contract.js base 0x833589fcd... "transfer(address,uint256)" 0x123... 1000000
node src/contract.js base 0x456... "approve(address,uint256)" 0x123... 1000000000

# Payable functions
node src/contract.js ethereum 0x789... "deposit()" --value 0.1
```

### Token Swaps (Odos Aggregator)
```bash
# Swap ETH → USDC on Base
node src/swap.js base eth 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 0.01

# Swap USDC → ETH on Base
node src/swap.js base 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 eth 100

# Custom slippage (1%)
node src/swap.js base eth 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 0.01 --slippage 1

# Quote only (no execution)
node src/swap.js base eth 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 0.01 --quote-only

# Skip confirmation
node src/swap.js base eth 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 0.01 --yes

# JSON output
node src/swap.js base eth 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 0.01 --json
```

## Security Features

- 🔒 Private key stored locally with 600 permissions
- ⚠️  Transaction confirmation prompts (unless --yes flag)
- 💰 Balance validation before transfers
- ⛽ Smart gas estimation with safety margins
- 🔍 Explorer links for all transactions

## Architecture

### Libraries (`src/lib/`)
- **chains.js** - Chain configurations and metadata
- **rpc.js** - RPC management with automatic failover
- **wallet.js** - Wallet state management
- **gas.js** - EIP-1559 gas estimation

### Wallet Storage
- **`~/.evm-wallet.json`** — Wallet keys stored in user's home directory (chmod 600, never in project)

### Key Features
- **Multi-RPC Failover:** Automatic rotation between public RPCs
- **Smart Gas:** EIP-1559 with base fee + priority fee analysis
- **Type Safety:** Full TypeScript support via viem
- **Error Handling:** Comprehensive validation and user-friendly errors

## Future Enhancements (Not Built Yet)
- 🏦 DeFi protocol integrations
- 📊 Portfolio tracking and analytics
- 🔐 Hardware wallet support
- 🌐 ENS domain resolution
- 💾 Transaction history storage

## Technical Stack

- **Runtime:** Node.js
- **EVM Library:** viem (modern, lightweight, typed)
- **DEX Aggregator:** Odos (multi-hop, multi-source routing)
- **Chains:** Public RPC endpoints with automatic failover
- **Gas:** Smart EIP-1559 estimation
- **Security:** Local key storage, confirmation prompts

## Common Token Addresses

### Base
- **USDC:** `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`
- **WETH:** `0x4200000000000000000000000000000000000006`

### Ethereum
- **USDC:** `0xA0b86a33E6441b8a46a59DE4c4C5E8F5a6a7A8d0`
- **WETH:** `0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2`

## Example Workflows

### Check Portfolio
```bash
# Quick overview
node src/balance.js --all

# Specific token balances
node src/balance.js base 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913  # USDC on Base
```

### Send Payment
```bash
# Send 0.01 ETH on Base (cheapest fees)
node src/transfer.js base 0x123... 0.01

# Send 100 USDC on Base
node src/transfer.js base 0x123... 100 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913
```

### DeFi Interactions
```bash
# Approve USDC spending
node src/contract.js base 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 "approve(address,uint256)" 0xSpender... 1000000000

# Check allowance
node src/contract.js base 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 "allowance(address,address)" 0xOwner... 0xSpender...
```

---

**⚠️ Security Notice:** This is a real wallet with real funds. Always:
- Test with small amounts first
- Use Base for testing (lowest fees)
- Verify recipient addresses
- Keep your `~/.evm-wallet.json` file secure
- Back up your wallet file!