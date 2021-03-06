---
codename: NFS
title: Non-Fungible Structure Standard (derived from EIP-721)
author: Ilya Tegmark 
discussions-to: https://github.com/ilyategmark/structuredNFT/discussions/
type: Standards
category: Improvement Proposal
status: Request for contributions
created: 2022-04-08
version: 0.1.6
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
> There exist **Non-Fungible Tokens**, **Non-Fungible Structures** and **Users** 

In this paper we follow this legend:

|Name|Wording used in this paper|Identifiers used in this paper|
|:-:|:-:|:-:|
|Non-Fungible Token|NFT \| plain NFT|-|
|Non-Fungible Structure|NFS \| Structure|X, Y, Z|
|Either NFT or NFS|NF \| Non-Fungible \| asset|A, B, C| 
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

// Non-Fungible A fully belongs to Structure X
assert ( ownerOf(A) == X )
assert ( ownersOf(A) == {X: 100%} )

// Structure X fully owns Non-Fungibles A and B
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
// 35% of NFT A belongs to Structure X and the rest 65% of that NFT belongs to User U 
assert ( ownersOf(A) == {X: 35%, U: 65%} ) // sum must equal to 100%
assert ( shareOf(A, X) == 35% ) // share of NFT A belonging to Structure X equals to 35%
assert ( shareOf(A, U) == 65% ) // share of NFT A belonging to User U equals to 65%

// User U owns 65% of NFT A and 20% of Structure Y
assert ( ownedBy(U) == {A: 65%, Y: 20%} ) // sum doesn't have to be equal to 100%
assert ( shareOf(A, U) == 65% ) // share of NFT A belonging to User U equals to 65%
assert ( shareOf(Y, U) == 20% ) // share of Structure Y belonging to User U equals to 20%

// Structure X owns 35% of NFT A and 70% of Structure Y
assert ( ownedBy(X) == {A: 35%, Y: 70%} ) // sum doesn't have to be equal to 100%
assert ( shareOf(A, X) == 35% ) // share of NFT A belonging to Structure X equals to 35%
assert ( shareOf(Y, X) == 70% ) // share of Structure Y belonging to Structure X equals to 70%
```

### Principle 4
> The value of **NFS** is equivalent to weighted sum of values of its contained elements. 
```motoko
// Value of Structure X is the sum of values of its contained non-fungibles A and Y 
// multiplied respectively by the shares owned by X
assert ( ownedBy(X) == {A: 35%, Y: 70%} ) // sum doesn't have to be equal to 100%
assert ( valueOf(X) == valueOf(A) * 35% + valueOf(Y) * 70% )
```


### Principle 5
> Users can change their ownership share in the asset by trading it either on the market, or with a specific user, e.g. other coowner

```motoko

/// Place the request to buy 
/// if you buy from market, then seller = None
buy_req_id = requestToBuy (buyer: accountId, seller: ?accountId, asset: assetId, maxShareToBuy: Float, maxPrice: Float, coin: Text, expiration: DateTime)

/// Place the request to sell
/// if you sell to market, then buyer = None
sell_req_id = requestToSell (seller: accountId, buyer: ?accountId, asset: assetId, maxShareToSell: Float, minPrice: Float, coin: Text, expiration: DateTime)

// Accept to become the counterparty on the request
acceptRequest(req: requestId): await Result

/// Partially accept to become the counterparty on the request, but only to some extent
partiallyAcceptRequest(req: requestId, acceptedShare: Float): await Result

/// Cancel your own request
cancelRequest(req: requestId): await Result 

/// Check the status of the request
type RequestStatus: {#issued, #completed, #cancelled, #expired, #partially_completed, #rejected};  
statusOfRequest(req: requestId): await RequestStatus

/// Reject request, issued at you
rejectRequest(req: requestId): await Result

/// Check who issued the request
authorOfRequest(req: requestId): await Principal


```

### Principle 6
> NFS can't contain shares of itself or can't own shares of itself through intermediaries. Transactions, that lead to such circles will be rejected. This rule is enforced to ensure that there won't be any circle references when evaluating the price of a Structure.
 
### Principle 7

> In the graph of ownership the final beneficiary is always a user and never an NFS. This is to ensure that voting is always elevated to a person and is never stuck at NFS.  


### Principle 8

> NFS is mutable and can be changed in several ways.

Here's an example case:

|	↓ Owners	|	NFT A	|	NFS X	|	NFS Y	|	Coin P	|	Worth of owners, P	|
|	:--	|	:-:	|	:-:	|	:-:	|	:-:	|	:-:	|
|	User U	|	—	|	65%	|	—	|	30	|	70	|
|	User V	|	—	|	15%	|	55%	|	70	|	100	|
|	NFS X	|	60%	|	—	|	30%	|	42	|	62	|
|	NFS Y	|	35%	|	—	|	—	|	33	|	38	|
|	Others market	|	5%	|	20%	|	15%	|	—	|	19	|
|	Check sum ∑	|	100%	|	100%	|	100%	|	—	|	—	|
|	Value without coins, P	|	14	|	20	|	5	|	—	|	—	|
|	Coins, P	|	—	|	42	|	33	|	—	|	—	|
|	Value with coins, P	|	14	|	62	|	38	|	175	|	289	|


Here are what changes are valid:

1. Owners may change how much of non-fungibles they own by trading their shares. This transaction doesn't affect the internal structure of the traded assets. 

    For example, User U can buy 10% of NFS X from User V for 6 Coins P

|	↓ Owners	|	NFT A	|	NFS X	|	NFS Y	|	Coin P	|	Worth of owners, P	|
|	:--	|	:-:	|	:-:	|	:-:	|	:-:	|	:-:	|
|	User U	|	—	|	**75%**	|	—	|	**24**	|	71	|
|	User V	|	—	|	**5%**	|	55%	|	**76**	|	100	|
|	NFS X	|	60%	|	—	|	30%	|	42	|	62	|
|	NFS Y	|	35%	|	—	|	—	|	33	|	38	|
|	Others market	|	5%	|	20%	|	15%	|	—	|	19	|
|	Check sum ∑	|	100%	|	100%	|	100%	|	—	|	—	|
|	Value without coins, P	|	14	|	20	|	5	|	—	|	—	|
|	Coins, P	|	—	|	42	|	33	|	—	|	—	|
|	Value with coins, P	|	14	|	62	|	38	|	175	|	289	|


2. User can send belonging to her shares and/or coins directly to Structure, in order to increase her owner share in a Structure. 
   
    For example, User V sends to the Structure X 10 Coins P and 10% of NFS Y, belonging to her. This transaction changes the internal structure of X, increasing it's total value. 
   
    Step 1:

    |	↓ Owners	|	NFT A	|	NFS X	|	NFS Y	|	Coin P	|	Worth of owners, P	|
    |	:--	|	:-:	|	:-:	|	:-:	|	:-:	|	:-:	|
    |	User U	|	—	|	75%	|	—	|	24	|	**81**	|
    |	User V	|	—	|	5%	|	**45%**	|	**66**	|	**87**	|
    |	NFS X	|	60%	|	—	|	**40%**	|	**52**	|	**76**	|
    |	NFS Y	|	35%	|	—	|	—	|	33	|	38	|
    |	Others market	|	5%	|	20%	|	15%	|	—	|	**22**	|
    |	Check sum ∑	|	100%	|	100%	|	100%	|	—	|	—	|
    |	Value without coins, P	|	14	|	**24**	|	5	|	—	|	—	|
    |	Coins, P	|	—	|	**52**	|	33	|	—	|	—	|
    |	Value with coins, P	|	14	|	**76**	|	38	|	175	|	**303**	|


    Total Value transferred from User V to Structure Y equals 10 Coins P + 10% of NFS Y * valueOf(Y) = 10P + 10% ∙ 38P = 14P. The value of Structure X has increased from 62P to 76P.  
    
    Hence we must increase the value of User V by 14P by transferring corresponding amount of shares of Structure X to User V. 
    
    After the transaction is finalized 14P/76P = 18% of Structure X should be added to User V. This will result in dilution of other owners of Structure X shares. Dilution coefficient in this case equals to 0.81 = (100% - 18% - 5%) / (100% - 5%), where 5% — is the previous share of Structure X owned by User V. 

    Step 2: 
    Adding 18% of Structure X to User V and diluting the rest owners: 

    |	↓ Owners	|	NFT A	|	NFS X	|	NFS Y	|	Coin P	|	Worth of owners, P	|
    |	:--	|	:-:	|	:-:	|	:-:	|	:-:	|	:-:	|
    |	User U	|	—	|	**61%**	|	—	|	24	|	**70**	|
    |	User V	|	—	|	**23%**	|	45%	|	66	|	**101**	|
    |	NFS X	|	60%	|	—	|	40%	|	52	|	76	|
    |	NFS Y	|	35%	|	—	|	—	|	33	|	38	|
    |	Others market	|	5%	|	**16%**	|	15%	|	—	|	**19**	|
    |	Check sum ∑	|	100%	|	100%	|	100%	|	—	|	—	|
    |	Value without coins, P	|	14	|	24	|	5	|	—	|	—	|
    |	Coins, P	|	—	|	52	|	33	|	—	|	—	|
    |	Value with coins, P	|	14	|	76	|	38	|	175	|	303	|


   
3. User U can retrieve shares and/or coins directly from NFS X, so that her owner share in X to be decreased. This case is the opposite of the previous case. 

4. Previous two operation can happen not only with direct subsidiary, but also deeper in the ownership structure. 

5. Owner may request the Structure to buy or sell shares of assets. Structure uses it's own currency wallet balance to pay for purchases and/or receives proceeds from selling the assets to the Structure's wallet. 




```motoko
/// Change the structure of NFS







```



### Principle 9
> As long as user owns > 50% of an asset, it can solely make transactions. If user owns <= 50% of an asset, they can propose transaction for a vote with specified deadline, which waits until it reaches > 50% support. User may terminate the proposal before the trasnaction happened. User may vote to agree with the proposal. User may see the proposals of others and the support they gathered. Users may ask the voting be either anonymous or not.

```motoko
// Users transact or vote
assert ( shareOf(Y, X) == 70% ) // share of Structure Y belonging to Structure X equals to 70%
public shared(msg) func transferFrom(from : Principal, to : Principal, tokenId : Nat) : () 


```



### Principle 10
> Non-Fungible Structures are created with charter, which addresses fundamental questions of NFS lifecycle: 
1. Approval vote level (k%) required to make changes to charter
2. Approval vote level (k%) required to change the holders of the Structured NFT
3. Approval vote level (k%) required to change the structure of NFT
4. Approval vote level (k%) required to forced buyout of the Structured NFT from the rest of owners
5. Can you exit the ownership at your will or it requires approval from other owners and at what vote level (k%)
6. Can you burn your ownership of the Structured NFT at your will
7. Approval vote level (k%) required to destroy the Structured NFT
8. What happens to shares of NFTs, owned by sNFT, after sNFT is destroyed (burned, transferred to the owners of destroyed sNFT)

### Principle 11
> Non-Fungible Structures can be destroyed



### Principle 12
> Only Users are conscious to vote and make decisions. If a vote from NFS is required, then it's elevated up the ownership graph until Users are found. requestForVote received by NFS is propagated further up to the Users 

### Principle 13
> Users and NFS can have special voting rules, regarding any NFS, which belongs them directly or through intermediary NFS. 

### Principle 14
> Users may delegate their voting rights to other Users regarding any NFS, which belongs them directly or through intermediary NFS.

### Principle 15
> NFS has its own balance of coins, which allowed in this network (blockchain).



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