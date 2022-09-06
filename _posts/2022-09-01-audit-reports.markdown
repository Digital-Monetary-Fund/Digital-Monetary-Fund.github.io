---
layout: post
title:  "State Channels Project"
date:   2022-09-01 12:15:00 +1100
categories: project update
---

The Digital Monetary Fund (DMF) has started a project to design, development and implement State Channels for our network of EVM-based smart stable-coins.

State channels are the general form of payment channels, applying the same idea to any kind of state-altering operation normally performed on a blockchain.

Moving these interactions off of the chain without requiring any additional trust can lead to significant improvements in cost and speed. State channels will be a critical part of scaling blockchain technologies to support higher levels of use.

The basic components of a state channel are very simple:

A 2 way state channel

1) Part of the blockchain state is locked via multisignature or some sort of smart contract, so that a specific set of participants must completely agree with each other to update it.

2) Participants update the state amongst themselves by constructing and signing transactions that could be submitted to the blockchain, but instead are merely held onto for now. Each new update “trumps” previous updates.

3) Finally, participants submit the state back to the blockchain, which closes the state channel and unlocks the state again (usually in a different configuration than it started with).

If the “state” being updated between participants was a digital currency balance, then we would have a payment channel. Steps 1 and 3, which open and close the channel, involve blockchain operations. But in step 2 an unlimited number of updates can be rapidly made without the need to involve the blockchain at all — and this is where the power of state channels comes into play, because only steps 1 and 3 need to be published to the network, pay fees, or wait for confirmations. In fact, with careful planning and design, state channels can remain open almost indefinitely, and be used as part of larger hub and spoke systems to power an entire economy or ecosystem.

DMF will use State Channels to facilitate complex trustless transactions, such as Escrow and Security facilities.



