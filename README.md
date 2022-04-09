---
codename: Structured NFT
title: Structured Non-Fungible Token Standard (derived from EIP-721)
author: Ilya Tegmark 
discussions-to: https://github.com/ilyategmark/structuredNFT/discussions/
type: Standards
category: Improvement Proposal
status: Request for contributions
created: 2022-04-08
version: 0.1.4
---

## Executive Summary

A standard interface for structured non-fungible tokens (sNFT), extending standard interface for plain NFT, which originated from EIP-721. 

This standard provides answer to question: 
> What if ownership of an NFT could be shared among several entities: not only users, but also other NFTs?

One small change creates multitude of use cases in registration of legal entities, structured financial assets, real estate, complex projects, SPVs (special purpose vehicles) etc.

Such higher-level NFT, which sole purpose is to hold ownership of other NFTs is named **Structured NFT** or **sNFT**. This paper provides public functions definitions for sNFT. 

We are very inclusive in the process of development of this document and invite anyone with contributions into our discussion. 

## Rationale

Structured ownership of non-fungible tokens is a logical next step in evolution of NFTs. Legal entities, structured financial assets, real estate properties, complex projects, SPVs (special purpose vehicles) and other cases require that owned property could be composed of many components and could belong to several entities in various proportions. This standard extends existing NFT standard to account for such use cases and compatible with any NFT on any blockchain.

## Principles

### Principle 1
> There are two types of NFTs: **Plain NFTs** and **Structured NFTs**. 

In this paper we follow this legend:

|Name|Abbr.|ids used in this paper|
|---|---|---|
|Plain NFT|pNFT|-|
|Structured NFT|sNFT|X, Y, Z|
|Any NFT|NFT|A, B, C| 
|User|-|U, V, W|

### Principle 2
> In contrast to **Plain NFTs**, the **Structured NFTs** can contain other NFTs. Here's the table of all possible and impossible states:

| Owner | Owned property | Possible state? |Comment|
|---|---|---|---|
|User|pNFT|True|Standard use case: User fully owns Plain NFT|
|User|sNFT|True|User fully owns Structured NFT|
|sNFT|pNFT|True|Structured NFT contains fully a Plain NFT|
|sNFT|sNFT|True|Structured NFT contains fully another Structured NFT|
|pNFT|sNFT|False|Plain NFT can't contain Structured NFT|
|NFT|User|False|NFT can't contain User|
|User|User|False|User can't own User|

The examples of this principle:
```motoko
// Given the following NFTs and user's ids: A, B, C, D, X, U

// NFT A fully belongs to sNFT X
assert ( ownerOf(A) == X )
assert ( ownersOf(A) == {X: 100%} )

// sNFT X fully owns NFTs A and B
assert ( ownedBy(X) == {A: 100%, B: 100%} )
assert ( ownerOf(B) == X )
assert ( ownersOf(B) == {X: 100%} )

// User U fully owns NFTs C and D
assert ( ownedBy(U) == {C: 100%, D: 100%} )
assert ( ownerOf(C) == U )
assert ( ownersOf(C) == {U: 100%} )
assert ( ownerOf(D) == U )
assert ( ownersOf(D) == {U: 100%} )
```

### Principle 3
> The price of **Structured NFT** is equivalent to sum of prices of its fully owned elements. 

Here's an example of code:
```motoko
// Price of sNFT X is the sum of prices of its elements A and Y
assert ( ownedBy(X) == {A: 100%, Y: 100%} )
assert ( priceOf(X) == priceOf(A) + priceOf(Y) )
```
### Principle 4
> Partial ownership is allowed: Users and sNFTs may own other NFTs partially. 

Here's the table of all possible and impossible states updated for this principle:

| Owner | Owned property | Possible state? |Comment|
|---|---|---|---|
|User|k% share of pNFT|True|User owns k% share of Plain NFT|
|User|k% share of sNFT|True|User owns k% share of Structured NFT|
|sNFT|k% share of pNFT|True|Structured NFT contains k% share of Plain NFT|
|sNFT|k% share of sNFT|True|Structured NFT contains k% share of another Structured NFT|
|pNFT|k% share of sNFT|False|Plain NFT can't contain Structured NFT|
|NFT|k% share of User|False|NFT can't contain User|
|User|k% share of User|False|User can't own User|

For example: 
```motoko
// 35% of NFT A belongs to Structured NFT X and the rest 65% of NFT A belongs to user U 
assert ( ownersOf(A) == {X: 35%, U: 65%} ) // sum equals to 100%
assert ( shareOf(A, X) == 35% ) // share of NFT A belonging to sNFT X equals to 35%
assert ( shareOf(A, U) == 65% ) // share of NFT A belonging to user U equals to 65%

// User U owns 65% of NFT A and 20% of sNFT Y
assert ( ownedBy(U) == {A: 65%, Y: 20%} ) // sum doesn't equal to 100%
assert ( shareOf(A, U) == 65% ) // share of NFT A belonging to user U equals to 65%
assert ( shareOf(Y, U) == 20% ) // share of sNFT Y belonging to user U equals to 20%

// Structured NFT X owns 35% of NFT A and 70% of sNFT Y
assert ( ownedBy(X) == {A: 35%, Y: 70%} ) // sum doesn't equal to 100%
assert ( shareOf(A, X) == 35% ) // share of NFT A belonging to sNFT X equals to 35%
assert ( shareOf(Y, X) == 70% ) // share of sNFT Y belonging to sNFT X equals to 70%
```


### Principle 5
> The price of **Structured NFT** is equivalent to weighted sum of prices of its partially owned elements. 
```motoko
// Price of sNFT X is the sum of prices of its elements A and Y 
// multiplied respectively by the shares owned by X
assert ( ownedBy(X) == {A: 35%, Y: 70%} ) // sum doesn't equal to 100%
assert ( priceOf(X) == priceOf(A) * 35% + priceOf(Y) * 70% )
```
### Principle 6
> As long as user owns > 50% of an asset, it can solely make transactions. If user owns <= 50% of an asset, they can propose transaction with specified deadline, which waits until it reaches > 50% support. User may terminate the proposal before the trasnaction happened. User may agree with the proposal.  User may see the proposals of others and the support they gathered. Users may ask the voting be either anonymous or not.   

### Principle 7
> While Plain NFTs are immutable, the Structured NFTs can change over time: new NFTs can be added, some NFTs can be removed, the share of NFT which belongs to the Structured NFT can change. 

### Principle 8
> Special rules can be applied to Structured NFT's charter during the creation: 
> 1. Approval vote level (k%) required to make changes to charter
> 1. Approval vote level (k%)required to change the holders of the Structured NFT
> 1. Approval vote level (k%)required to change the structure of NFT
> 1. Approval vote level (k%)required to forced buyout of the Structured NFT from the rest of owners
> 1. Can you exit the ownership at your will or it requires approval from other owners and at what vote level (k%)
> 1. Can you burn your ownership of the Structured NFT at your will

## Abstract

The following standard allows for the implementation of a standard API for Plain and Structured NFTs within smart contracts. Its derived from https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md. 

This standard provides basic functionality to track and transfer Plain and Structured NFTs accounting for partial ownership.

We considered use cases of NFTs being owned and transacted by individuals or entities. NFTs can represent ownership over digital or physical assets:

- Physical property — land, real estate
- Digital property — licenses
- Legal entities — companies, decentralised organizations, special purpose vehicles


## Specification

**Every sNFT compliant contract must implement the `ERC721` interfaces:**

```motoko
/// sNFT Structured Non-Fungible Token Standard
/// https://github.com/ilyategmark/structuredNFT
/// Derived from https://github.com/SuddenlyHazel/DIP721/blob/main/src/DIP721/DIP721.mo 
/// Derived from https://blocks-editor.github.io/blocks/ 
/// Derived from https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md 

/// Count all NFTs assigned to an owner
/// Derived from the standard 721
/// p: Principal — an identifier for whom to query the balance
public shared query func balanceOf(p : Principal) : async ?Nat { ... };

/// Find the owner of an NFT
/// Derived from the standard 721
/// tokenId — an identifier for an NFT
/// Returns the address of the owner of the NFT
public shared query func ownerOf(tokenId : TokenId) : async ?Principal { ... };

/// Transfers the ownership of an NFT from one address to another address
/// Derived from the standard 721
/// from — an identifier of the current owner of the NFT
/// to — an identifier of the new owner
/// tokenId — an identifier of the NFT to transfer
public shared(msg) func transferFrom(from : Principal, to : Principal, tokenId : Nat) : () { ... };

/// Change or reaffirm the approved address for an NFT
/// tokenId — the NFT to approve
public shared(msg) func approve(to : Principal, tokenId : TokenId) : async () { ... };

/// Enable or disable approval for a third party ("operator") to manage
/// op — an address to add to the set of authorized operators
/// isApproved — true if the operator is approved, false to revoke approval
public shared(msg) func setApprovalForAll(op : Principal, isApproved : Bool) : () { ... };

/// Get the approved address for a single NFT
/// tokenId — an NFT to find the approved address for
/// Returns the approved address for this NFT, or the zero address if there is none
public shared func getApproved(tokenId : Nat) : async Principal { ... }; 

/// Query if an address is an authorized operator for another address
/// owner — the address that owns the NFTs
/// operator — the address that acts on behalf of the owner
/// Returns true if `operator` is an approved operator for `owner`, false otherwise
public shared func isApprovedForAll(owner : Principal, opperator : Principal) : async Bool { ... };

/// A descriptive name for a collection of NFTs
public shared query func name() : async Text { ...};

/// An abbreviated name for NFTs 
public shared query func symbol() : async Text { ... };

/// A distinct Uniform Resource Identifier (URI) for a given asset.
public shared query func tokenURI(tokenId : TokenId) : async ?Text { ... };

/// Creation of an NFT (minting)
public shared(msg) func mint(uri : Text) : async Nat { ... };

/// Destruction of an NFT (burning)
public shared(msg) func burn(tokenId : Nat): async () { ... };


```



The **enumeration extension** is OPTIONAL. This allows your contract to publish its full list of NFTs and make them discoverable.

```solidity
/// @title ERC-721 Non-Fungible Token Standard, optional enumeration extension
/// @dev See https://eips.ethereum.org/EIPS/eip-721
///  Note: the ERC-165 identifier for this interface is 0x780e9d63.
interface ERC721Enumerable /* is ERC721 */ {
    /// @notice Count NFTs tracked by this contract
    /// @return A count of valid NFTs tracked by this contract, where each one of
    ///  them has an assigned and queryable owner not equal to the zero address
    function totalSupply() external view returns (uint256);

    /// @notice Enumerate valid NFTs
    /// @dev Throws if `_index` >= `totalSupply()`.
    /// @param _index A counter less than `totalSupply()`
    /// @return The token identifier for the `_index`th NFT,
    ///  (sort order not specified)
    function tokenByIndex(uint256 _index) external view returns (uint256);

    /// @notice Enumerate NFTs assigned to an owner
    /// @dev Throws if `_index` >= `balanceOf(_owner)` or if
    ///  `_owner` is the zero address, representing invalid NFTs.
    /// @param _owner An address where we are interested in NFTs owned by them
    /// @param _index A counter less than `balanceOf(_owner)`
    /// @return The token identifier for the `_index`th NFT assigned to `_owner`,
    ///   (sort order not specified)
    function tokenOfOwnerByIndex(address _owner, uint256 _index) external view returns (uint256);
}
```



## References

**Standards**

1. This document https://github.com/ilyategmark/structuredNFT
1. [DIP-721] (https://github.com/SuddenlyHazel/DIP721/blob/main/src/DIP721/DIP721.mo) DIP-721 implementation in Internet Computer in Motoko lang 
1. [EIP-721] (https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md) EIP-721 original NFT standart in Ethereum in Solidity lang 

**Issues**

1. The Original ERC-721 Issue. https://github.com/ethereum/eips/issues/721

**Discussions**

1. sNFT discussion https://github.com/ilyategmark/structuredNFT/discussions

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).