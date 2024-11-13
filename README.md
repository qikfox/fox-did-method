# Fox DID Method Specification

# Authors

* Smahi Maini (smahi@qikfox.com)
* Tarun Gaur (tarun@qikfox.com)
* Robert Wyatt Lutt (wyatt@qikfox.com)
* Hitesh Sevlia (hitesh.s@qikfox.com)
* Pawan Kumar (pawan.k@qikfox.com)

## 1. Introduction

The `did:fox` method is a decentralized identifier (DID) method that adheres to the W3C DID Core Specification. It operates on **Trustnet**, a modified version of the Hyperledger Indy distributed ledger, enabling the creation, resolution, and management of DIDs and cryptographic key-based authentication and verification. The method also introduces **Universal Handles**, human-readable aliases for DIDs that simplify online interactions and allow users to share personal information safely without revealing their primary DID.

Designed to be privacy-preserving and scalable through the use of a permissioned distributed ledger, the did:fox method is suitable for a variety of use cases, including decentralized digital identity management, credential issuance, and verification across hybrid & decentralized applications.

## 2. Fox Method DID Components

A DID on the Fox method has 3 components and follows the W3C DID Core Specification:

1. DID Prefix: The hardcoded string `did:` indicates that the identifier is a DID. This must always be lowercase.
2. Method Name: The hardcoded string `fox:` indicates the use of the Fox DID method. This must always be lowercase.
3. Method-Specific Identifier: A Base58-encoded string derived from the initial Ed25519 public key. See Section 3 for more details.

### 2.1 Example Fox DID
`did:fox:ALRFWtmnDFcAemPQxAKPvo`

## 3. Method-Specific Identifier Creation

The method-specific identifier is generated through the following steps:

1. Hash the public key: Compute the SHA3-256 hash of the entire Ed25519 public key (32 bytes).
2. Truncate the Hash: Take the first 16 bytes of the resulting 32-byte hash.
3. Base58 Encode: Encode the 16-byte truncated hash using Base58 for compactness and readability.

## 4. DID Document

DID documents in the Fox method conform to the W3C DID Core Specification. By default, the controller property is not included because the controller is the DID itself, specified in the id key of the DID document. However, entities may include the controller property if they wish.

### 4.1 Example DID Document
```json
{
   "@context":[
      "https://www.w3.org/ns/did/v1",
      "https://w3id.org/security/suites/ed25519-2020/v1",
      "https://w3id.org/security/bbs/v1",
      "https://w3id.org/security/suites/jws-2020/v1"
   ],
   "id":"did:fox:ALRFWtmnDFcAemPQxAKPvo",
   "assertionMethod":[
      "did:fox:ALRFWtmnDFcAemPQxAKPvo#key-ed25519-1",
      "did:fox:ALRFWtmnDFcAemPQxAKPvo#key-bbs-1"
   ],
   "authentication":[
      "did:fox:ALRFWtmnDFcAemPQxAKPvo#key-ed25519-1",
      "did:fox:ALRFWtmnDFcAemPQxAKPvo#key-bbs-1"
   ],
   "keyAgreement":[
      "did:fox:ALRFWtmnDFcAemPQxAKPvo#key-x25519-1-7W5z1KKR"
   ],
   "verificationMethod":[
      {
         "id":"did:fox:ALRFWtmnDFcAemPQxAKPvo#key-ed25519-1",
         "type":"Ed25519VerificationKey2020",
         "controller":"did:fox:ALRFWtmnDFcAemPQxAKPvo",
         "publicKeyMultibase":"z8oxPJKHj1LFaD4yYPo3CX5cjmNQuhs9KYbrSRYG9FaBc"
      },
      {
         "id":"did:fox:ALRFWtmnDFcAemPQxAKPvo#key-bbs-1",
         "type":"Bls12381G2Key2020",
         "controller":"did:fox:ALRFWtmnDFcAemPQxAKPvo",
         "publicKeyBase58":"uv6D9jLpq87ke3XJTQ1HL7neC52ZZ6662rVjdZhW9VHc28XuFxr7ye1mcdYLWA2KqVYUTFgqePeu4pLxtucXS6xtXU9j45Sh562ZeWfEAVRhgPHAQKFGyCvDqgvP3CsDTVR"
      },
      {
         "id":"did:fox:ALRFWtmnDFcAemPQxAKPvo#key-x25519-1-7W5z1KKR",
         "type":"JsonWebKey2020",
         "controller":"did:fox:ALRFWtmnDFcAemPQxAKPvo",
         "publicKeyJwk":{
            "kty":"OKP",
            "crv":"X25519",
            "x":"VxUX7bCIGphGwzCH7YInhU-JLjqY4LBOsojsrdmMjnY"
         }
      }
   ]
}
```

## 5. CRUD Operations
Because Trustnet is a permissioned ledger the Create, Update, and Delete operations can only be performed by authorized entities. However, there are ledger gateway services that provide RESTful APIs for these operations that can be accessed by any entity. These ledger gateways use DIDComm v2 requests and require the entity to sign requests with the private key associated with the DID for authentication and authorization purposes.

### 5.1 Create
To create and register a did:fox DID:

#### Generate Keys:
The entity generates an Ed25519 keypair at a minimum.

#### Create the DID:
Generate the DID as described in Section 3.

#### Assemble the DID Document:
Construct a DID Document following the W3C DID Core Specification, including the necessary verification methods and services.

#### Submit a Registration Request:
Send a `registerIdentity` request to an authorized entity's API, including the DID, public key, and DID Document. You can refer to the ledger gateway service's documentation for the specific request format.

The NYM transaction (as defined in the Hyperledger Indy specification) will be used by the authorized entity to create the DID on the ledger.

### 5.2 Read (Resolve)
Read operations can be performed by any entity to resolve a did:fox DID and retrieve the associated DID Document. Clients can use the Indy SDK to create a GET-NYM transaction or a ledger gateway service's `resolveDID` request can be used to resolve DIDs.

Ledger gateway services or clients using the Indy SDK utilize the GET-NYM transaction (as defined in the Hyperledger Indy specification) to fetch the DID Document from the ledger.

### 5.3 Update
To update a did:fox DID Document:

#### Modify the DID Document:
Make the necessary changes to the DID Document, such as adding or removing verification methods or services.

#### Submit an Update Request:
Send an `updateIdentity` request to a ledger gateway service. Please refer to the ledger gateway service's documentation for the specific request format.

The ledger gateway service will update the document using an NYM transaction (as defined in the Hyperledger Indy specification).

### 5.4 Deactivate
Direct deletion of a DID is not supported. However, a DID can be deactivated by:

#### Update the DID Document:
Modify the DID Document to remove all verification methods and services, effectively rendering the DID inactive.

#### Submit the Deactivation Request:
Send an `updateIdentity` request with the modified DID Document to a ledger gateway service. Please refer to the ledger gateway service's documentation for the specific request format.

The ledger gateway service will update the document using an NYM transaction (as defined in the Hyperledger Indy specification), effectively making the DID inactive on the ledger.

## 6. Universal Handles

Universal Handles are a concept introduced in Trustnet and are an effective replacement for DNS-based domain names, thereby providing a brand-new method for discovering content on both the centralized and decentralized web. The handles are registered on the Trustnet ledger, and each handle is associated with a DID, making them irrevocable (similar to blockchain domains). Handle entries are automatically picked up by QNS (a brand-new graph-based replacement for DNS) and can be associated with IP addresses, CIDs, application links, or GUIDs. These associations help others discover you using your handle, making handles a reliable secondary method for discovering friends, websites, and content on the next-generation web. This method is not biased in favor of either the centralized or decentralized web and can be used to discover content in all its manifestations.

### 6.1 Handle Registration

Handles can be purchased from authorized retailers online that are Trustnet endorsers. During the purchase process, the entity will generate a new DID to be associated with the handle and submit a `registerHandle` request to the issuer's ledger gateway service API. Please refer to the issuer service's documentation for the specific request format.

## 7. Security and Privacy Considerations
The did:fox method is designed with the following security and privacy considerations:

#### Permissioned Ledger:
Trustnet is a permissioned ledger, ensuring that only trusted entities write to the ledger and participate in consensus.

#### Data Minimization:
Only essential information is stored on-chain, minimizing exposure of sensitive data.

#### Cryptographic Security:
Utilizes strong cryptographic algorithms (Ed25519, BBS+, & X25519) to ensure the integrity and authenticity of DIDs and their associated documents.

#### Privacy-Preserving:
Entities control their DIDs and associated keys, enabling self-sovereign identity management.
