


# Bittensor — Wallet SDK (bittensor-wallet)

---

## Overview

The Wallet SDK (`bittensor-wallet` / `btwallet`) is a **separate, standalone package** — a Python interface over a Rust cryptographic backend. It handles everything to do with keys and wallets at a low level.

**It is not the same as the Python SDK or btcli.** Here is how the three packages relate:

|Package|PyPI name|What it does|
|---|---|---|
|Python SDK|`bittensor`|Blockchain interaction, mining, validating, metagraph|
|CLI|`bittensor-cli`|Command line operations|
|**Wallet SDK**|**`bittensor-wallet`**|**Key generation, encryption, signing — wallet only**|

The Wallet SDK is automatically installed as a dependency when you install either the Python SDK or btcli. But it can also be installed **standalone** for apps that only need wallet functionality — no blockchain, no mining, no validating.

### When to use the Wallet SDK standalone

- Building a lightweight app that only needs to sign transactions
- Building a mobile/browser integration that doesn't need Subtensor
- Embedding wallet functionality in a non-Bittensor Python app
- Keeping your signing logic completely isolated from chain logic

---

## Installation

```bash
# Standalone (user)
python3 -m venv btwallet-venv
source btwallet-venv/bin/activate
pip install bittensor-wallet

# Standalone (developer — builds from Rust source)
git clone https://github.com/opentensor/btwallet.git
cd btwallet
pip install maturin
maturin develop

# Verify
python3 -c "import bittensor_wallet; print(bittensor_wallet.__version__)"
```

**Requirements:** Python 3.9–3.12. Rust is only needed if contributing to the Rust source — not for usage.

> If you already have the Python SDK or btcli installed (v8.2.0+), the Wallet SDK is already on your machine.

---

## Key Concepts

### What the Wallet SDK manages

```
~/.bittensor/wallets/
└── my_wallet/               ← coldkey name = wallet name
    ├── coldkey              ← encrypted coldkey file (NaCl)
    ├── coldkeypub.txt       ← unencrypted public coldkey
    └── hotkeys/
        └── my_hotkey        ← unencrypted hotkey file
```

- **Coldkey:** NaCl encrypted by default. Holds TAO. Must decrypt to sign high-value ops.
- **Coldkeypub:** Always unencrypted. Public address only. Safe on any machine.
- **Hotkey:** Unencrypted by default. Used for day-to-day ops (mining, validating, signing queries).

### Existential Deposit

The minimum TAO balance a wallet must hold to stay active is **500 RAO** (RAO = smallest TAO unit, 1 TAO = 10⁹ RAO). If a wallet drops below this, the account is deactivated and the remaining balance is destroyed.

---

## 1. Wallet Class

```python
from bittensor_wallet import Wallet

# Create wallet with defaults (name='default', hotkey='default')
wallet = Wallet()
wallet.create()  # generates coldkey + hotkey, prints mnemonics

# Create wallet with custom names
wallet = Wallet(
    name='my_wallet_name',
    hotkey='my_hotkey_name',
    path='~/.bittensor/wallets/'  # default path
)
wallet.create()

# Load existing wallet (no key decryption yet)
wallet = Wallet(name='my_wallet', hotkey='my_hotkey')
print(wallet)
# → Wallet (Name: 'my_wallet', Hotkey: 'my_hotkey', Path: '~/.bittensor/wallets/')
```

### Accessing Keys

```python
# Public coldkey — always accessible, no password needed
pub = wallet.coldkeypub
print(pub.ss58_address)    # Your public TAO address

# Full coldkey — requires password (prompts user)
ck = wallet.coldkey        # decrypts and loads

# Hotkey — unencrypted, always accessible
hk = wallet.hotkey
print(hk.ss58_address)    # Hotkey SS58 address
```

### Creating Keys Separately

```python
# Create only a new coldkey
wallet.create_new_coldkey(
    n_words=12,            # 12 or 24 word mnemonic
    use_password=True,     # encrypt the coldkey file
    overwrite=False        # don't overwrite if exists
)

# Create only a new hotkey
wallet.create_new_hotkey(
    n_words=12,
    use_password=False,    # hotkeys are unencrypted by default
    overwrite=False
)

# Create from URI (derivation path — advanced)
wallet.create_coldkey_from_uri(
    uri='//Alice',         # dev URI
    use_password=False
)
```

### Regenerating Keys (Recovery)

```python
# Recover coldkey from mnemonic
wallet.regenerate_coldkey(
    mnemonic="word1 word2 word3 word4 word5 word6 word7 word8 word9 word10 word11 word12",
    use_password=True,
    overwrite=True
)

# Recover hotkey from mnemonic
wallet.regenerate_hotkey(
    mnemonic="word1 word2 ... word12",
    use_password=False,
    overwrite=True
)

# Recover public coldkey only (for new machine setup)
wallet.regenerate_coldkeypub(
    ss58_address='5D...',  # your public address
    overwrite=True
)
```

---

## 2. Keypair

The `Keypair` is the low-level cryptographic object. The Wallet wraps it — but you can use it directly if needed.

```python
from bittensor_wallet import Keypair

# Generate a new random keypair
kp = Keypair.generate_mnemonic(words=12)
kp = Keypair.create_from_mnemonic(mnemonic)

# Create from seed
kp = Keypair.create_from_seed(seed_hex)

# Create from URI (dev/test)
kp = Keypair.create_from_uri('//Alice')

# Key properties
kp.ss58_address        # Public SS58 address
kp.public_key          # Raw public key bytes
kp.private_key         # Raw private key bytes (if available)
kp.mnemonic            # Mnemonic (if generated with one)
```

### Signing & Verifying

```python
# Sign a message
message = b"hello bittensor"
signature = kp.sign(message)

# Verify a signature
is_valid = kp.verify(message, signature)
print(is_valid)  # True

# Verify with a different keypair (checking sender)
verifier = Keypair(ss58_address=sender_address)
is_valid = verifier.verify(message, signature)
```

---

## 3. Keyfile

`Keyfile` is the low-level class managing the encrypted file on disk. The `Wallet` class uses it internally.

```python
from bittensor_wallet import Keyfile

# Load a keyfile
kf = Keyfile(path='~/.bittensor/wallets/my_wallet/coldkey')

# Check status
kf.exists_on_device()   # True/False
kf.is_readable()        # True/False
kf.is_encrypted()       # True/False — NaCl encrypted?

# Get the keypair (decrypts if encrypted — prompts for password)
keypair = kf.keypair

# Encryption formats
# NaCl      → current default (secure)
# Ansible   → legacy format
# Legacy    → very old format
# Use btcli wallet update to migrate legacy wallets to NaCl
```

---

## 4. Config

```python
from bittensor_wallet.config import Config

config = Config()

# Use config when creating wallets in non-interactive apps
wallet = Wallet(config=config)
```

---

## 5. Relationship to Python SDK and btcli

The Python SDK imports the Wallet SDK internally:

```python
# Inside bittensor SDK (you don't write this — it happens automatically)
from bittensor_wallet import Wallet, Keypair

# When you do this in the Python SDK:
from bittensor import Wallet
# → It's actually bittensor_wallet.Wallet underneath
```

This means the `Wallet` object you use in the Python SDK and the `Wallet` object from the standalone Wallet SDK are the **same class**. The Python SDK just re-exports it.

btcli also uses the Wallet SDK under the hood for all key operations — every `btcli wallet` command ultimately calls into `bittensor_wallet`.

---

## 6. Wallet File Structure (Full)

```
~/.bittensor/
└── wallets/
    └── <coldkey_name>/           ← this IS the wallet name
        ├── coldkey               ← NaCl encrypted JSON, contains private key
        ├── coldkeypub.txt        ← plain text SS58 address
        └── hotkeys/
            └── <hotkey_name>     ← plain JSON, unencrypted by default
```

**File formats:**

- `coldkey` — NaCl encrypted. Never share. Password required to access private key.
- `coldkeypub.txt` — Safe to copy to any machine. Required for mining on remote servers without exposing coldkey.
- `hotkeys/<name>` — Unencrypted JSON. Safe to have on a mining server. Contains no TAO.

---

## 7. Security Best Practices

**Coldkey:**

- Never put the full coldkey on a mining/validating server
- Copy only `coldkeypub.txt` to servers that need the public address
- Store the mnemonic offline — paper, hardware wallet, encrypted offline storage
- Use a separate machine (coldkey workstation) for any operation requiring the coldkey private key

**Hotkey:**

- Can live on your mining server unencrypted — this is by design
- If a hotkey is compromised, an attacker can impersonate your miner/validator operations but cannot steal your TAO (which is controlled by the coldkey)
- Rotate hotkeys with `btcli wallet swap-hotkey` if compromised (costs 1 TAO)

**Proxy wallets:**

- For frequent staking/transfer operations, set up a proxy wallet
- Proxies can perform limited operations on behalf of your coldkey
- If a proxy is compromised, there is a time delay before unauthorized transactions execute — giving you time to cancel

**Mnemonic:**

- The only way to recover a lost coldkey is from the mnemonic
- If the mnemonic is lost, the TAO is permanently inaccessible
- Never store the mnemonic digitally on an internet-connected device

---

## 8. Three Packages — Quick Comparison

||bittensor (SDK)|bittensor-cli|bittensor-wallet|
|---|---|---|---|
|**Purpose**|Build miners/validators/subnets|Operate the network|Manage keys only|
|**Interface**|Python API|Terminal commands|Python API|
|**Backend**|Python|Python|Python + Rust|
|**Blockchain access**|Yes — Subtensor|Yes — via btcli|No|
|**Standalone use**|Yes|Yes|Yes|
|**Auto-installs wallet SDK**|Yes|Yes|—|
|**Import**|`import bittensor`|`btcli` in terminal|`import bittensor_wallet`|
|**GitHub**|opentensor/bittensor|opentensor/btcli|opentensor/btwallet|

---

## Open Questions

- Can the Wallet SDK be used with hardware wallets (Ledger) directly from Python, or only via the browser extension?
- How does the NaCl encryption scheme compare to BIP-38 used in Bitcoin wallets?
- Is there a way to use the Wallet SDK with a Substrate URI derivation path for generating child keys?
- What exactly happens during a coldkey swap at the cryptographic level — does the old key get burned on-chain?