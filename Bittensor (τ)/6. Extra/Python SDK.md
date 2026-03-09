



# Bittensor — Python SDK

---

## Overview

The Bittensor Python SDK is the programmatic interface for everything btcli does on the command line — plus much more. It is what you use to actually **build miners, validators, and subnet incentive mechanisms**. While btcli is for operators, the SDK is for developers.

```
pip install bittensor
```

**Requirements:** Python 3.9–3.12

**Current version:** SDK v10 (PascalCase imports — see note below)

> SDK v10 changed all imports to PascalCase. `bittensor.subtensor` → `bittensor.Subtensor`. Code using the old style will break.

---

## Core Classes — The Six Primitives

Everything in the SDK revolves around six core classes:

| Class       | What it is                               | Brain analogy                   |
| ----------- | ---------------------------------------- | ------------------------------- |
| `Subtensor` | Gateway to the blockchain                | The spine — connects everything |
| `Wallet`    | Manages coldkey + hotkey                 | Your identity                   |
| `Metagraph` | Snapshot of subnet state                 | The map of the network          |
| `Axon`      | Server — receives queries (miners)       | Dendrite receiver in biology    |
| `Dendrite`  | Client — sends queries (validators)      | Axon transmitter in biology     |
| `Synapse`   | Data object passed between Axon/Dendrite | The signal itself               |

---

## Installation & Import

```python
# Install
pip install bittensor

# SDK v10 imports (PascalCase — required)
from bittensor import Subtensor, Wallet, Metagraph, Axon, Dendrite, Synapse
from bittensor import AsyncSubtensor  # async version

# Old style (v9 and below — REMOVED in v10)
# from bittensor import subtensor, wallet  ← breaks in v10
```

---

## 1. Subtensor — Blockchain Gateway

`Subtensor` is your connection to the Bittensor blockchain (Subtensor chain). All on-chain reads and writes go through here.

### Connecting

```python
from bittensor import Subtensor

# Connect to mainnet
sub = Subtensor(network='finney')

# Connect to testnet
sub = Subtensor(network='test')

# Connect to local node
sub = Subtensor(network='local')

# Connect via websocket URL directly
sub = Subtensor(network='ws://127.0.0.1:9944')
```

### Reading Chain State

```python
# Get current block number
block = sub.get_current_block()

# Get metagraph for subnet 1
metagraph = sub.metagraph(netuid=1)

# Get metagraph for specific mechanism (SDK v10 multi-mechanism support)
metagraph = sub.metagraph(netuid=1, mechid=0)

# Get neuron info for a specific UID
neuron = sub.neuron_for_uid(uid=5, netuid=1)

# Get all neurons in a subnet
neurons = sub.neurons(netuid=1)

# Get stake for a hotkey
stake = sub.get_stake_for_coldkey_and_hotkey(
    coldkey_ss58=wallet.coldkeypub.ss58_address,
    hotkey_ss58=wallet.hotkey.ss58_address
)

# Get subnet hyperparameters
params = sub.get_subnet_hyperparameters(netuid=1)

# List all subnets
subnets = sub.get_all_subnet_netuids()

# Get total TAO supply
total_supply = sub.total_issuance()
```

### Writing to Chain (Extrinsics)

```python
wallet = Wallet(name='myWallet', hotkey='myHotkey')

# Register a neuron on a subnet (burns TAO)
success = sub.register(wallet=wallet, netuid=1)

# Set weights (validator core operation)
success = sub.set_weights(
    wallet=wallet,
    netuid=1,
    uids=[0, 1, 2, 3],
    weights=[0.4, 0.3, 0.2, 0.1],  # must sum to 1
    wait_for_inclusion=True,
    wait_for_finalization=False
)

# Set weights for specific mechanism (v10)
success = sub.set_weights(wallet, netuid=1, uids=uids, weights=weights, mechid=0)

# Add stake
success = sub.add_stake(
    wallet=wallet,
    hotkey_ss58=wallet.hotkey.ss58_address,
    amount=100.0,  # TAO
    safe_staking=True,
    rate_tolerance=0.005  # max 0.5% price change allowed
)

# Remove stake
success = sub.unstake(
    wallet=wallet,
    hotkey_ss58=wallet.hotkey.ss58_address,
    amount=50.0
)

# Transfer TAO
success = sub.transfer(
    wallet=wallet,
    dest='<ss58_destination_address>',
    amount=10.0
)

# Register as delegate (validator)
success = sub.nominate(wallet=wallet)

# Delegate TAO to another validator
success = sub.delegate(
    wallet=wallet,
    delegate_ss58='<validator_hotkey_ss58>',
    amount=100.0
)

# Serve axon (advertise miner endpoint on chain)
success = sub.serve_axon(netuid=1, axon=my_axon)
```

### Async Subtensor

```python
import asyncio
from bittensor import AsyncSubtensor

async def main():
    async with AsyncSubtensor(network='finney') as sub:
        block = await sub.get_current_block()
        metagraph = await sub.metagraph(netuid=1)

asyncio.run(main())
```

---

## 2. Wallet — Keys & Identity

```python
from bittensor import Wallet

# Load existing wallet
wallet = Wallet(name='myWallet', hotkey='myHotkey')

# Create new wallet (also creates files on disk)
wallet = Wallet(name='newWallet')
wallet.create_new_coldkey(n_words=12, use_password=True)
wallet.create_new_hotkey(n_words=12, use_password=False)

# Access keys
wallet.coldkey              # Full coldkey (requires decryption)
wallet.coldkeypub           # Public coldkey (always available)
wallet.hotkey               # Hotkey keypair
wallet.hotkey.ss58_address  # Hotkey SS58 address string

# Check if keys exist
wallet.coldkey_file.exists_on_device()
wallet.hotkey_file.exists_on_device()

# Regenerate from mnemonic
wallet.regenerate_coldkey(mnemonic="word1 word2 ... word12")
wallet.regenerate_hotkey(mnemonic="word1 word2 ... word12")
```

### Coldkey vs Hotkey reminder

||Coldkey|Hotkey|
|---|---|---|
|Holds TAO|Yes|No|
|Used for|Staking, transfers, high-value ops|Mining, validating, signing queries|
|Should be|Encrypted, offline if possible|On the running server|
|Risk if compromised|Full fund loss|Operational disruption only|

---

## 3. Metagraph — Subnet State

The Metagraph is a snapshot of everything happening in a subnet at a given block — every neuron's stake, rank, trust, incentive, emission, weights, and axon endpoint.

```python
from bittensor import Metagraph, Subtensor

# Basic — lite mode (fast, no weights/bonds)
metagraph = Metagraph(netuid=1, network='finney', lite=True)

# Full mode — includes weights W and bonds B matrices
metagraph = Metagraph(netuid=1, network='finney', lite=False)

# Sync to latest block
sub = Subtensor(network='finney')
metagraph.sync(subtensor=sub)

# Sync to specific historical block
metagraph.sync(block=12345, lite=False, subtensor=sub)

# For historical data beyond 300 blocks — use archive node
sub_archive = Subtensor(network='archive')
metagraph.sync(block=old_block, subtensor=sub_archive)
```

### Metagraph Attributes

```python
metagraph.n              # Number of neurons (int)
metagraph.block          # Current block number
metagraph.uids           # Array of UIDs [0, 1, 2, ...]
metagraph.hotkeys        # List of hotkey SS58 addresses
metagraph.coldkeys       # List of coldkey SS58 addresses
metagraph.axons          # List of AxonInfo objects (IP, port, etc.)

# Tensors (one value per neuron)
metagraph.S              # Stake vector
metagraph.R              # Rank vector
metagraph.T              # Trust vector
metagraph.C              # Consensus vector
metagraph.I              # Incentive vector
metagraph.E              # Emission vector
metagraph.D              # Dividends (validator earnings)
metagraph.V              # Validator trust (vtrust)
metagraph.active         # Active status per neuron

# Matrices (only in lite=False mode)
metagraph.W              # Weight matrix [n x n]
metagraph.B              # Bond matrix [n x n]

# Quick analysis examples
total_stake = metagraph.S.sum()
top_miners = metagraph.I.argsort()[-10:]        # Top 10 by incentive
trusted_validators = metagraph.V > 0.5          # High vtrust validators
```

### Async Metagraph

```python
import asyncio
from bittensor import AsyncSubtensor
from bittensor.core.metagraph import async_metagraph

async def main():
    async with AsyncSubtensor(network='finney') as sub:
        metagraph = await async_metagraph(
            netuid=1,
            network='finney',
            lite=False,
            subtensor=sub
        )
        print(f"Total stake: {metagraph.S.sum():.2f}")

asyncio.run(main())
```

---

## 4. Axon — Miner Server

Axon is the **server** that miners run. It uses FastAPI under the hood and listens for incoming Synapse requests from validators.

```python
from bittensor import Axon, Wallet, Synapse

wallet = Wallet(name='myWallet', hotkey='myHotkey')

# Create axon server
axon = Axon(
    wallet=wallet,
    port=8091,
    ip='0.0.0.0',
    external_ip='<your_public_ip>',
    external_port=8091
)
```

### Attaching Handlers

```python
# Define your custom synapse type
class MySynapse(Synapse):
    input: str = ''
    output: str = None

# Forward function — processes the request, returns synapse
def forward(synapse: MySynapse) -> MySynapse:
    synapse.output = f"Processed: {synapse.input}"
    return synapse

# Blacklist function — return True to reject request
def blacklist(synapse: MySynapse) -> tuple[bool, str]:
    if synapse.dendrite.hotkey not in allowed_validators:
        return True, "Not a registered validator"
    return False, "Allowed"

# Priority function — higher = processed first
def priority(synapse: MySynapse) -> float:
    return 1.0  # Can be based on validator stake

# Attach all handlers
axon.attach(
    forward_fn=forward,
    blacklist_fn=blacklist,
    priority_fn=priority
)

# Start serving
axon.start()

# Serve on-chain (advertises endpoint to validators)
sub = Subtensor(network='finney')
sub.serve_axon(netuid=1, axon=axon)

# Stop server
axon.stop()
```

---

## 5. Dendrite — Validator Client

Dendrite is the **client** that validators use to query miners. It sends Synapse objects to Axon servers and collects responses.

```python
from bittensor import Dendrite, Wallet, Metagraph, Subtensor

wallet = Wallet(name='myWallet', hotkey='myHotkey')
dendrite = Dendrite(wallet=wallet)

# Get axons to query from metagraph
metagraph = Metagraph(netuid=1, network='finney')
axons = metagraph.axons  # All miner endpoints

# Query all miners
responses = await dendrite(
    axons=axons,
    synapse=MySynapse(input="What is 2+2?"),
    timeout=12.0,       # seconds
    deserialize=True
)

# Query specific subset
responses = await dendrite(
    axons=axons[:10],   # Top 10 miners only
    synapse=MySynapse(input="query text"),
    timeout=5.0
)

# Check response status
for response in responses:
    if response.is_success:
        print(response.output)
    elif response.is_timeout:
        print("Timed out")
    elif response.is_failure:
        print(f"Failed: {response.dendrite.status_code}")

# Streaming response (for streaming subnets)
async for chunk in dendrite.forward(axons[0], synapse, streaming=True):
    print(chunk)
```

---

## 6. Synapse — Data Object

Synapse is the **data container** that flows between validators (Dendrite) and miners (Axon). It is a Pydantic model — you always subclass it to define your subnet's specific data format.

```python
from bittensor import Synapse

# Define a custom synapse for your subnet
class TextSynapse(Synapse):
    # Input fields — set by validator, read-only for miner
    prompt: str = ''
    max_tokens: int = 100

    # Output field — filled in by miner
    completion: str = None

class ImageSynapse(Synapse):
    prompt: str = ''
    image_bytes: bytes = None   # Miner fills this in

class PredictionSynapse(Synapse):
    ticker: str = ''
    horizon: int = 24           # hours
    price_prediction: float = None
```

### Synapse Status Properties

```python
synapse.is_success      # True if status 200
synapse.is_failure      # True if non-200, non-timeout
synapse.is_timeout      # True if request timed out
synapse.is_blacklist    # True if validator was blacklisted (401)

# Terminal info
synapse.axon.hotkey     # Miner's hotkey
synapse.dendrite.hotkey # Validator's hotkey
synapse.dendrite.ip     # Validator's IP
```

---

## 7. Full Workflow Examples

### Minimal Miner

```python
import bittensor as bt
import asyncio

class MyMiner:
    def __init__(self):
        self.wallet = bt.Wallet(name='miner_wallet', hotkey='miner_hotkey')
        self.sub = bt.Subtensor(network='finney')
        self.axon = bt.Axon(wallet=self.wallet, port=8091)

    def run(self):
        # Attach forward function
        self.axon.attach(forward_fn=self.forward)

        # Start server and advertise on chain
        self.axon.start()
        self.sub.serve_axon(netuid=1, axon=self.axon)

        print("Miner running...")
        try:
            while True:
                pass
        except KeyboardInterrupt:
            self.axon.stop()

    def forward(self, synapse: bt.Synapse) -> bt.Synapse:
        # Your model inference here
        synapse.output = "model_response"
        return synapse

if __name__ == '__main__':
    MyMiner().run()
```

### Minimal Validator

```python
import bittensor as bt
import asyncio
import time

class MyValidator:
    def __init__(self):
        self.wallet = bt.Wallet(name='val_wallet', hotkey='val_hotkey')
        self.sub = bt.Subtensor(network='finney')
        self.metagraph = bt.Metagraph(netuid=1, network='finney')
        self.dendrite = bt.Dendrite(wallet=self.wallet)

    async def run(self):
        while True:
            # Sync metagraph
            self.metagraph.sync(subtensor=self.sub)

            # Query all miners
            responses = await self.dendrite(
                axons=self.metagraph.axons,
                synapse=bt.Synapse(),
                timeout=10.0
            )

            # Score responses → build weights
            scores = self.score(responses)
            weights = scores / scores.sum()

            # Submit weights on-chain
            self.sub.set_weights(
                wallet=self.wallet,
                netuid=1,
                uids=self.metagraph.uids,
                weights=weights
            )

            time.sleep(360)  # Wait one tempo

    def score(self, responses):
        import torch
        scores = torch.zeros(len(responses))
        for i, r in enumerate(responses):
            if r.is_success:
                scores[i] = self.evaluate(r.output)
        return scores

    def evaluate(self, output):
        # Your evaluation logic here
        return 1.0

if __name__ == '__main__':
    asyncio.run(MyValidator().run())
```

### Reading Metagraph Data (Analytics)

```python
import bittensor as bt

sub = bt.Subtensor(network='finney')
mg = bt.Metagraph(netuid=1, network='finney', lite=False)
mg.sync(subtensor=sub)

# Print top 5 miners by incentive
import torch
top_uids = mg.I.argsort(descending=True)[:5]
for uid in top_uids:
    print(f"UID {uid.item()} | Incentive: {mg.I[uid]:.4f} | Stake: {mg.S[uid]:.2f} | Trust: {mg.T[uid]:.4f}")

# View weights matrix
print("Weight matrix shape:", mg.W.shape)  # [n, n]
print("Bonds matrix shape:", mg.B.shape)   # [n, n]
```

---

## 8. Key SDK Concepts

### mechid — Multiple Incentive Mechanisms (v10)

SDK v10 introduced support for multiple incentive mechanisms within a single subnet. Each mechanism has its own weight matrix, bond pool, and emission distribution.

```python
# Default — mechid=0, backward compatible
metagraph = sub.metagraph(netuid=1)

# Specific mechanism
metagraph = sub.metagraph(netuid=1, mechid=1)

# Set weights for mechanism 1
sub.set_weights(wallet, netuid=1, uids=uids, weights=w, mechid=1)
```

### Safe Staking

```python
# Protects against price slippage when staking
sub.add_stake(
    wallet=wallet,
    hotkey_ss58=wallet.hotkey.ss58_address,
    amount=100.0,
    safe_staking=True,
    rate_tolerance=0.005,         # 0.5% max price change
    allow_partial_stake=True      # Stake as much as price allows
)
```

### Commit-Reveal Weights

Validators use a two-step weight submission to prevent frontrunning:

```python
import hashlib

# Step 1 — Commit hash of weights
sub.commit_weights(
    wallet=wallet,
    netuid=1,
    uids=uids,
    weights=weights,
    salt=[163, 241, 217, 11]   # Random salt for security
)

# Step 2 — Reveal actual weights later (different block)
sub.reveal_weights(
    wallet=wallet,
    netuid=1,
    uids=uids,
    weights=weights,
    salt=[163, 241, 217, 11]   # Same salt used in commit
)
```

### BT_NO_PARSE_CLI_ARGS

When embedding the SDK in an application that manages its own config:

```python
import os
os.environ['BT_NO_PARSE_CLI_ARGS'] = '1'
from bittensor import Subtensor  # Won't interfere with your app's CLI
```

---

## 9. Class Reference Summary

|Class|Key Methods|
|---|---|
|`Subtensor`|`metagraph()`, `set_weights()`, `add_stake()`, `unstake()`, `register()`, `transfer()`, `nominate()`, `delegate()`, `serve_axon()`, `get_current_block()`, `neurons()`|
|`Wallet`|`create_new_coldkey()`, `create_new_hotkey()`, `regenerate_coldkey()`, `.coldkey`, `.hotkey`, `.coldkeypub`|
|`Metagraph`|`sync()`, `.S`, `.R`, `.T`, `.I`, `.E`, `.D`, `.V`, `.W`, `.B`, `.axons`, `.uids`, `.hotkeys`|
|`Axon`|`attach()`, `start()`, `stop()`, `.serve()`|
|`Dendrite`|`__call__()` (query axons), `.forward()` (streaming), `.query()`|
|`Synapse`|Subclass to define data schema. `.is_success`, `.is_timeout`, `.is_failure`, `.is_blacklist`|

---

## Open Questions

- How does the SDK handle disconnections from the Subtensor node mid-operation?
- What are the best practices for running validators across multiple subnets with one SDK instance?
- How are bond matrices `B` updated — by the SDK automatically or manually?
- What is the recommended pattern for persisting metagraph state between validator loops?