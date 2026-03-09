---
banner: "[[Bittensor-TAO.jpg]]"
icon: "[[22974.png]]"
tags:
  - biττensor
---
# Bittensor — Overview

---

## What is Bittensor?

A **decentralized market for machine intelligence**. The goal is to take on centralized AI giants (OpenAI, Anthropic, Google DeepMind) by removing the need for any single company to own, fund, or control AI development.

Instead of one lab hiring engineers and building infrastructure, Bittensor lets the world's compute and intelligence compete openly — rewarding useful AI in **TAO**, the network's native token.

> "You define a problem. The world's intelligence tries to solve it in a brutal Darwinian manner to get rewarded."

---

## Why Decentralize AI?

Centralized AI labs face:

- Massive infrastructure overhead (HR, compute, legal, political pressure)
- Closed models — no permissionless access or contribution
- Single points of failure and control
- Misaligned incentives (VC returns vs. open progress)

Bittensor replaces this with:

- **Permissionless participation** — anyone can mine intelligence
- **No central infrastructure needed** — the protocol is the infrastructure
- **Market-driven quality** — bad models get starved of rewards, good ones thrive
- **Token incentives aligned with actual output** — you get paid for useful intelligence, nothing else

---

## The Three Participants

The global network layer consists of three roles, each competing to survive in a capitalistic, merit-driven environment:

|Participant|Role|Incentive|
|---|---|---|
|**Miners**|Produce the commodity (run models, generate outputs)|Earn TAO by producing high-quality responses|
|**Validators**|Score and rank miner outputs|Earn TAO by ranking honestly and accurately|
|**Subnet Owners**|Define the problem and rules of a subnet|Earn a cut of subnet emissions; attract miners/validators|

Each participant is self-interested. The protocol is designed so that **acting honestly is the most profitable strategy** — this is what Yuma Consensus enforces.

---

## Subnets

Each **subnet** is a dedicated, self-contained competition focused on one problem domain:

- Examples: text generation, image synthesis, financial prediction, protein folding
- Subnet owners define what "good output" looks like and set the scoring rules
- Miners compete to produce the best outputs for that subnet
- Validators judge the outputs and assign weights
- The subnet has its own emission pool — TAO flows to top performers

Think of each subnet as an independent company with no employees — just contractors competing for payment.

---

## TAO — The Native Token

- TAO is the reward for useful intelligence
- New TAO is emitted each block (`τ` emission rate) and distributed via the incentive vector `I`
- Stake in TAO determines voting power — validators with more TAO have more influence over who gets rewarded
- Over time, stake concentrates toward honest, high-performing participants

---

## Yuma Consensus

The mechanism that makes the whole system work at the global level. It solves the core problem: **how do you reward honest validators and prevent colluding groups from farming emissions without doing real work?**

Yuma Consensus ensures:

- Validators who agree with the honest majority earn more
- Validators who deviate (collude, self-vote, lie) get penalized
- Intelligence is found and rewarded in the long run, not just gamed

$(Detailed\ breakdown\ in\ → Incentives\ folder)$

---

## Mental Model

```
Problem defined by Subnet Owner
        ↓
Miners compete to solve it
        ↓
Validators rank miner outputs → Weight matrix W
        ↓
Yuma Consensus aggregates weights → Incentive vector I
        ↓
TAO emitted → S(t+1) = S(t) + τI
        ↓
Stake accumulates to best performers → they gain more influence
```

---

## Key Properties

- **Permissionless** — no application, no approval needed to join
- **Trustless** — the protocol, not a company, enforces rules
- **Darwinian** — underperformers(Subnets, Miners & Validators) are starved of rewards and exit
- **Composable** — subnets can be stacked, intelligence from one can feed another
- **Censorship resistant** — no single entity can shut it down or exclude participants


---

## Explore:

 [[1. Mechanics]]
 [[2. Participants]]
 [[3. Incentives]]
 [[4. Subnets]]
 [[5. Token]]
 [[6. Extra]]
