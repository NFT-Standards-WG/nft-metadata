---
eip: <to be assigned>
title: JSON Metadata Standard
description: a specification for JSONs of metadata
author: <a list of the author's or authors' name(s) and/or username(s), or name(s) and email(s), e.g. (use with the parentheses or triangular brackets): FirstName LastName (@GitHubUsername), FirstName LastName <foo@bar.com>, FirstName (@GitHubUsername) and GitHubUsername (@GitHubUsername)>
discussions-to: <URL>
status: Draft
type: <Standards Track, Meta, or Informational>
category (*only required for Standards Track): <Core, Networking, Interface, or ERC>
created: <date created on, in ISO 8601 (yyyy-mm-dd) format>
---

## Abstract

A specification (in TypeScript) of a JSONs representing metadata, popularized in NFTs, but also usable in other usecases. The intent is to cover a wide variety of applications and usecase; while it is unlikely any particular NFT would use _all_ fields specified, the idea is to provide a standard for major usecases to converge on.


## Motivation

JSONs of metadata stored off-chain have become a popular way of referencing metadata, and have been popularized by the two foundational NFT standards, [EIP-721](./eip-721.md) and [EIP-1155](./eip-1155.md), both of which include an optional metadata section. The sections in these EIPs are brief and do not cover many of the current data fields NFTs would like to leverage.

The current state of metadata cases not outlined in the standards is somewhat chaotic. Some projects will implement their own bespoke key names, some platforms have their own unique metadata requirements including keys not standardized in the EIPs mentioned above.

By specifying major use cases we can aid interoperability, and give both platforms and artists/creators the assurance of a standardized API for metadata.

## Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.

Any EIP-<TBD>-compliant JSON referncing information MUST use the nomenclature of EIP-<TBD>. An EIP-<TBD>-compliant JSON MAY also include additional key/value pairs not referenced in EIP-<TBD>.

Code other than JSON will also be considered EIP-<TBD>-compliant if it produces an EIP-<TBD> compliant JSON when queried. This is of use especially for smart contracts which store metadata locally (on-chain), and return a valid JSON when queried (as is the case for many generative NFT contracts).

```typescript
/**
 * A URI is a string pointing to a location, it can be using one of the many
 * schemes available [1]. Using http(s), data or ipfs URI schemes should
 * provide the best results in terms of compatibility.
 * */
export type URI = string;

/**
 * An object representing a resource. `url` points to its location, and `type`
 * could be any valid Media Type [1].
 *
 * [1] https://www.iana.org/assignments/media-types/media-types.xhtml
 */
type MediaTypeUrl = {
  url: URI;
  type: string;
};

/**
 * Represents the possible sizes of a visual asset (image or video). It can
 * take a single resolution or multiple, if supported by the format (e.g. .ico
 * files support multiple sizes).
 *
 * Examples:
 *
 * - "1280x720"
 * - "72x72 96x96 128x128 256x256"
 */
type Sizes = string;

/**
 * Represents an image asset. It could be a URI as a string, or a MediaTypeUrl
 * with an additional `sizes` field.
 */
export type Image = URI | (MediaTypeUrl & { sizes: Sizes });

/**
 * Represents a video asset. It could be a URI as a string, or a MediaTypeUrl
 * with an additional `sizes` field.
 */
export type Video = URI | (MediaTypeUrl & { sizes: Sizes });

/**
 * Represents an audio asset. It could be a URI as a string, or a
 * MediaTypeUrl.
 */
export type Audio = URI | MediaTypeUrl;

/**
 * Represents an asset of any type. It could be a simple URI as a string, or a
 * MediaTypeUrl.
 */
export type Asset = URI | MediaTypeUrl;

interface NftMetadataBase {
  /** Identifies the JSON schema version */
  version: "nft-media-types:1";

  /** Identifies the asset to which this NFT represents */
  name: string;

  /** Describes the asset to which this NFT represents */
  description: string;

  /** A URI pointing to a resource this NFT represents */
  url?: URI;

  /**
   * A name that represents the author of the NFT. It could be a person or an
   * entity.
   */
  author_name?: string;

  /**
   * A URI pointing to a resource representing the author of the NFT, which
   * could be a person or an entity.
   */
  author_url?: URI;

  /**
   * A name representing the minter of the NFT. For example, it could be the
   * name of a service used to mint the NFT.
   */
  minter_name?: string;

  /** A URI pointing to a resource representing the minter of the NFT.*/
  minter_url?: URI;

  /** Arbitrary properties. Values may be strings, numbers, object or arrays. */
  properties?: Record<string, string | number | object | Array<unknown>>;
}

export interface NftMetadataImage extends NftMetadataBase {
  /**
   * Use this type to indicate that the NFT is mostly representing an image
   * assset.
   */
  type: "image";

  /**
   * Either a URI pointing to a resource with mime type image/* this NFT
   * represents (see EIP-721), or an Image object, or an array of Image objects
   * to provide alternative formats.
   */
  image: Image | Image[];
}

export interface NftMetadataVideo extends NftMetadataBase {
  /**
   * Use this type to indicate that the NFT is mostly representing a video
   * assset.
   */
  type: "video";

  /**
   * Either a URI pointing to a resource with mime type image/* this NFT
   * represents (see EIP-721), or an Image object, or an array of Image objects
   * to provide alternative formats.
   *
   * The image should NOT be a video file but a static image representing it.
   * For example, this image could be used by video players before they start.
   */
  image: Image | Image[];

  /**
   * Either a URI pointing to a resource with mime type video/* this NFT
   * represents, or an Video object, or an array of Video objects
   * to provide alternative formats.
   */
  video: Video | Video[];
}

export interface NftMetadataAudio extends NftMetadataBase {
  /**
   * Use this type to indicate that the NFT is mostly representing an audio
   * asset.
   */
  type: "audio";

  /**
   * Either a URI pointing to a resource with mime type image/* this NFT
   * represents (see EIP-721), or an Image object, or an array of Image objects
   * to provide alternative formats.
   *
   * For example, this image could point to an album cover corresponding to the
   * audio file.
   *
   * This is an optional field.
   */
  image?: Image | Image[];

  /**
   * Either a URI pointing to a resource with mime type audio/* this NFT
   * represents, or an Audio object, or an array of Audio objects
   * to provide alternative formats.
   */
  audio: Audio | Audio[];
}

export interface NftMetadataOther extends NftMetadataBase {
  /**
   * Use this type to indicate that the NFT is representing something that is
   * not an image, audio or video.
   */
  type: "other";

  /**
   * Either a URI pointing to a resource with mime type image/* this NFT
   * represents (see EIP-721), or an Image object, or an array of Image objects
   * to provide alternative formats.
   *
   * This is an optional field.
   */
  image?: Image | Image[];

  /**
   * Either a URI pointing to a resource this NFT represents, or an Asset, or
   * an array of Asset objects to provide alternative formats.
   */
  asset: Asset | Asset[];
}

export type NftMetadataMediaTypes =
  | NftMetadataImage
  | NftMetadataAudio
  | NftMetadataVideo
  | NftMetadataOther;

```

## Rationale



## Backwards Compatibility

The specification is compatible with the metadata standards in [EIP-721](./eip-721.md) and [EIP-1155](./eip-1155.md), and is a superset of them.

This standard does use a different key for video content than OpenSea as of this writing. The specified key here is `video`, whereas OpenSea uses `animation_url` ([source](https://docs.opensea.io/docs/metadata-standards#metadata-structure)). If compatability with OpenSea is desirable, an `animation_url` key/value pair can be added to the JSON.


## Test Cases

As the suggested standard is a fairly straightforward JSON, there is no test suite. The specification is written in valid TypeScript, and can be compiled.

## Reference Implementation

(optional)

## Security Considerations



## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).