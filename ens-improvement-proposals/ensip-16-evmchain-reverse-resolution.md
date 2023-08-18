---
description: >-
  Specifies reverse resolution in a cross-chain context
---

# ENSIP-XX: EVM-chain Reverse Resolution

| **Author**    | Jeff Lau \<jeff@ens.domains> |
| ------------- | ---------------------------- |
| **Status**    | Draft                        |
| **Submitted** | 2023-03-14                   |

## Abstract

This ENSIP specifies a way for reverse resolution to be used on other EVM chains.

## Motivation

Reverse resolution has been in use since ENS's inception, however at the time Ethereum had no concrete scaling plans. In the past 5 years, we've seen layer 2s and sidechains become more prevalent and we first allowed support for these with ENSIP-9 (formerly EIP-2304) to allow addresses from other chains to be stored on ENS. Reverse resolution can be expanded to allow the forward resolution to first check the records for the chain that the user is on.

With account abstraction becoming more popular, it is becoming increasingly important to be able to setup different addresses for each L2, but using the same ENS name. It is no longer good practise to assume that the address of a user in mainnet, is also controlled by the same user on an EVM-compatible L2. This can be solved with ENS names resolving to different addresses on each chain, but also additionally the primary ENS name being set to the same ENS name on each chain preserving the identity of the user across chains.

## Specification

### Overview

* Reverse registrars will be setup on each L2, with a corresponding registry
* Reverse registrar will only allow setting the name, without resolver customisability. This is to allow a consistent storage slot that can be checked on L1.
* User can now claim their reverse on this L2 and set it to their ENS name
* Their ENS name will need to setup their record for the same EVM-cointype as the network, which is specified in [ENSIP-11](https://docs.ens.domains/ens-improvement-proposals/ensip-9-multichain-address-resolution)
* A dapp can then detect the chainID that a user is on, find the corresponding cointype and resolve their primary ens name by resolving the name record at [userAddress].[evmChainCointype].reverse. This will be resolved via CCIP-read and look up the reverse record on the corresponding EVM-chain.
* Dapp will then resolve this name via ENS on L1 to check if the forward resolution matches. This forward resolution can be on L1, or the user can set up CCIP-read records for each network and put those addresses wherever they want.

### Resolving Primary ENS Names by a dapp

* If a dapp has a connected user it SHOULD first check the chainId of the user
* If the chainId is not 1, it SHOULD then construct the ENS name [address].[evmChainCoinType].reverse to obtain the primary ENS name of the user. If none is found, it SHOULD assume that the user has no primary ENS name set
* If the dapp finds an ENS name, it MUST first check the forward resolution. The forward resolution MUST match the same coin type as the chain id that the user

### Resolving an avatar by a dapp on L2

ENSIP-12 was concieved before the ENS L2 reverse resolution specification and therefore should be updated to reflect the current state of ENS primary name resolution. This means that all avatar records are able to be updated on a per-chain basis by updating the avatar record on their reverse node.

#### Example for an EVM Address

To determine the avatar URI for a specific EVM chain address, the client MUST reverse-resolve the address by querying the ENS registry on Ethereum for the resolver of `<address>.<evmChainId>.reverse`, where `<address>` is the lowercase hex-encoded Ethereum address, without leading '0x'. Then, the client calls `.text(namehash('<address>.<evmChainId>.reverse'), 'avatar')` to retrieve the avatar URI for the address.

If a resolver is returned for the reverse record, but calling `text` causes a revert or returns an empty string, the client MUST call `.name(namehash('<address>.<evmChainId>.reverse'))`. If this method returns a valid ENS name, the client MUST:

1. Validate that the reverse record is valid, by resolving the returned name and calling `addr` on the resolver, checking it matches the original <chaiId> address.
2. Perform the process described under 'ENS Name' to look for a valid avatar URI on the name.

A failure at any step of this process MUST be treated by the client identically as a failure to find a valid avatar URI.

## Further considerations

* ENS has not been explicit about how to use the mainnet `addr()` record and it is often used as a backup to a user not having an address record set. This practice should not be recommended in the future for a couple reasons. Firstly users may not have control of that address on the other network. Secondly it should not be used as it creates confusion in regard to which address record was matched via forward resolution to ensure a successful reverse resolution.
* Possibility for a resolver that allows gateway choices for the record level to allow a user to split their records between different chains. This would allow them to have arbitrum address records on arbitrum, optimism on optimism and so on.
* Would require subnames and resolvers setup under [evmChainCoinType].reverse that would need to be governed by the DAO. Each new chain would likely need to be approved by the DAO or could be delegated to a multi-sig at first as things are rolled out and then moved to DAO ownership

### Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).