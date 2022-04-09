---
codename: NFS
title: Non-Fungible Structure Standard (derived from EIP-721)
author: Ilya Tegmark 
discussions-to: https://github.com/ilyategmark/structuredNFT/discussions/
type: Standards
category: Improvement Proposal
status: Request for contributions
created: 2022-04-08
version: 0.1.5
---

## Executive Summary

A standard interface for Non-Fungible Structures (NFS), extending standard interface for non-fungible tokens (NFT), which originated from EIP-721. 

This standard provides answer to question: 
> How would change the way we treat NFTs, if they could have internal mutable structure composed of other NFTs or shares of other NFTs? 

Structured ownership of non-fungible tokens is a logical next step in evolution of NFTs and creates multitude of use  cases: legal entities, structured financial assets, real estate properties, complex projects, SPVs (special purpose vehicles) and many more. All such cases require that owned property could be composed of many components, could belong to several entities in various proportions and are mutable by nature. 

Such higher-level NFT, which holds ownership of other NFTs is named **Non-Fungible Structure** or **NFS**. This paper provides principles of operation and public functions definitions for NFS. 

We are very inclusive in the process of development of this document and invite anyone with contributions into our discussion. 

This standard extends existing NFT standard to account for such use cases and compatible with any NFT on any blockchain.

Examples and definitions in this paper are written in a language of Internet Computer Network — Motoko, which is a clean, powerful and self-explanatory language for smartcontracts and dApps built with Rust. 


## Principles

### Principle 1
> There exists **Non-Fungible Tokens**, **Non-Fungible Structures** and **Users** 

In this paper we follow this legend:

|Name|Abbreviations used in this paper|Identifiers used in this paper|
|---|:-:|--:|
|Non-Fungible Token|NFT|-|
|Non-Fungible Structure|NFS|X, Y, Z|
|Non-Fungibles (either NFT or NFS)|NF|A, B, C| 
|User|-|U, V, W|

### Principle 2
> In contrast to **NFT**, which doesn't have internal structure, the **NFS** can contain other Non-Fungibles, either NFTs or NFSs. 

Here's the table of all possible and impossible states:

|Owner|Owned property|Possible state?|Comment|
|---|:-:|:-:|---|
|User|NFT|True|Classic use case: User fully owns NFT|
|User|NFS|True|User fully owns NFS|
|NFS|NFT|True|Non-Fungible Structure contains fully an NFT|
|NFS|NFS|True|NFS contains fully another NFS|
|NFT|NF|False|NFT can't contain other Non-Fungibles|
|NF|User|False|Non-Fungibles can't contain User|
|User|User|False|User can't own User|

The examples of this principle:
```motoko
// Given the following identifiers of Non-Fungibles (A, B, C, D, X) and User (U): 

// Non-Fungible A fully belongs to NFS X
assert ( ownerOf(A) == X )
assert ( ownersOf(A) == {X: 100%} )

// NFS X fully owns Non-Fungibles A and B
assert ( ownedBy(X) == {A: 100%, B: 100%} )
assert ( ownerOf(B) == X )
assert ( ownersOf(B) == {X: 100%} )

// User U fully owns Non-Fungibles C and D
assert ( ownedBy(U) == {C: 100%, D: 100%} )
assert ( ownerOf(C) == U )
assert ( ownersOf(C) == {U: 100%} )
assert ( ownerOf(D) == U )
assert ( ownersOf(D) == {U: 100%} )
```

### Principle 3
> Partial ownership is allowed: User may own and NFS may contain other non-fungibles partially. 

Here's the table of all possible and impossible states updated for this principle:

|Owner|Owned property|Possible state?|Comment|
|---|:-:|:-:|---|
|User|k% share of NFT|True|User owns k% share of NFT|
|User|k% share of NFS|True|User owns k% share of NFS|
|NFS|k% share of NFT|True|Non-Fungible Structure contains k% share of NFT|
|NFS|k% share of NFS|True|NFS contains k% share of another NFS|
|NFT|k% share of NF|False|NFT can't contain other Non-Fungibles|
|NF|k% share of User|False|Non-Fungibles can't contain User|
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
> The value of **NFS** is equivalent to weighted sum of values of its partially owned elements. 
```motoko
// Price of sNFT X is the sum of prices of its elements A and Y 
// multiplied respectively by the shares owned by X
assert ( ownedBy(X) == {A: 35%, Y: 70%} ) // sum doesn't equal to 100%
assert ( valueOf(X) == valueOf(A) * 35% + valueOf(Y) * 70% )
```
### Principle 6
> As long as user owns > 50% of an asset, it can solely make transactions. If user owns <= 50% of an asset, they can propose transaction with specified deadline, which waits until it reaches > 50% support. User may terminate the proposal before the trasnaction happened. User may agree with the proposal.  User may see the proposals of others and the support they gathered. Users may ask the voting be either anonymous or not.   

### Principle 7
> While Plain NFTs are immutable, the Structured NFTs can change over time: new NFTs can be added, some NFTs can be removed, the share of NFT which belongs to the Structured NFT can change. 

### Principle 8
> Structuredd NFTs are created with charter, which addresses fundamental questions of Structured NFT lifecycle: 
1. Approval vote level (k%) required to make changes to charter
2. Approval vote level (k%) required to change the holders of the Structured NFT
3. Approval vote level (k%) required to change the structure of NFT
4. Approval vote level (k%) required to forced buyout of the Structured NFT from the rest of owners
5. Can you exit the ownership at your will or it requires approval from other owners and at what vote level (k%)
6. Can you burn your ownership of the Structured NFT at your will
7. Approval vote level (k%) required to destroy the Structured NFT
8. What happens to shares of NFTs, owned by sNFT, after sNFT is destroyed (burned, transferred to the owners of destroyed sNFT)

### Principle 9
> Structured NFTs can be destroyed

### Principle 10
> Shares of Structured NFTs can be sold

### Principle 11
> Only Users are conscious to vote and make decisions. If a vote from sNFTs is required, then it's elevated up the ownership graph until Users are found.  
> requestForVote received by sNFT is propagated further up to the owners of sNFT

### Principle 12
> sNFT can possess own shares directly or indirectly through another intermediary sNFTs. The final beneficiary of the NFT doesn't have to be User at all, it can be sNFTs as well. 


### Principle 13
> Users and sNFTs can have special voting rules, regarding any sNFT, which belongs them directly or through intermediary sNFTs. 

### Principle 14
> Users may delegate their voting rights to other Users regarding any sNFT, which belongs them directly or through intermediary sNFTs.

### Principle 15
> NFS has its own balance of coins, which allowed in this network (blockchain).

Charter of sNFT may have rules,  that 

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
public shared query func balanceOf(p : Principal) : async ?Nat 

/// Find the owner of an NFT
/// Derived from the standard 721
/// tokenId — an identifier for an NFT
/// Returns the address of the owner of the NFT
public shared query func ownerOf(tokenId : TokenId) : async ?Principal 

/// Transfers the ownership of an NFT from one address to another address
/// Derived from the standard 721
/// from — an identifier of the current owner of the NFT
/// to — an identifier of the new owner
/// tokenId — an identifier of the NFT to transfer
public shared(msg) func transferFrom(from : Principal, to : Principal, tokenId : Nat) : () 

/// Change or reaffirm the approved address for an NFT
/// Derived from the standard 721
/// tokenId — the NFT to approve
public shared(msg) func approve(to : Principal, tokenId : TokenId) : async () 

/// Enable or disable approval for a third party ("operator") to manage
/// Derived from the standard 721
/// op — an address to add to the set of authorized operators
/// isApproved — true if the operator is approved, false to revoke approval
public shared(msg) func setApprovalForAll(op : Principal, isApproved : Bool) : () 

/// Get the approved address for a single NFT
/// Derived from the standard 721
/// tokenId — an NFT to find the approved address for
/// Returns the approved address for this NFT, or the zero address if there is none
public shared func getApproved(tokenId : Nat) : async Principal 

/// Query if an address is an authorized operator for another address
/// Derived from the standard 721
/// owner — the address that owns the NFTs
/// operator — the address that acts on behalf of the owner
/// Returns true if `operator` is an approved operator for `owner`, false otherwise
public shared func isApprovedForAll(owner : Principal, opperator : Principal) : async Bool 

/// A descriptive name for a collection of NFTs
/// Derived from the standard 721
public shared query func name() : async Text 

/// An abbreviated name for NFTs 
/// Derived from the standard 721
public shared query func symbol() : async Text 

/// A distinct Uniform Resource Identifier (URI) for a given asset
/// Derived from the standard 721
public shared query func tokenURI(tokenId : TokenId) : async ?Text 

/// Creation of an NFT (minting)
/// Derived from the standard 721
public shared(msg) func mint(uri : Text) : async Nat 

/// Destruction of an NFT (burning)
/// Derived from the standard 721
public shared(msg) func burn(tokenId : Nat): async () 


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