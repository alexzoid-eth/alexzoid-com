---
title: "Practical Guide to Certora Formal Verification Contests"
seoTitle: "Certora Formal Verification: Step-by-Step Guide"
seoDescription: "Learn how Certora Formal Verification contests differ from standard audits, explore best practices, and catch a real Uniswap V4 mutation bug."
datePublished: Tue Feb 25 2025 11:58:06 GMT+0000 (Coordinated Universal Time)
cuid: cm7kfn7h0000709jpcekug88m
slug: practical-guide-to-certora-formal-verification-contests
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1740484251511/dc40c767-fda4-442a-bbe3-70da5cc9186c.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1740484337755/4a9ff5ae-3314-4726-aa13-986d585c3c74.jpeg
tags: security, solidity, smart-contracts, formal-verification, certora

---

What are Certora contests? How do they differ from standard audit competitions? How do you get started? Are there any specific nuances? You’ll find the answers to all of these questions here. At the time of writing this article, I’ve participated in nine contests over the past 1.5 years and climbed to the top of the [leaderboard](https://www.certora.com/leaderboard).

This tutorial is divided into two parts:

1. General information about contests.
    
2. A practical walkthrough - a step-by-step guide to catching a public mutation from the previous [Uniswap V4 contest](https://cantina.xyz/competitions/e2cf6906-ec8b-4c78-a585-74ac90615659).
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1740474458089/23d1baac-69bf-495d-b661-c560ee7344ba.png align="center")

## Overview of Certora Contests

Certora contests revolve around Formal Verification (FV). FV isn’t meant to replace other security measures; it's an additional layer on top of manual reviews (including static analyzers, integrity checks, or fuzzing tests). That’s why Certora contests often complement standard audit contests, such as those held by [code4arena](https://code4rena.com/audits), [hats.finance](https://app.hats.finance/), and [cantina](https://cantina.xyz/). This is fantastic because you can participate in both contests and FV pools simultaneously.

You can find announcements on Certora’s official site under the [community contests](https://www.certora.com/contests) page, as well as on [Discord](https://discord.com/channels/795999272293236746/1078776554970173620) or [Twitter](https://x.com/CertoraInc).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1740474861974/7c3c3b01-66ec-4be7-93f9-bfa4a0059d78.png align="center")

### Core Tasks

If you come from regular audit contests, you know you’re typically paid for finding bugs and writing PoCs: the more unique bugs you find, the larger your share of the prize. However, in Certora contests, you’re rewarded for writing FV specifications. The higher the quality of your specifications, the larger your incentive.

You can think of it like writing tests: you write a rule that proves a particular part of the code must behave in a specific way. If the code behaves as assumed, the rule passes. Otherwise (when a mutation is introduced), it violates.

FV specifications consist of rules and invariants in a Solidity-like language called CVL. Rules that focus on high-level properties with broad coverage are more likely to catch multiple mutations (planted bugs). For instance, in the Euler contest, a single high-level invariant [caught all 3 public mutations](https://x.com/alexzoid_eth/status/1805857941947564215):

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1740389556234/280e6552-6ef2-46c9-a6c2-2ea81514b02e.png align="center")

For more examples, check out my past contest repositories for [Euler](https://github.com/alexzoid-eth/euler-vault-cantina-fv) and [UniswapV4](https://github.com/alexzoid-eth/uniswap-v4-periphery-cantina-fv). You’ll also find numerous example rules in Certora’s public [audit reports](https://www.certora.com/audits).

### Work Evaluation

The judging process leverages [mutation testing](https://x.com/alexzoid_eth/status/1806165687012130904). It modifies the source code and checks whether those modifications are caught by your specifications. Each mutation is a modified copy of the source code file.

The best mutations introduce [realistic mistakes](https://github.com/alexzoid-eth/uniswap-v4-periphery-cantina-fv-tutorial/blob/main/certora/mutations/PositionManager/PositionManager_P0.sol#L419-L424):

```solidity
/// @dev overrides solmate transferFrom in case a notification to subscribers is needed
function transferFrom(address from, address to, uint256 id) public virtual override {
    // mutation: replace 'to' with 'address(this)'
    super.transferFrom(from, address(this), id);
    if (positionInfo[id].hasSubscriber()) _notifyTransfer(id, from, to);
}
```

Or [break](https://github.com/alexzoid-eth/euler-vault-cantina-fv/blob/master/certora/mutations/AssetTransfers/AssetTransfers_0.sol#L43-L50) core invariants:

```solidity
// mutation: add pullPushAssets function
function pullPushAssets(VaultCache memory vaultCache, address to, address from, Assets amount) public virtual {
    vaultCache.asset.safeTransferFrom(from, address(this), amount.toUint(), permit2);
    vaultStorage.cash = vaultCache.cash = vaultCache.cash + amount;

    vaultStorage.cash = vaultCache.cash = vaultCache.cash - amount;
    vaultCache.asset.safeTransfer(to, amount.toUint());
}
```

Each mutation is a separate [modified copy](https://github.com/Certora/uniswap-v4-periphery-cantina-fv/tree/main/certora/mutations) of the source code file. The judging process is:

1. Run all your specs to ensure they don’t fail on the source code.
    
2. Replace the original contract with a mutated copy.
    
3. Run all your specs again and check if at least one fails, catching the mutation.
    

In Certora contests, there are two types of mutations: **public** and **private**.

* **Public mutations** are shared at the start of the contest. They’re tied to a smaller **Participation** pool and help new participants understand how mutations work.
    
* **Private mutations** are revealed after the contest ends. They’re tied to the main **Coverage** pool and allow judges to evaluate the quality of your specifications more comprehensively.
    

### Prize Pool Distribution

Yes, Certora contests have a separate judging process and prize pool - often [up to $100k](https://www.certora.com/contests). In recent contests, the FV pool has been split into [three categories](https://github.com/Certora/silo-v2-cantina-fv/blob/main/README.md?plain=1#L53-L56):

1. **Participation (10% of pool):** Awarded for properties that identify public mutants. Think of it as a guaranteed incentive for contributing.
    
2. **Real Bugs (20% of pool):** Awarded for properties that uncover legitimate bugs within the FV scope or related contracts. If no real bugs are found, this part of the pool rolls over to Coverage.
    
3. **Coverage (70% of pool):** Awarded for properties that identify private mutants (revealed after the contest ends). Each mutation is weighted, and the fewer participants who catch a mutation, the larger the reward for those who do.
    

## Practice Part

Let’s go through the step-by-step process of the FV contest. We’ll use a previous [Uniswap V4 contest](https://cantina.xyz/competitions/e2cf6906-ec8b-4c78-a585-74ac90615659) to build a specification for a public mutation.

For a detailed Certora setup, see [First Steps with Certora Formal Verification](https://alexzoid.com/first-steps-with-certora-fv-catching-a-real-bug) tutorial.

All Certora contests have a [dedicated repo](https://github.com/Certora/uniswap-v4-periphery-cantina-fv) containing:

* A clone of the project’s main repo.
    
* A `certora` folder with FV-related materials.
    
* A contest-specific README with essential details.
    

Always read the [README](https://github.com/Certora/uniswap-v4-periphery-cantina-fv/blob/main/README.md) carefully, as it will guide you through the contest setup.

### Import the repo

Create a private repo named `your_handle/uniswap-v4-periphery-cantina-fv` and import the public contest repo. For example:

```bash
git clone https://github.com/Certora/uniswap-v4-periphery-cantina-fv
cd uniswap-v4-periphery-cantina-fv
# move to the right commit at the start of contest
git reset --HARD 5c2f7a46b0c0edb361989fd4d17a5885979f9da8
git push --mirror git@github.com:your_handle/uniswap-v4-periphery-cantina-fv
cd ../
```

Then clone your new private repo and build:

```bash
git clone git@github.com:your_handle/uniswap-v4-periphery-cantina-fv
cd uniswap-v4-periphery-cantina-fv
forge build
```

In a real contest, you’ll add the judges as collaborators (their GitHub handles are in the contest readme) so they can access your specs after the contest ends.

A ready-to-use example repo with this tutorial’s rule is available [here](https://github.com/alexzoid-eth/uniswap-v4-periphery-cantina-fv-tutorial).

### Check that everything is working

A typical initial configuration includes a built-in `sanity` rule that checks for any functions that always revert or time out. You might also see some example rules.

Run the prover:

```bash
certoraRun certora/confs/PositionManager.conf
```

I got this link: [https://prover.certora.com/output/52567/98eeff489c3a4e4b99e3275ad392cbbf/?anonymousKey=8c1855862b6cb9bdb03958ec1520d47e4c340a31](https://prover.certora.com/output/52567/98eeff489c3a4e4b99e3275ad392cbbf/?anonymousKey=8c1855862b6cb9bdb03958ec1520d47e4c340a31)

No violations occurred, which is great!

### Write a rule for a public mutation

Now, let’s examine the public mutation [PositionManager\_P0.sol](https://github.com/Certora/uniswap-v4-periphery-cantina-fv/blob/main/certora/mutations/PositionManager/PositionManager_P0.sol#L419C1-L424):

```solidity
/// @dev overrides solmate transferFrom in case a notification to subscribers is needed
function transferFrom(address from, address to, uint256 id) public virtual override {
    // mutation: replace 'to' with 'address(this)'
    super.transferFrom(from, address(this), id);
    if (positionInfo[id].hasSubscriber()) _notifyTransfer(id, from, to);
}
```

The mutation changes the destination address to `address(this)`. In English, we want a rule: “The destination address should always receive the token during a transfer.” So we can add this to `certora/specs/PositionManager.spec`:

```solidity
// The destination address can always receive the token during a transfer
rule destinationCanReceiveToken(env e, address from, address to, uint256 id) {
      
    transferFrom(e, from, to, id);
   
    assert(ownerOf(e, id) == to);
}
```

Then run (with `--rule` parameter execute only our rule):

```bash
certoraRun certora/confs/PositionManager.conf --rule destinationCanReceiveToken
```

The rule passes on the original code:  
[https://prover.certora.com/output/52567/f9c27d9c87dd4433a52910255cf9626e/?anonymousKey=5abde2d84e349232252d24027c7f10a11423c992](https://prover.certora.com/output/52567/f9c27d9c87dd4433a52910255cf9626e/?anonymousKey=5abde2d84e349232252d24027c7f10a11423c992)

### Test your rule against the mutation

A quick test is to temporarily replace the original [src/PositionManager.sol](https://github.com/Certora/uniswap-v4-periphery-cantina-fv/blob/main/src/PositionManager.sol) with the mutated [certora/mutations/PositionManager/PositionManager\_P0.sol](https://github.com/Certora/uniswap-v4-periphery-cantina-fv/blob/main/certora/mutations/PositionManager/PositionManager_P0.sol), then run the prover again:

```bash
certoraRun certora/confs/PositionManager.conf --rule destinationCanReceiveToken
```

I got this link: [https://prover.certora.com/output/52567/568642afb9484a71ad589ea526637783/?anonymousKey=a1982bd2b1977d8102685ac63b0ac51c7ab34d3d](https://prover.certora.com/output/52567/568642afb9484a71ad589ea526637783/?anonymousKey=a1982bd2b1977d8102685ac63b0ac51c7ab34d3d)

This time, the rule is violated on the mutated code, which means it successfully catches the public mutation. Remember to revert `PositionManager.sol` afterward.

### Test with the mutation engine

Certora’s `certoraMutate` framework offers an automated mutation engine called [Gambit](https://github.com/Certora/gambit), plus a web [dashboard](https://prover.certora.com/mutations) and server infrastructure for parallel testing.

It can check both manually added mutations and automatically generated ones (small modifications like changing arithmetic operators, commenting out lines, etc.). In the [certora/confs/PositionManager.conf](https://github.com/Certora/uniswap-v4-periphery-cantina-fv/blob/main/certora/confs/PositionManager.conf) file, you’ll see something like:

```json
"mutations": {
    "gambit": [
        {
            "filename" : "src/PositionManager.sol",
            "num_mutants": 5
        }
    ],
    "manual_mutants": [
        {
            "file_to_mutate": "src/PositionManager.sol",
            "mutants_location": "certora/mutations/PositionManager"
        }
    ]
}
```

Run:

```bash
certoraMutate certora/confs/PositionManager.conf
```

Under the hood, it:

1. Runs your spec against the original code.
    
2. Generates 5 Gambit mutations to test automatically.
    
3. Tests each manual mutation in `certora/mutations/PositionManager` one by one.
    

For my run, I got: [https://mutation-testing.certora.com/?id=7205302c-9702-4bc5-82e3-63d9408287e1...](https://mutation-testing.certora.com/?id=7205302c-9702-4bc5-82e3-63d9408287e1&anonymousKey=ca0fc1c6-72cf-4609-ac64-ef6b50ebb4ea). You can see that the `destinationCanReceiveToken` rule catches the `PositionManager_P0` mutation.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1740478512738/b3a72b0d-7184-4935-8467-d067730e379a.png align="center")

---

Good luck in future Certora contests, and remember that the key to success is writing broad, high-level rules that can catch multiple types of changes.