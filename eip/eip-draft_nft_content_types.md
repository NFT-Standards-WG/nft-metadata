---
eip: <to be assigned>
title: Identify NFT Content Type
description: NFT contracts to provide a contentType getter so clients can identify the type of NFT and how to fetch metadata
author: Louis Garoche <louis@garoche.me>
discussions-to: https://www.nftstandards.wtf
status: Draft
type: Standards Track
category: ERC
created: 2021-08-30
---

## Abstract
Define a standardized way to identify the type of an NFT. The contract provides a MIME-like Content Type description field queried by clients (wallets, marketplaces...) to render it as it was designed.

## Motivation
The ERC721/1155 standard defines a tokenURI/uri getter on the contract, pointing to a JSON document containing the NFT metadata according to a conventional schema. This works for many audio/image/video projects but this scheme falls short when a creator wants to combine on-chain and off-chain data. Even for fully on-chain data, building JSON documents from the smart contract code is quite costly. <br>
Every project has its own metadata requirements, and this EIP aims to unleash the creativity of NFT creators, by providing a common base to describe NFT metadata type.<br>
The current conventional NFT format can be described as one of the possible metadata format and becomes the first entry in the NFT format registry.
```TODO: should we start and maintain a registry?```

## Specification
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.<br>

Smart Contracts implementing this ERC MUST implement the following interface. 

```solidity
pragma solidity ^0.8.0;

interface IERCContentType {
    
    /**
        @dev The metadata type magic bytes of the token given by `tokenId` 
        The returned value MUST be the first 4 bytes of the keccak256 hash of the type name.
        The contract MAY return the same value for all tokens.
    */
    function contentType(uint256 tokenId) external view returns (bytes4);
}
```

It is RECOMMENDED that Smart contracts implementing this ERC also implement the ERC-165 interface, with the interface id ```0x77ad916e```, which is the result of ```bytes4(keccak256('contentType(uint256)')```.

The content type MUST be the first 4 bytes of the keccak256 hash of the type name. <br>

For example, a token that implements the existing [ERC-1155 metadata JSON schema](https://eips.ethereum.org/EIPS/eip-1155#metadata) as-is SHOULD implement the following: 
```solidity
pragma solidity ^0.8.0;

import "./IERCContentType.sol";

abstract contract ERC1155Example is IERCContentType /* , ERC1155Metadata */ {
    
    /**
        @dev The ERC-1155 metadata standard content type (0x4e0268aa)
    */
    bytes4 constant ERC1155_CONTENT_TYPE = bytes4(keccak256("json/erc1155"));
    
    function contentType(uint256 tokenId) public view override returns (bytes4) {
        return ERC1155_CONTENT_TYPE;
    } 
}
```
Clients will know that this NFT implements the ERC-1155 metadata standard, and that the URI to fetch the metadata JSON can be queried by the ```uri(uint256 tokenId)``` function.

Content types that match an existing [IANA Media Type](https://www.iana.org/assignments/media-types/media-types.xhtml) SHOULD implement a ```content(uint256 tokenId) public view returns (bytes memory)``` function that returns the raw content OR a ```contentURI(uint256 tokenId) public view returns (string memory)``` function that returns the off-chain location. (```TODO: should we strictly define this?```)

## Rationale
This design is inspired from the [IANA Media Types (formerly known as MIME types)](https://www.iana.org/assignments/media-types/media-types.xhtml), the same way an operating system recognizes supported formats with the available installed software, wallet or marketplace clients can check if they "support" a given NFT. The document does not intend to describe all possible NFT formats but provide an effective way to identify the NFT type. Creators are free to imagine new content types that provide additional on-chain or off-chain metadata content.

### Type value
The ```bytes4``` of a ```keccak256``` idea was taken from EIP-165. Content types could be returned as a ```string memory```, but using magic numbers may allow for gas optimization.

### Additional example
A NFT solely consisting of HTML content can be created with the following implementation:
```solidity
pragma solidity ^0.8.0;

import "./IERCContentType.sol";

abstract contract HtmlNFT is IERCContentType {
    
    /**
        @dev HTML content-type
    */
    bytes4 constant HTML_CONTENT_TYPE = bytes4(keccak256("text/html"));
    
    function contentType(uint256 tokenId) public view override returns (bytes4) {
        return HTML_CONTENT_TYPE;
    }
    
    /**
        @dev On-chain HTML content (actual implementation omitted)
    */
    function content(uint256 tokenId) public view returns(bytes memory) {
        return _content(tokenId);
    }
}
```

### New custom type
A creator may submit his own type specification (```TODO: where?```) that describes custom getters for on-chain data and URI schemes for off-chain data.

## Backwards Compatibility
This EIP does not introduce any backward compatibility with existing NFT standards. It extends these standards by adding the possibility of having additional metadata. 
NFT creators may keep the existing ERC721/1155 metadata URI schemes on top of new content types to preserve compatibility with existing client implementations (it could be used as a fallback). 

## Reference Implementation
See the example in the Specification section.

## Security Considerations
Clients that decide to support and implement executable content (binary, javascript...) must ensure that proper security measures are in place before running any code on the client.  

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
