A proposal to improve the NFT metadata standards.

```ts
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
