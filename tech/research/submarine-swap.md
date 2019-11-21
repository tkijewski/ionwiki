---
latest-revision: '2019-11-21T00:00:00.000Z'
original-author: Ryan Shea (ryan-shea)
created: '2019-01-01T00:00:00.000Z'
status: Accepted
title: Submarine Swap
contributors: Ryan Shea (ryan-shea)
type: article
description: ''
discussions-to: GitHub URL
category: lightning-rnd
---

# Submarine Swap

Submarine Swaps are atomic on-chain to off-chain swaps \(and vice versa, often called Reverse Submarine Swaps\) of cryptocurrencies.

This was made possible \(and put into production\) by Alex Bosworth. Olaoluwa Osuntokun originally coined the term Submarine Swap — one half is above water \(on-chain\), one half is below water \(off-chain\).

## Technical Details

### Problem

Transactions between on-chain blockchain addresses and off-chain Lightning addresses are not directly compatible. This creates a transaction barrier between the underlying blockchain and the off-chain Lightning Network, regardless of implementation.

Submarine swaps solve this issue by enabling Lightning channels to be refilled via an on-chain transfer from the underlying blockchain to the off-chain LN channel.
Reverse Submarine swaps solves the off-chain to on-chain transactions and enable Lightning channels to be reversed obtaining inbound liquidity via an off-chain transfer in the LN to a on-chain address

### Structure

#### A submarine swap essentially looks like this:

1. Alice produces or retrieves a Lightning Network payment invoice.  _It doesn't matter whether the Lightning payment is to Alice or to someone else that Alice is trying to pay_.
2. Alice presents the Lightning invoice to Bob, a "submarine swap provider".
3. Bob quotes what he must be paid _on chain_ in order to pay the Lightning invoice _off-chain_.
4. If Alice accepts the exchange rate, Bob and Alice work together and construct an HTLC that creates a conditional **on-chain** payment to Bob.
5. Alice makes the conditional payment to Bob.
6. The conditional payment to Bob is hashlocked with the same secret that will be revealed if the Lightning invoice is paid.  Bob can only redeem the conditional payment from Alice by making the Lightning payment.
7. Bob pays the Lightning invoice, forcing the Lightning payment recipient to reveal the secret S.
8. Bob uses the secret S to redeem the conditional payment from Alice.

If Bob does not pay the Lightning invoice before it expires, Bob is not able to redeem the conditional payment from Alice. In this case, Alice can wait for the HTLC to expire, then redeem the conditional payment's funds back to herself.

What do submarine swaps allow you to do?

* _trustlessly_ pay someone on-chain to perform an off-chain payment for you

This means that a user can make Lightning Network payments without being on the Lightning Network, rebalance their Lightning Network channels \(refill\) with a fast and low-cost payment on another chain, and perform fast trustless swaps where the usual slow step is made instant.

Submarine conditional swaps have been demonstrated on the Bitcoin and Litecoin Lightning Networks, using on-chain payments on Bitcoin, Litecoin, and BCH. They are possible \(albeit with more steps\) using on-chain payments on other Bitcoin derivatives, Ethereum, Stellar, Ripple, and more.

#### A reverse submarine swap essentially looks like this:

1. Alice generates a secret preimage, think in the secret as a random number that only the user knows, we call it preimage because we are going to use it as an input of a hash function, and we are going to make public (at the beginning) only the hash, hashes are one way functions, so knowing the hash give other no clues about which is the secret, but reached the point where the secret is revealed anyone can check that the secret originated that exact hash

2. Alice sends off-chain \(via Lightning\) a conditional payment to  Bob, a "submarine swap provider", implemented with a HODL invoice, the condition is that the payment need the secret as an input to be settled or it will return to the payer \(Alice\) if a determined amount of time has elapsed. As soon as Bob discovers the secret \(when Alice reveals it\) he could take that money off-chain, meanwhile it will be on hold \(HODL\). We will call this, the swap payment

3. Alice sends off-chain \(via Lightning\) another payment, called the prepayment, in this case, this is a normal payment of at most a few thousands satoshis, with no condition as its purpose is to cover Bobs on-chain fees that needed to afford while constructing the swap structure. In case the swap fails or its cancelled, Alice won’t get this prepayment back, as the provider has already \(if he is fair\) spent that in the on-chain fees of the tx described below

4. Bob broadcasts an on-chain hash locked output tx ,it is a transaction to an output that can only be spent using the secret preimage, \(the one only Alice knows\) this is the payment to Alice on-chain and the amount of this transaction is the amount of the intended swap. It also has a time condition in case the swap fails, so as we have described we are in front of a Hash Time Locked Contract \(HTLC\), this is constructed using the same hash mentioned in 1., so the secret needed to unlock it, is the same that Alice generated and that at this point only Alice knows. This contract is imposing the following condition to Alice: “if you want the on-chain payment in your wallet, you have to let me settle the swap payment \(off-chain conditional payment\) by publishing your secret” in this way we have the same secret that unlocks the swap payment in favor of Bob and the on-chain payment in favor of Alice 

5. Alice publishes the secret by disclosing the secret preimage in the spending transaction, HTLC \(the on-chain transaction that put the money in Alice wallet, often called sweep the payment\), so now anyone can see the secret, especially the provider

6. Bob uses the same secret published by Alice in 5. to settle the swap payment, the HODL invoice \(Bobs off-chain money in exchange for the money he locked on-chain\)

What do reverse submarine swaps allow you to do?

* _trustlessly_ pay someone off-chain to perform an on-chain payment for you

This means that a user can make BTC payments even if he has all his funds commited to Lightning Network channels, rebalance their Lightning Network channels obtaining inbound liquidity, etc.

## Examples

**REDSHIFT** 

{% embed url="https://ion.radar.tech/redshift" %}


**Lightning Labs** has released **Loop**, a submarine swap service that plugs into LND.  Loop allows users running LND nodes to exchange BTC in Lightning channels for BTC on-chain, and vice-versa.  Loop charges a fee \(0.1% as of 2019-10-24\) for this service.  The Loop client is open source but the server is proprietary—to perform a swap, you have to perform it with Lightning Labs.

The Loop swap server is hosted on this node:

`03fb2a0ca79c005f493f1faa83071d3a937cf220d4051dc48b8fe3a087879cf14a`

It's not possible to connect to this node directly, but you can connect to [this node ](https://1ml.com/node/021c97a90a411ff2b10dc2a8e32de2f29d2fa49d41bfbb52bd416e460db0747d0d)that acts as a gateway to the Loop node:

`021c97a90a411ff2b10dc2a8e32de2f29d2fa49d41bfbb52bd416e460db0747d0d@18.224.56.146:9735`

A Loop server is also available on the Bitcoin testnet at [this node](https://1ml.com/testnet/node/0223acffd7f363b4591ce860eda870fea352e981212d8a25e96a0ebea37faae288):

`0223acffd7f363b4591ce860eda870fea352e981212d8a25e96a0ebea37faae288@40.71.39.161:9735`

{% embed url="https://blog.lightning.engineering/posts/2019/03/20/loop.html" %}

{% embed url="https://github.com/lightninglabs/loop" %}

**Alex Bosworth** released an early demo of submarine swaps using fully open-source code:

{% embed url="https://submarineswaps.org/" %}

{% embed url="https://github.com/submarineswaps/swaps-service" %}

Boltz Exchange has also open-sourced an implementation that is also compatible with Litecoin LN:

{% embed url="https://boltz.exchange/" %}

{% embed url="https://github.com/BoltzExchange" %}

## Resources

[London Bitcoin Devs - Submarine Swaps and Loop](http://diyhpl.us/wiki/transcripts/london-bitcoin-devs/2019-07-03-alex-bosworth-submarine-swaps/)

[Submarine Swaps](https://submarineswaps.org)

[Loop out in depth](https://blog.lightning.engineering/technical/posts/2019/04/15/loop-out-in-depth.html)

### Key People

* [Alex Bosworth](https://twitter.com/alexbosworth)

### See also

[Submarine Swaps Service](https://github.com/submarineswaps/swaps-service)

## References

\[1\] [https://submarineswaps.org/](https://submarineswaps.org/)
