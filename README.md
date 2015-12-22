# Publishing Blockchain Specification

## Overview

### Executive Summary

The Publishing Blockchain is a decentralized database of completed resources.  The blockchain contains metadata about these resources as well as digital signatures of people and organizations that approve of them.  The database is ideal for publishers of all sizes and developers to pull resources from to input into their pubishing engines.

### Technical Summary

New transactions are input through an existing `node` that performs basic verification of the record before adding it to the blockchain.  Once the first node approves of the transaction, it is then timestamped and broadcast to all other nodes.  Once 3 (or perhaps 10% of the) `nodes` have also verified the transaction it becomes an immutable part of the blockchain.  Transactions can be signed by other entities once they are a part of the blockchain.  Further, new URIs can be added as pointers to content that exists in the blockchain.  The actual content of the resources are never stored in the database, rather the hash and URIs are what get stored in the database.

<img src="https://docs.google.com/drawings/d/1KobvkGFYO16XAC7DBh7Xc5tkhkeBpXfMV8-dUgF-3bo/pub?w=910&amp;h=647">

## Blockchain Structure
*Service app name: node*

### Database Structure
Each entry in the blockchain needs the following fields:

 - `Timestamp` - Created by `node` when transaction is first accepted.
 - `SHA512 Content Hash` - The hash of the content that is referenced by the URI(s).
 - `Signature of Content Hash` - The requester's signature of the content hash.
 - `Signer ID` - The identifier for the signer (possibly superseded by a signature that includes identity information).
 - `URI(s)` - The location(s) that the content is available from.  Any location that is accessible by the node is allowed.
 - `Related Content` - The type of relationship and the content hash of related content (e.g. `revision`, `translation`, `format-shift`, etc.).
 - `Resource Name` - The name of the resources.  `clients` should offer predefined lists of resource names to be chosen from.
 - `IETF Language Code` - A well formed IETF language code.
 - `License` - The license of the resource being referenced.
 - `Format` - The format of the resource.  `clients` should handle this as much as possible for the user, possible examples: `USFM`, `USX`, `JSON`, `mp3`, `mp4`, `PDF`.

Notes on structure:
 - A record is expected to be `1k` or less, so the total blockchain size will be about 100MB when there are 100,000 entries.
 - Resource content is never stored in the blockchain, only metadata
 - Any format of resource is accepted into the blockchain, but publishers will likely only be tooled for publishing certain types.
 - Versioning of content is managed entirely by timestamps and the `Related Content` field.  Publishers are free to implement different versioning systems based on this if they so choose (for example: https://unfoldingword.org/versioning/).

Example entries in the blockchain:
https://docs.google.com/spreadsheets/d/1t4b1-fRkKWGZSE7HH5oFUsEr7QeaXl51fOC2RTOlc90/

### Service Description
Each node in the Publishing Blockchain follows this procedure for accepting or rejecting a new transaction request:

1. Does the `signature` verify?
 - No → `Reject`
 - Yes → `Continue`
2. Is the `content hash` in the catalog already?
 - No → `Continue`
 - Yes → Process as either a new `signature` or a new `URI` addition
3. Is the content available at 1 or more `URI`s?
 - No → `Reject`
 - Yes → `Continue`
4. Does the content at the `URI`s match the hash?
 - No → `Reject`
 - Yes → `Continue`
5. Add `timestamp` and add to blockchain.
6. Notify other `nodes` of new transaction.


## Identities
*Service app name: client*

### Client Functions

Clients are expected to offer one or more of the following services:

 - Data entry services (content creation, content translation, etc)
 - Identity management (signing keys)
 - Content upload services
 - Transaction requests to the Publishing Blockchain when content is completed
 - Publishing Blockchain browsing

### Identity Management

Clients that offer identity management need to implement the following features:

 - Management of signing keys
 - Management of trusted relationships
   - Mutual friendship relationships
   - One-way trust relationships (User X trusts User Y)
   - Language based trust relationships (User X trusts User Y for Spanish content)
 - Verification of node identity (possibly implemented as badges)
   - Email confirmation (required)
   - Twitter/Facebook/Google+ confirmation
   - Phone number/text message verification
   - Physical address verification? (any reason for this?)
   - How to handle organization verification?
     - Domain verification
     - Facebook/Twitter/Google+ organization page verification?
     - Trust relationships with others?

The forthcoming http://keybase.io service may be an ideal solution.


## Data
*Service app name: bucket*

### Bucket Functions

Although the blockchain `nodes` themselves do not mirror content at all, operators in the network may choose to run a `bucket` to mirror some or all of the content referenced in the blockchain.  These `bucket` processes are intended to be super-simple wrappers that do the following functions:

 - Read the blockchain
 - Mirror specified locations
 - Submit new `URI` entries to the blockchain

### Configurable Mirroring

The mirroring should be configurable based on the following operator defined criteria:

 - All data for a given language code (e.g. 'es' or 'es*').
 - All data from my trusted friends (see Identities above)
 - All data with a certain number of `signatures`
 - All data with a certain `license`
 - Any combination of the above
 - All data

Note that operators running a bucket are liable for the content that they mirror and may be subject to take down notices or other legal ramifications.  Transactions in the blockchain cannot be guaranteed to be unencumbered from legal issues, so bucket operators have the greatest risk of the three services.
