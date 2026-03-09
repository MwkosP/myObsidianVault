


# Bittensor — btcli (Command Line Interface)

---

## Overview

`btcli` is the official command line tool for interacting with the Bittensor network. Use it to manage wallets, stake TAO, register on subnets, inspect the network state, and participate in governance. Everything you can do on Bittensor can be done through btcli.

**Config file location:** `~/.bittensor/config.yml` **Debug log location:** `~/.bittensor/debug.txt`

> Always get btcli directly from official docs or GitHub. Malicious lookalikes exist.

---

## Installation

```bash
# Via pip (recommended)
pip install bittensor-cli

# Via Homebrew (macOS)
brew install bittensor-cli

# From source
git clone https://github.com/opentensor/btcli
cd btcli
python3 -m venv btcli_venv
source btcli_venv/bin/activate
pip install .

# Verify installation
btcli --version
```

**Requirements:** Python 3.9–3.12. Windows users must use WSL 2. Mining and validating are not supported on Windows — only wallet operations work on WSL.

---

## Global Options

```bash
btcli --version        # Show installed version
btcli --commands       # List all commands
btcli --debug          # Save debug log from last command
btcli --help           # Help for any command
```

Add `--help` to any command for detailed options:

```bash
btcli wallet --help
btcli stake add --help
```

---

## Networks

|Network|What it is|
|---|---|
|`finney`|Mainnet — real TAO, real consequences|
|`test`|Testnet — for development, no real TAO|
|`local`|Local node — for development|

Specify network per command:

```bash
btcli wallet balance --network finney
btcli subnets list --network test
```

Or set it once in config:

```bash
btcli config set --network finney
```

---

## 1. Config Commands

```bash
btcli config set                                    # Interactive config setup
btcli config set --wallet-name myWallet             # Set default wallet
btcli config set --network finney                   # Set default network
btcli config set --hotkey myHotkey                  # Set default hotkey
btcli config set --safe-staking --rate-tolerance 0.1 # Enable safe staking with 0.1% tolerance
btcli config get                                    # View current config
btcli config clear --all                            # Reset entire config
btcli config clear --chain --network                # Clear specific fields
```

---

## 2. Wallet Commands (`btcli wallet` / `btcli w`)

### Create & Manage Keys

```bash
btcli wallet create                                 # Create full wallet (coldkey + hotkey)
btcli wallet new-coldkey                            # Create new coldkey only
btcli wallet new-hotkey                             # Create new hotkey only
btcli wallet list                                   # List all local wallets and hotkeys
```

### Regenerate Keys (Recovery)

```bash
btcli wallet regen-coldkey --mnemonic "word1 word2 ... word12"   # Recover coldkey from mnemonic
btcli wallet regen-hotkey --mnemonic "word1 word2 ... word12"    # Recover hotkey from mnemonic
btcli wallet regen-coldkeypub --ss58 <address>                   # Recover public coldkey only
```

> Use `regen-coldkeypub` when moving to a new machine for mining — you only need the public key on the server, not the full coldkey.

### View Wallet Info

```bash
btcli wallet balance                                # Check TAO balance
btcli wallet balance --wallet-name myWallet        # Check specific wallet balance
btcli wallet overview                              # Full overview of registered accounts
btcli wallet overview --netuid 1                   # Overview for specific subnet
```

### Transfer

```bash
btcli wallet transfer --dest <ss58_address> --amount 10    # Send 10 TAO to address
```

### Identity

```bash
btcli wallet set-identity                          # Create/update on-chain identity
btcli wallet get-identity --key <ss58_address>     # View identity for any address
```

### Key Management

```bash
# Swap hotkey (costs 1 TAO — fee burned)
btcli wallet swap-hotkey new_hotkey_name --wallet-name myWallet --hotkey old_hotkey

# Schedule coldkey swap (security migration)
btcli wallet swap-coldkey --new-wallet my_new_wallet

# Check status of coldkey swap
btcli wallet swap-check --wallet-name myWallet
btcli wallet swap-check --all                     # Check all pending swaps
```

### Sign & Verify

```bash
btcli wallet sign --wallet-name myWallet --message "hello"    # Sign a message
btcli wallet verify --message "hello" --signature <sig> --address <ss58>  # Verify signature
```

---

## 3. Stake Commands (`btcli stake` / `btcli st`)

### Add / Remove Stake

```bash
btcli stake add --wallet-name myWallet --hotkey myHotkey --amount 100    # Stake 100 TAO
btcli stake add --all-hotkeys                                             # Stake to all hotkeys
btcli stake remove --wallet-name myWallet --hotkey myHotkey --amount 50  # Unstake 50 TAO
btcli stake remove --unstake-all                                          # Remove all stake
```

### View Stake

```bash
btcli stake show                                   # Show all stake accounts
btcli stake show --wallet-name myWallet            # Show stake for specific wallet
btcli stake show --all                             # Show stake across all wallets
```

Output shows: coldkey, balance, hotkey/delegate, amount staked, daily TAO rate of return.

### Child Hotkeys (Stake Delegation to Children)

```bash
# Delegate stake to child hotkeys on a subnet
btcli stake set-children \
  --children <child_hotkey1>,<child_hotkey2> \
  --hotkey <parent_hotkey> \
  --netuid 1 \
  --proportions 0.4,0.6           # Must sum to 1.0

# Get childkey take rate
btcli stake get-childkey-take --hotkey <hotkey> --netuid 1

# Set childkey take rate (0–18%)
btcli stake set-childkey-take --hotkey <hotkey> --netuid 1 --take 0.1
```

---

## 4. Subnet Commands (`btcli subnets` / `btcli s`)

### View Subnets

```bash
btcli subnets list                                 # List all active subnets with emissions
btcli subnets metagraph --netuid 1                 # Full metagraph for subnet 1
btcli subnets metagraph --netuid 1 --reuse-last    # Reuse cached metagraph (faster)
```

Metagraph columns include: UID, HOTKEY, COLDKEY, STAKE, RANK, TRUST, CONSENSUS, INCENTIVE, DIVIDENDS, EMISSION, VTRUST, ACTIVE, AXON, UPDATED.

### Register on a Subnet

```bash
btcli subnets register --netuid 1 --wallet-name myWallet --hotkey myHotkey
```

> This burns TAO. Cost varies by subnet demand.

### Create a Subnet

```bash
btcli subnets create --wallet-name myWallet
```

> Registration fee scales with number of existing subnets. Fee is burned.

### Subnet Info

```bash
btcli subnets show --netuid 1                      # Show hyperparameters for subnet 1
btcli subnets lock-cost                            # Check current subnet creation cost
```

---

## 5. Weights Commands (`btcli weights` / `btcli wt`)

```bash
btcli weights reveal \
  --netuid 1 \
  --uids 1,2,3,4 \
  --weights 0.1,0.2,0.3,0.4 \
  --salt 163,241,217,11,161,142,147,189     # Reveal committed weights

btcli weights commit \
  --netuid 1 \
  --uids 1,2,3,4 \
  --weights 0.1,0.2,0.3,0.4               # Commit weights (before reveal)
```

> Bittensor uses a commit-reveal scheme for weights. Validators first commit a hash of their weights, then reveal the actual values later. This prevents frontrunning.

---

## 6. Delegation Commands

```bash
btcli stake add --delegate <hotkey_ss58> --amount 100    # Delegate TAO to a validator
btcli stake remove --delegate <hotkey_ss58> --amount 50  # Undelegate TAO

btcli wallet list-delegates                              # Show all available delegates
btcli wallet my-delegates                                # Show your current delegations
```

### Become a Delegate (Validators)

```bash
btcli wallet nominate --wallet-name myWallet --hotkey myHotkey   # Register as delegate

# Set your take rate (0–18%)
btcli wallet set-take --hotkey myHotkey --take 0.1
```

---

## 7. Root Network Commands (`btcli root`)

```bash
btcli root list                                    # List root network neurons (top validators)
btcli root weights                                 # View root network weights (subnet rankings)
btcli root set-weights \
  --netuids 1,2,3 \
  --weights 0.5,0.3,0.2                            # Set root weights (SN0 validators only)
btcli root boost --netuid 1 --amount 0.1           # Boost a subnet's root weight
btcli root senate-vote --proposal <hash> --vote yes  # Vote on governance proposal
btcli root proposals                               # View active governance proposals
```

---

## 8. Sudo Commands (`btcli sudo` / `btcli su`)

These are admin commands for subnet owners to set hyperparameters:

```bash
btcli sudo set --netuid 1 --param max_n --value 256           # Set max neurons
btcli sudo set --netuid 1 --param tempo --value 100           # Set tempo (blocks per epoch)
btcli sudo set --netuid 1 --param min_allowed_weights --value 8
btcli sudo set --netuid 1 --param immunity_period --value 5000
btcli sudo get --netuid 1                                      # View all hyperparameters
```

---

## 9. View Dashboard

```bash
btcli view dashboard    # Opens HTML dashboard in browser
                        # Shows subnets, stake, and neuron info visually
```

---

## Key Concepts for CLI Use

### Wallet flags used in most commands

|Flag|What it sets|
|---|---|
|`--wallet-name`|Your coldkey wallet name|
|`--hotkey`|Your hotkey name|
|`--netuid`|Subnet ID (e.g. `--netuid 1`)|
|`--network`|Network (finney / test / local)|
|`--no-prompt` / `-y`|Skip confirmation prompts|
|`--quiet`|Suppress verbose output|
|`--verbose`|Show extra detail|
|`--json-output`|Output results as JSON|

### Common flags pattern

Almost every state-changing command follows:

```bash
btcli <command> --wallet-name <name> --hotkey <hotkey> --netuid <id> --network finney
```

---

## Common Workflows

### New miner setup

```bash
btcli wallet create                               # 1. Create wallet
btcli wallet new-hotkey                           # 2. Create hotkey for mining
btcli subnets register --netuid <id>              # 3. Register on target subnet
# → Start your miner script pointing to hotkey
```

### New validator setup

```bash
btcli wallet create                               # 1. Create wallet
btcli wallet new-hotkey                           # 2. Create hotkey
btcli stake add --amount <TAO>                    # 3. Add stake (must meet min_stake)
btcli subnets register --netuid <id>              # 4. Register on subnet
btcli wallet nominate                             # 5. Optionally become delegate
# → Start validator script, begin submitting weights
```

### Delegate TAO (passive yield)

```bash
btcli wallet list-delegates                       # 1. Find validators to delegate to
btcli stake add --delegate <hotkey> --amount 100  # 2. Delegate TAO
btcli stake show                                  # 3. Monitor yield
```

### Check subnet state

```bash
btcli subnets list                                # Overview of all subnets
btcli subnets metagraph --netuid 1                # Drill into subnet 1
```

---

## Important Notes

- **Transaction fees apply** to most state-changing operations. Check fees before running.
- **Registration fees are burned** — non-refundable. Confirm before registering.
- **Coldkey swap has a delay** (several days) — plan ahead for security migrations.
- **Safe staking mode** (`--safe-staking`) adds a rate tolerance check — protects against unexpected TAO rate changes during transactions.
- **Commit-reveal for weights** — validators must commit weights first, then reveal in a later block. This is enforced at the protocol level.
- **Windows:** Only wallet/transfer/stake operations work on WSL. Mining and validating require Linux/macOS.