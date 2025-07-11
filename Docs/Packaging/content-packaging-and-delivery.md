---
title: Content Packaging and Delivery
description: The basic capability of PlayReady is to protect content from unauthorized use.
ms.assetid: "7E87CB5D-59E1-4E25-8271-82FB97998ECD"
keywords: playready content encryption, encryption and delivery
ms.date: 02/01/2018
ms.topic: article
---


# Content Packaging and Delivery


The basic capability of PlayReady is to protect content from unauthorized use. To do this, your content must first be encrypted, and an associated PlayReady Header be inserted in the content. The system that does this operation is the packager, also known as the encryptor, which is sometimes integrated with the encoder. 

This topic describes various ways to encrypt and deliver your content using PlayReady.

## Packaging PlayReady content &mdash; Encrypting and inserting the DRM header 

The process of encrypting clear content consists of defining one or several encryption keys, using these keys to encrypt the bytes that constitute the content itself, and inserting a DRM header in the content (in the files of the content, or in the manifest if the content has one).

All encrypted content protected by PlayReady must have a PlayReady Header inserted in the encrypted file. This PlayReady Header is used by a PlayReady Client to locate or acquire a license for that particular piece of content. A PlayReady Header is composed of XML that is encoded in UTF-16. It includes the key identifiers (KIDs) that are used to encrypt the content, a default URL that the client will use to acquire a license from if no other is provided, and any custom attributes.  

Any packager that packages clear content needs to implement a PlayReady Header generator to build the header and embed it in the encrypted content. The PlayReady Header must be implemented according to the [PlayReady Header Specification](../Specifications/playready-header-specification.md). There are multiple ways to create a PlayReady Header generator in your packager: 

  *  Develop it yourself based on the *PlayReady Header Specification*. 
  *  Use the PlayReady Server SDK API that generates a PlayReady Header. 
  *  Use the Windows 10 API that generates a PlayReady Header. 

Your encrypted content can contain multiple DRM headers, including PlayReady Headers along with third-party DRM headers. For more information on how this works, see [Using encryption tools](#encryptiontools).

### Content Type
PlayReady can be used to protect audio and video content. The most common types of encoding used with PlayReady are MPEG-4 AVC (H.264), High Efficiency Video Coding (HEVC) H.265 standards, and the AV1 standard. PlayReady is not limited to these standards and can be used with any audio and video format that is supported on the client device.

PlayReady version 1 and 2 allows you to create a protected package containing content that is not limited to audio or video payloads. These packages, referred to as envelopes, can contain files such as a collection of data files and executables (for example, an application distributed by an application store), pictures (for example, screen wallpaper), or ebooks. This content is packaged by encapsulating the files into an envelope file, which can be decrypted in a manner similar to audio/video content.

These non audio/video content types are no longer supported in PlayReady 3.0 and later. 


## Encryption tools

Microsoft does not include a packager as part of the PlayReady deliverables. PlayReady instead provides specifications based on common encryption standards for use by encoders. Therefore the encryption format is not PlayReady specific, rather it's a function of the file format. The most widely used encryption format today is the Common Encryption ISO Standard format, **ISO/IEC 23001-7**.

Basically, you could either create your own packager, or you could work with any type of open source encryptor (such as ffmpeg). In addition, you could work with a professional encoder company if you want to encrypt content with PlayReady (such as Harmonic, Elemental, Ericsson, Wowza, Allegro). Azure Media Services also provides a packaging functionality for clear content.

<a id="encryptiontools"></a>

## Using encryption tools

PlayReady supports the ISO IEC common encryption standard. This process is the same as described in [Basic encryption and licensing process](../Overview/simple-end-to-end-system.md#basicprocess), except headers will be included for other DRMs &mdash; each as the payload of the Protection System Specific Header (‘pssh’) box, identified by that DRM’s SystemID. All those headers will have their own syntax that designates the KIDs or the information required to ultimately access the content keys. And the content keys for this asset are going to be the same for all the DRMs.

![Common Encryption Diagram](../images/common_encryption_diagram.png)

## Using encryption keys

There are many different ways to encrypt your assets. The simplest one to the most sophisticated one depends on how much complexity you want to design in the system and what the needs of the service are.

Let's take for example, an adaptive streaming asset, as shown in the figure below. It has four different video qualities, one audio track, and one subtitle track. It is encoded in segmented MP4 files, with segments of 2.0 seconds each. It is one asset that is served in multiple formats depending on what the client would prefer to play back. Smooth Streaming, HLS, and DASH are the most common variants. During playback, the client (the video player) is going to successively download the segments of the asset over the network, selecting for each playback time the video segment from the adequate video track, in order to keep playback quality as high as possible, given the constraints of the network bandwidth, the playback speed, and other limited resources like the player capabilities. This logic is known as adaptive streaming playback, governed by some heuristics rules implemented in the player. 

![Content Assets and Playback](../images/content_assets_and_playback.png)

### Encrypting the asset with just one key

The simplest way to encrypt these assets would be to use a single content key to encrypt everything (typically subtitles are not encrypted &mdash; it's not against any rule, but they are usually kept in the clear). Using one content key makes life easy for the License Server because the License Server has to deliver one key {KID, CK}. This key would typically be acquired by the client before playback occurred.

![Content Assets and Encryption Keys (I)](../images/assets_and_encryption_keys_1.png)


>[!NOTE]
>PlayReady Clients can acquire licenses proactively or reactively. See the [License Acquisition](../Overview/license-acquisition.md#proactivereactive) page for a description of these two modes.


### Encrypting the asset with two keys, dedicating one to the highest quality 

There have been some enhancements in the past years to use multiple keys per asset, mostly driven by the requirement to allow only certain highest-robustness clients to consume the highest quality content. With the arrival of Ultra HD (4K) content, and with the addition of high dynamic range (HDR) for higher color content, there was a need by studios and services to allow the highest quality only on certain clients, which typically have hardware DRM built in. In this scenario, the asset is encrypted using one content key {kid1, ck1} for all the tracks, except for the 4K track that is encrypted using a different content key {kid2, ck2}. That is:

  *  A client that is allowed to play only up to Full HD (not the 4K track) will be delivered a PlayReady license including only {kid1, ck1}. 
  *  A client that is allowed to play up to 4K will be delivered a PlayReady license including {kid1, ck1} and {kid2, ck2}.

Using this additional complexity, the service can ensure that some clients will not be able to decrypt the 4K track, and that 4K track can be reserved to only the clients that the service most trusts. 

![Content Assets and Encryption Keys (II)](../images/assets_and_encryption_keys_2.png)

### Encrypting the asset with one key per track

The service may have a more complex map of rights to enforce. Some clients, depending on their screen size, their robustness, their outputs, and their location, may be allowed to access only some video tracks, some video qualities, and some audio tracks. To ensure the service has full flexibility in enforcing an arbitrary set of restrictions in the future, it may encrypt an asset with a content key specific to each track. For example:

  *  A client that is allowed to play only 720p will be delivered a PlayReady license including {kid1, ck1}, {kid2, ck2}, and {kidA, ckA}. 
  *  A client that is allowed to play up to 4K will be delivered a PlayReady license including {kid1, ck1}, {kid2, ck2}, {kid3, ck3}, {kid4, ck4}, and {kidA, ckA}. 
  *  A client playing offline the 4K version of the asset (previously downloaded) will be delivered a PlayReady license including {kid4, ck4} and {kidA, ckA}. 

![Content Assets and Encryption Keys (III)](../images/assets_and_encryption_keys_3.png)

### Changing the encryption keys periodically (multi-period asset)&mdash;license rotation 

In some scenarios, the service wants to change the encryption keys occasionally, typically at program boundaries. For example, a live linear stream has multiple periods with free to air content that you want everyone to have access to, followed by some content that is restricted to subscribers. Changing the encryption keys at program boundaries allows the service to deliver the free to air keys {KIDi1, CKi1} to all users without any restrictions, and deliver the content keys {kidi2, cki2} only to the subscribers that have successfully logged in the service.

Note that this license rotation is not very scalable: every time the encryption keys change, all clients request the new encryption keys using their own license request. This can result in a high peak of license requests in systems with a large number of clients. 

![Content Assets and Encryption Keys (IV)](../images/assets_and_encryption_keys_4.png)

### Changing the encryption keys frequently&mdash;scalable key rotation 

There is an advanced mechanism in PlayReady called scalable key rotation (as opposed to license rotation). This method stores an Embedded License Store (ELS) in the stream of the actual content. In this mechanism, the key used to encrypt the A2 segment itself is called the leaf key {kidA2, ckA2}, and is delivered in the ELS of the segment A2, being itself encrypted with a separate key that is the same for all the segments of track A, called the root key {kidRA, ckRA}. If you are familiar with MPEG-2 TS and the Control Word encryption, this is a similar mechanism except for the encryption is much stronger and is also more flexible.

Let's say this asset is live linear TV. When the client attempts playback, it finds kidRA in the PlayReady Header of the stream manifest, and requests a license for kidRA. The License Server returns a root license for the root key {kidRA, ckRA}. Then the client parses segment A1 and discovers the ELS in the header of the segment. Parsing this ELS, it finds the leaf license {kidA1, ckA1} in this ELS. Using the root key {kidRA, ckRA} and the leaf license {kidA1, ckA1}, it can get the value of ckA1, and decrypt and render the segment A1. 

The PlayReady scalable key rotation feature is extremely scalable because it does not require clients to contact the License Server every time the encryption keys are changed. It keeps the volume of license requests to the lowest possible, as a client only needs one root license from the License Server per stream, or track. It allows encryption keys to rotate as frequently as every segment, typically every two seconds if necessary. 

![Content Assets and Encryption Keys (V)](../images/assets_and_encryption_keys_5.png)


## See also

[Key and Key IDs (KIDs)](../Overview/key-and-key-ids-kids.md)
