<pre>
id: 1
title: NFT Image Specification
author: Ryota Uno (@palon7)
status: Review
type: Standards Track
created: 2021-03-29
</pre>

# NFT Image Specification

## Notice

This proposal is in the review stage. Discussion and feedback are widely welcome.

## Abstract

This document proposes a Monaparty standard for metadata using an asset's description.
This standard provides basic functionality for linking assets to an image.

This document does not address how to ensure the persistency or reliability of images.

## Motivation

As seen in the popularity of NFT art, tokens tied to images are an essential part of the token platform. However, Monaparty does not have a standard that allows free-form image data to be linked to tokens. Monaparty devs planned the [XMPIP-0019](https://github.com/monaparty/XMPIP/blob/master/XMPIP-0019.md) standard to include metadata in assets, but it is not implemented yet.

On the other hand, [Monacard](https://card.mona.jp/) contains metadata in JSON in the asset's `description` field. Although this method results in an increased transaction size at the time of issuance, it is not considered a problem at the current Monacoin fee level.
Therefore, we propose a standard for attaching free-form images to Monaparty assets using the same method.

## Terms

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## Specification

Assets compliant with this standard MUST include JSON compliant with the following JSON Schema as the asset's `description`.
The JSON MAY include properties that are not specified in this standard.

The asset `description` MUST NOT contain strings that cannot be parsed as JSON.

### JSON Schema

```json
{
  "type": "object",
  "properties": {
    "image": {
      "type": "object",
      "properties": {
        "name": {
          "type": "string"
        },
        "desc": {
          "type": "string"
        },
        "cid": {
          "type": "string"
        }
      },
      "required": ["name", "cid"]
    }
  },
  "required": ["image"]
}
```

### JSON Example

```json
{
  "image": {
    "name": "Monacoin-chan",
    "desc": "This is the NFT of Monacoin-chan.",
    "cid": "bafkreifzjut3te2nhyekklss27nh3k72ysco7y32koao5eei66wof36n5e"
  }
}
```

### name

Name identifying the asset.

### desc

A description of the asset. This item is OPTIONAL.

### cid

The cid on the IPFS of the image file is associated with the asset. You SHOULD use CID v1.

The MIME type of the image file MUST be `image/*`. You SHOULD use the `image/png`, `image/apng`, `image/jpeg`, or `image/gif` format. For other formats, the application MAY refuse to display the image.

Images SHOULD be at least 300px on the short side and no larger than 1920px on the long side.

## Rationale

### IPFS

This standard requires that images be stored only in IPFS. This limitation is because IPFS has the following advantages:

- Because it is a decentralized system, it is unnecessary to depend on a specific server/individual/company.
- Although pinning is required to retain data, anyone can do this, not just the asset's issuer.
- It is impossible to rewrite image data off-chain since file-dependent IPFS hash values.
- Once the file was deleted on IPFS, it is possible to prove the link between the asset and the image again by re-uploading the original file if it is still available.

### Image Format

Since this standard assumes general image files, the MIME-type is limited to `image/*`. Since `image/png`, `image/gif`, and `image/jpeg` are widely used on the Internet, `image/apng` is one of the most popular formats for high-quality animation in many environments; this standard recommends these formats.

## Considerations

### Losing Files on IPFS

This standard limits the destination of images to storage on IPFS, but that files on IPFS will disappear unless they are correctly pinned.

However, the same problem is common to NFTs such as Ethereum, and this risk could occur even when stored outside of IPFS.

### Malicious SVG File

SVG files can embed Javascript, and there is a risk of XSS if the user opens the SVG file directly.
If the image is served from the same origin as the application, the attacker could steal sensitive data (such as cookies).

If implemented according to this standard, it is necessary to avoid providing SVG images from the same origin, not displaying SVG images, and alert the user to this risk.

See also: v4.0.3-5.2.7 in [OWASP ASVS 4.0](https://github.com/OWASP/ASVS/blob/v4.0.3/4.0/en/0x13-V5-Validation-Sanitization-Encoding.md)

### Change the Asset Description

Currently, the owner can change the asset's description by `issuance` message.
The history of the change remains on the chain, but a simple implementation may not be able to detect this, and the image data of the assets it holds may appear to have been rewritten.

To prevent rewritten images:

- The issuer should transfer the ownership to the burn address (such as `MMonapartyMMMMMMMMMMMMMMMMMMMUzGgh`) in advance to prove that issuer will not rewrite the contents.
- The application should search the asset's `issuance` history when referencing image data to ensure that the image in the description has not changed.

## Copyright

This document publishes under [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
