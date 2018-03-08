# Decentralized Autonomic Data (DAD)

Samuel M. Smith Ph.D. (sam@samuelsmith.org)  and Vishal Gupta (vishal@diro.io)

2018/03/07

## Abstract

This paper proposes a new class of data called *decentralized autonomic data* (DAD). The term *decentralized* means that the governance of the data may not reside with a single party. A related concept is that the trust in the data provenance is diffuse in nature. Central to the approach is leveraging the emerging [*DID*](https://w3c-ccg.github.io/did-spec/) (decentralized identifier) standard. The term *autonomic* means self-managing or self-regulating. In the context of data we crystalize the meaning of self-managing to include cryptographic techniques for maintaining data provenance that make the data self-identifying, self-certifying, and self-securing. Implied thereby is the use of cryptographic keys and signatures to provide a root of trust for data integrity and maintain that trust over transformation of that data, e.g. provenance. Thus key management must be a first order property of DADs. This includes key reproduction, rotation, revocation, and recovery.

The motivating use of DAD is to provide provenance for streaming data that is generated and processed in a distributed 
manner with decentralized governance. Streaming data are typically measurements that are collected and aggregated to form higher level constructs. Applications include analytics and instrumentation of distributed web or internet of things (IoT) applications. Of particular interest is the use of DADs in self-sovereign reputation systems. A DAD seeks to maintain a provenance chain for data undergoing various processing stages that follows diffuse trust security principles including signed at rest and in motion. 

Streaming data applications may impose significant performance demands on the processing of the associated data. Consequently one major goal is to use efficient mechanisms for providing the autonomic properties. This means finding minimally sufficient means for managing keys and cryptographic integrity.

Importantly this paper provides detailed descriptions of the minimally sufficient means for key reproduction, rotation, revocation, and recovery for DID leveraged DADS. 



## Overview

A decentralized autonomic data (DAD) item is associated with a decentralized identifier, ([DID](https://w3c-ccg.github.io/did-spec/)). This paper does not provided a detailed definition of DIDs but does describe how DIDs are used by a DAD. The DID syntax specification is a modification of standard URL syntax per [RFC-3986](https://www.ietf.org/rfc/rfc3986.txt). As such it benefits from familiarity which is a boon to adoption. One of the features of a DID is that it is a self certifying identifier in that a DID includes either a public key or a fingerprint of a public key from a cryptographic public/private key pair. Thereby a signature created with the private key can be verified using the public key provided by the DID. The inclusion of the public part of a cyptographic key pair in the DID give the DID other desirable properties. These include universal uniqueness and pseuodnynmity. Because a cryptographic key pair is generated from a large random number there is an infinitessimal chance that any two DIDs are the same (collision resistance). Another way to describe a DID is that it is a cryptonym, a cryptographically derived pseudonym.

Associated with a DID is a did document (DDO). The DDO provides meta-data about the DID that can be used to manage the DID as well as discover services affiliated with the DID. Typically the DDO is meant to be provided by some service. The DID/DDO model is not a good match for streaming data especially if a new DID/DDO pair would need to be created for each new DAD item. But a DID/DDO is a good match when used as the root or master identifier from which an identifier for the DAD is derived. This derived identifier is called a *derived DID* or *DDID*. Thus only one DID/DDO paring is required to manage a large number of DADs where each DAD may have a unique DDID. The syntax for a DDID is identical for a DID. The difference is that only one DDO with meta-data is needed for the root DID and all the DAD items carry any additional DAD specific meta-data, thus making them self-contained (autonomic).

### DID Syntax

A DID or DDID has the following required syntax:

did:*method*:*idstring*

The *method* is some short string that namespaces the did and provides for unique behavior in the associated method specification. In this paper we will use the method *dad*.

The  *idstring* is linked to a cryptographic key pair and is defined by the method. In this paper we will use a 44 character Base64 URL-File safe  encoding as per [RFC-4648](https://tools.ietf.org/html/rfc4648) with one trailing pad byte of the 32 byte public verification key for an EdDSA (Ed25519) signing key pair. Unless otherwise specified Base64 in this document refers to the URL-File safe version of Base64. The URL-File safe version of Base64 encoding replaces "+" with “-” and  “\” with  “_”. 

As an example a did using this format would be as follows:

```bash
did:dad:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=
```

A DID may have optional parts including a path, query, or fragment. These used the same syntax of a URL, that is, the path is delimited with slashes, */*, the query with a question mark, *?*, and the fragment with a pound sign, *#*. When the path part is provided then the query applies to the resource referenced by the path and the fragment refers to an element in the document referenced by the path. An example follows:

```bash
did:dad:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=/mom?who=me#blue
```

In contrast, when the path part is missing but either the query or fragment part is provided then the query and/or fragment parts have special meaning. A query without a path means the the query is an operation on the either the DID itself or the DID document (DDO). Likewise when a fragment is provided then the fragment is referencing an elemet of the DDO. An example of a DID without a path but with a query follows:

```bash
did:dad:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=?who=me
```

As will be described later, a query part on a DID expression without a path part will enable the generation of *DDIDs* (derived DIDs)

### Minimal DAD

A minimal DAD (decentralized autonomic data) item is a data item that contains a DID or DDID that helps uniquely identify that data item or affiliated data stream.  In this paper JSON is used to represent serialized DAD items but other formats could be used instead. An example minimal trivial DAD is provided below. Its trivial because there is no data payload.

```json
{
    "id": "did:dad:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148="
}
```

To ensure data integrity, i.e. that the data has not been tampered with. Appended to the DAD item is a signature that is verifiable as being generated by the private key associated with the public key in the *id* field value. This signature verifies that the DAD item was created by the holder of the associated private key  The DAD item is both self-identifing and self-certifying because the identifier value given by the *id* field is included in the signed data and is verifiable against the private key associated with the public key obtained from the associated DID in the *id* field. In the example below is a trivial DAD with an appended signature. The signature is separated from the JSON serialization with characters that may not appear in the JSON.

```json
{
    "id": "did:dad:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148="
}
\r\n\r\n
u72j9aKHgz99f0K8pSkMnyqwvEr_3rpS_z2034L99sTWrMIIJGQPbVuIJ1cupo6cfIf_KCB5ecVRYoFRzAPnAQ==
```

An example DAD with a payload follows:

```json
{
    "id": "did:dad:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=",
    "data":
    {
        "name": "John Smith",
        "nation": "USA"
    }
}
\r\n\r\n
u72j9aKHgz99f0K8pSkMnyqwvEr_3rpS_z2034L99sTWrMIIJGQPbVuIJ1cupo6cfIf_KCB5ecVRYoFRzAPnAQ==
```

While, the simple DADs given in the examples above are minimally self-identifying and self-certifying, they do not provide support for other self-management properties such such as privacy. Moreover because each DID (Decentralized Identifer) or DDID references a public signing key with its associated private key, it needs to be managed as a key not just as an identifier. The following sections will introduce the core key management properties and the associated meta-data that a DAD needs in order to support those properties.

## Key Management

The four main key management operations are:

* Reproduction
* Rotation 
* Revocation 
* Recovery 

We call these the four R's of key management.

### Key Reproduction

Key reproduction is all about managing the creation of new or derived keys. Each new DID requires a new public/private key pair. The private keys must be kept in a secured location. One reason to create unique public private key pairs for each pair-wise relationship is to minimize the risk of exposure to exploit from the repeated use of a given key-pair. Another reason to create unique public/private keys for each interaction between parties is as a means for maintaining privacy through *pseudonymity*. This is discusses in more detail below. Minimizing the number of private keys that must be securely preserved for a given number of public keys simplifies management and reduces both expense and risk of exposure. To reiterate, there are two key storage issues, one is storing public keys and the other is securing storing private keys. An exploit that captures a store of public keys may mean a loss of privacy because the expoiter can now correlate activity associated with those public keys. An exploit that captures a store of private keys means that the exploiter can sign attestations with those private keys and may take control of any associated resources. Consequently, one wants to avoid storing privates as much as possible.

#### Privacy and Confidentiality

One desirable feature of a DAD is that it be privacy preserving. A simplified definition of privacy is that if two parties are participating in an exchange of data in a given context that the parties are not linked to other interactions with other parties in other contexts. A simplified definition of confidentiality is that the content of the data exchanged is not disclosed to a third party. Confidentiality is usually obtained by encrypting the data. This paper does not specifically cover encryption but in general the mechanisms for managing signing keys and encryption keys are highly similar.

An exchange can be private but not confidential, confidential but not private, both, or neither. A minimally sufficent means to preserving privacy is to use a DID as a pseudonomous identifier of each party to the exchange. A *pseudonynm* is a made up alias e.g. identifier that is under the control of its creator and is used to identify a given interaction but is not linkable to other interactions by its owner. The ability of a third party to correlate and entity's behavior across contexts is reduced when the entity uses a unique DID for each context.  Although, there are more sophisticated methods for preserving privacy such as zero knowledge proofs, the goal here is to use methods that are compatible with the performance demands of streaming data. 

As mentioned above, the problem with using unique pseudonyms/cryptonyms for each exchange is that a large number of such identifiers may need to be maintained. Fortunately hierachically derived keychains provide a way to manage these cryptonyms with minimal effort. 

#### Hierachical Deterministic Key Generation

As previously mentioned, reproduction has to do with the generation of new keys. One way to accomplish this is with a deterministic proceedure for generating new public/private keys pairs where the private keys may be reproduced securely without having to be stored. A hierarchically deterministic key generation algorithm does this by using a master or root private key and then generating new key pairs using a deterministic key derivation algorithm. A derived key is expressed as a branch in a tree of parent/child keys. Each public key includes the path to its location in the tree. The private key for a given public key in the tree can be securely regenerated using the root private key and the key path. Only one private key, (the root) needs to be stored. 

The [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) specification, for example, uses an indexed path representation for its HD *chain* code, such as, "0/1/2/0".
The BIP-32 algorithm needs a master or root key pair and a chain code for each derived key. Then only the master key pair needs to be saved and only the master private key needs to be kept securely secret. The other private keys can be reproduced on the fly given the key generation algorithm and the chain code. An extended public key would include the chain code in its representation so that the associated private key can be derived by the holder of the master private key anytime the extended public key is presented. 

The query part of the DID syntax may be used to represent an HD chain code or key path for an HD key that is derived from a root DID. This provides an economoical way to specify derived DIDs (DDIDs) used to identify DADS. An example follows:

```bash
did:dad:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=?chain=0\1\2
```
This expression above discloses the root public DID as well as the key derivation path or chain via the query part. For the sake of brevity this will be call an extended DID. The actual derived DDID is create by applying the HD algorithm such as:

```bash
did:dad:Qt27fThWoNZsa88VrTkep6H-4HA8tr54sHON1vWl6FE=
```
Thus a database of DDIDs could be indexed by DDID expressions with each value being the extended DID. Looking up the extended DID allows the holder to recreate on the fly the associated private key for the DDID without ever having to store the private key. 

Some refinements to this approach may be useful. One is the granularity of DDID allocation. A unique DDID could be used for each unique DAD or a unique DID could be used for each unique destination party that is receiving a data stream. In this case each DAD would need an additional identifier to disambiguate on DAD from another that is sent to the same party. This can be provided with an additional field or using the DID path part. The following shows the sequence number provided in the DID path.

```bash
did:dad:Qt27fThWoNZsa88VrTkep6H-4HA8tr54sHON1vWl6FE=/10057
```
The associated DAD is as follows:

```json
{
    "id": "did:dad:Qt27fThWoNZsa88VrTkep6H-4HA8tr54sHON1vWl6FE=/10057",
    "data":
    {
        "temp": 50,
        "time": "12:15:35"
    }
}
\r\n\r\n
u72j9aKHgz99f0K8pSkMnyqwvEr_3rpS_z2034L99sTWrMIIJGQPbVuIJ1cupo6cfIf_KCB5ecVRYoFRzAPnAQ==
```

#### Change Detection

Using a sequence number or some other identifier could provide change detection. Often stale DAD items must be detectable to prevent replay attacks. A later re-transmission of an old copy of the DAD item not supercede a newer copy. One way to provide change detection is for the DAD item to include a *changed* field whose value is monotonically increasing and changes everytime the data is changed. The souce of the data can enforce that the value is monotonically increasing. Typical  approaches include a monotonically increasing date-time stamp or sequence number. Any older data items will have older date-time stamps or lower sequence numbers and will thus be detectable as stale.

Below is an example of an non-trivial data item that has a *changed* field for change detection.

```json
{
    "id": "did:dad:Qt27fThWoNZsa88VrTkep6H-4HA8tr54sHON1vWl6FE=/10057",
    "changed" : "2000-01-01T00:00:00+00:00",
    "data":
    {
        "temp": 50,
        "time": "12:15:35"
    }
}
\r\n\r\n
u72j9aKHgz99f0K8pSkMnyqwvEr_3rpS_z2034L99sTWrMIIJGQPbVuIJ1cupo6cfIf_KCB5ecVRYoFRzAPnAQ==
```

Change detection prevents replay attacks in the following manner. A second party receives DAD updates that are each signed by the associated private key. Each update has a monitonically increasing changed field. The source signer controls the contents of the data wrapped by the signature. Therefore the signer controls any the changed field. A consistent signer will use a monotonically increasing changed value whenever the data wrapped by the signature is changed. Thus a malicious third party cannot replay earlier instances of the DAD wrapped by a valid signature to the orginal second party because the second party knows to discard any receptions that have older changed fields than the latest one they have already received. 

#### On the Fly DDIDS in DADs

One important use case for DDIDS in DADS is to identify data that is received from a source that is not providing identifying information with the data. The receiver then creates an associated DID and DDIDs to identify the data. At some later point in the point the receiver may be able to link this data with some other identifying information or the source may *claim" this data later by supplying identifying information. In this case the DDIDs are private to the receiver but can later be used to credibly provenance the internal use of the data. This may be extremely beneficial when shared amongst the entities in the processing chain as a way to manage the entailed proliferation of keys that may be all claimed later as a hierarchial group. The DIDs and associated derivation operations for DDIDS may be shared amongst a group of more-or-less trusted entities that are involved in the processing chain.

#### Public Derivation

Another important used case for DDIDS in DADS is to avoid storing even the the DDID with its derivation chain. This may be an issue when a client wishes to communicate with a potenially very large number of public services. Each public service would be a new pairing with a unique DDID. If the derivation algorithm for an HD Key DDID could use the public key or public DID of the public service to generate the DDID then the client need not store the actual DDID but can recover the DDID by using the public DID of the server to re-derive the associated DDID.



### Key Rotation

Key rotation is necessary because keys used for signing (and/or encryption) may become compromised as some point or risk becoming compromized if overused. Changing the key that is used to sign a data item to a new key manages the risk of compromise. 

One way to reduce lookups is to include key-rotation support directly in the data item. Key rotation can be expressed directly as part of a data item by adding a keys field. The value of the  keys field is a list of Public keys or DIDs. Rotating a key is accomplished by adding a key to the key list and then changing the signer field value to reference the new key.

If the data is used in an immutable data system the original data item is not changed but a new one created with changed keys. If the data item is used in a mutable data system then the original data item may be replaced with a new one.  


When the the signer field DID prefix is the same as the item DID but has a fragment that references one of the keys in the keys field list, then the data item is self-signed in that the signer field key reference is contained in the data item itself. To establish that the signer private key and did private key are held by the same entity, either attach two signatures one by each private key or issue a  challenge to the public key of the data item DID.

If the signer field value of a given data item (DID prefix with fragment) references the keys field of a different data item, the the given data item is not self-signed. 

Example of key rotatable self-signed data item.

```json
{
   "did": "did:rep:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=",
   "signer": "did:igo:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=#keys/0",
   "changed": "2000-01-01T00:00:00+00:00",
   "keys": 
   [
    {
      "key": "Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=",
    }
   ],
   "name": "Jim",
   "age": 30
}
/r/n/r/n
B0Qc72RP5IOodsQRQ_s4MKMNe0PIAqwjKsBl4b6lK9co2XPZHLmzQFHWzjA2PvxWso09cEkEHIeet5pjFhLUDg==

```

When the cryptographic suite is not the intelligent default then an optional *kind* field can be included in a given keys list item that provides the cryptographic suite and version.


A modified version would allow each key value to be a DID.

```json
{
   "did": "did:rep:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=",
   "signer": "did:igo:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=#keys/0",
   "changed": "2000-01-01T00:00:00+00:00",
   "keys": 
   [
    {
      "key": "Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=",
    },
    {
      "key": "did:rep:Qt27fThWoNZsa88VrTkep6H-4HA8tr54sHON1vWl6FE=",
      "kind": "ecdsa:1.0",
    },
   ],
   "name": "Jim",
   "age": 30
}
/r/n/r/n
B0Qc72RP5IOodsQRQ_s4MKMNe0PIAqwjKsBl4b6lK9co2XPZHLmzQFHWzjA2PvxWso09cEkEHIeet5pjFhLUDg==

```

A pair-wise interaction is reasonably private if a unique key pair is generated for the interaction and not reused else where.  The idea I presented which he helped refine is that for public services, the hd key could be generated from the public service identifier so the
client would not have to remember anything to recreate the key pair.  For non-public services this would not be true but would require the client remember some information about the interaction. 

The other idea is to pre-rotate each key pair by publishing in the DID document associated with the DID key pair the next key to rotate too. This eliminates an exploit where a key gets compromised and then is used to rotate to a key not in control of the orginal owner. By pre-rotating a comprimized key can at best trigger a rotation to a key that is not compromised at which point the orignal owner uses the rotated key to generate a new pre-rotated key.  The pre-rotated key  is not vulnerable to exploit since it is not used to sign anything.

The client then needs to keep a list of all the rotated keys so that if the client needs to regenerate an hd-key and doesnt remember which master key was used it can try the list of pre-rotated keys. Key recovery would also keep this list. This a couple of orders of magnitude less effort than having to keep all the keys pairs. Only the master keys in sequence.





### Key Recovery



### Key Revocation
Everytime a Key is rotated or compromised the key needs to be revoked which involves basically stopping to use the key and then rotate to the new set of keys. And also vice-versa, whenever a key is rotated the previous key ideally must be revoked.
In circumstances like termination or completeion of interaction the key may just be revoked and not rotated.

Rovocation may involve informing the counterperties and further in case of highly senetive environments maintaining a time stamped list of revoked keys for reference. 

The DDIDs usually would just rotate to the next set of keys and have no overhead of a revocation process. However a Master key should never be revoked or rotated.







Adding support for key management to a self-identifying data item is a good example where design choice trade-offs relative to expressive power can be made in the data item representation. 






### Example
If the signer field value is the same as the did field value then the data item is trivially self-signed. 

Example of a trivial self-identifying data item that is also trivally self-signed. A JSON serialization of the data item is followed by a separator and a signature generated by the did=signer's private key as applied to the JSON serializaton.

```json
{
   "did": "did:rep:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=",
   "signer": "did:igo:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=",
}
/r/n/r/n
B0Qc72RP5IOodsQRQ_s4MKMNe0PIAqwjKsBl4b6lK9co2XPZHLmzQFHWzjA2PvxWso09cEkEHIeet5pjFhLUDg==

```

The data above conveys no additional meaning besides its included identifier. To make this data item convey more meaning, additional fields may be included. Below is an example of an non-trivial data item that is still trivally self-signed

```json
{
   "did": "did:rep:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=",
   "signer": "did:igo:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=",
   "name": "Jim",
   "age": 30
}
/r/n/r/n
B0Qc72RP5IOodsQRQ_s4MKMNe0PIAqwjKsBl4b6lK9co2XPZHLmzQFHWzjA2PvxWso09cEkEHIeet5pjFhLUDg==

```

The data item is tamper-proof in the sense that any change to any of the fields will invalidate the signature. 




### HTTP example

In web applications that use HTTP, the simplest most compatible way to associate or attach the signature in an HTTP packet is to include it in a custom HTTP header. Standad JSON parsers raise an error if there are additional characters after a closing object bracket thus one cannot simply append the signature after the JSON serialization in the message body. Another approach would be to use a custom JSON parser that guarantees a cononical representation of a JSON serialization (including white-space) and then wrap the data item and the signature in another JSON object where the signature and the data item are both field in the wrapper object. This is more verbose and is not compatible with the vast majority of web application framework tools for handling JSON serialized message bodies. Thus it is non-trivial to include the signature in the message body.  Using a custome HTTP header is relatively easy and has the advantage that is is compatible with the vast majority of existing web frameworks.  

A suggested header name is  *Signature* header that provides one or more signatures of the request/response body text.

The format of the custom Signature header follows the conventions of [RFC 7230](https://tools.ietf.org/html/rfc7230)

Signature header has format:

```http
Signature: headervalue

Headervalue:
  tag = "signature"
or
  tag = "signature"; tag = "signature"  ...
  
where tag is replaced with a unique string for each signature value
```

An example is shown below where one *tag* is the string *signer* and the other *tag* is the string *current*.

```http
Signature: signer="Y5xTb0_jTzZYrf5SSEK2f3LSLwIwhOX7GEj6YfRWmGViKAesa08UkNWukUkPGuKuu-EAH5U-sdFPPboBAsjRBw=="; current="Xhh6WWGJGgjU5V-e57gj4HcJ87LLOhQr2Sqg5VToTSg-SI1W3A8lgISxOjAI5pa2qnonyz3tpGvC2cmf1VTpBg=="
```


Where tag is the name of a field in the body of the request whose value
is a DID from which the public key for the signature can be obtained.
If the same tag appears multiple times then only the last occurrence is used.

Each signature value is a doubly quoted string ```""``` that contains the actual signature
in Base64 url safe format. By the signatures should use an intelligent default cryptographic suite such as the  64 byte Ed25519 signatures that have been encoded into BASE64 url-file safe format. The encoded signatures are 88 characters in length and include two trailing pad characters ```=```.

An optional *tag* name = *kind*  may be present to specify the cryptographic suite and version of the signatures.
The *kind* tag field value specifies the type of signature. All signatures within the header
must be of the same kind.

```http
Signature: signer="B0Qc72RP5IOodsQRQ_s4MKMNe0PIAqwjKsBl4b6lK9co2XPZHLmzQFHWzjA2PvxWso09cEkEHIeet5pjFhLUDg=="; did="B0Qc72RP5IOodsQRQ_s4MKMNe0PIAqwjKsBl4b6lK9co2XPZHLmzQFHWzjA2PvxWso09cEkEHIeet5pjFhLUDg=="; kind="ed25519:1.0"

```





where ";" semi-colon indicates that what follows is key path and the number represent which
child at each level.

The following is a data item using a signing key that is a hierachically determined DID.

```json
{
   "did": "did:rep:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=",
   "signer": "did:igo:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=#keys/0",
   "changed": "2000-01-01T00:00:00+00:00",
   "keys": 
   [
    {
      "key": "did:rep:Xq5YqaL6L48pf0fu7IUhL0JRaU2_RxFP0AL43wYn148=;0\1\2",
    }
   ],
   "name": "Jim",
   "age": 30
}
/r/n/r/n
B0Qc72RP5IOodsQRQ_s4MKMNe0PIAqwjKsBl4b6lK9co2XPZHLmzQFHWzjA2PvxWso09cEkEHIeet5pjFhLUDg==

```

## Relative Expressive Power

One way to measure and compare different knowledge representations is called *relative expressive power*.  In the physics world *power* is defined as work done per unit time. Its a ratio. *Expressive power* is similary defined as the ratio of meaning conveyed per dependency, where dependency is something that must be kept track of or transmitted to convey the meaningful information. Because dependencies are a measure of complexity, relatively higher expressive power conveys more meaning relatively more simply.

### Intelligent Defaults

One approach to acheiving higher expressive power in a data representation specification is the use of intelligent defaults. An intelligent default assigns meaning to the absence of data. For example, if there are several options for a given data item value such as the *type* of a data item, an intelligent default would assign the type to a predetermined default if no type is provided in the data. This provides high expressive power as the type meaning is conveyed without the transmission of any bytes to represent type.

Typically in any given knowledge representation application the relative frequency of the appearance of optional values is not evenly distributed, but follows a Pareto distribution. This means that if an intelligent default (the Pareto optimal value) is specified as part of the schema the average expressive power of data items will be increased.

A practical example of this is the RAET (Reliable Asynchronous Event Transport) protocol header (see https://github.com/RaetProtocol/raet). Typically in protocols the header has a fixed format binary representation for two reasons. The first is that every packet includes the header so a verbose header reduces the payload capacity of each packet thereby making the protocol comsume more bandwidth. The second is that the header is used to interpret the rest of the packet and therefore must be consistenly parsable which is easier if the format is fixed. The problem with fixed format headers is that they are not extensible. To make the extensible usually means adding addition fields to the header to indicate the precense of additional extended fields.  RAET used an *intelligent default policy* to achieve a completely flexible extensible header that on average is the size of a non-extensible fixed format header. In RAET the header is composed of a serialized list of key-value pairs where each key is the field name of the associated field value. This makes it easy to add new key-value pairs as needed to extend the protocol to different uses and with different behavior. Unfortunately, transmitting the keys makes the header much larger relative to a fixed format header where the offset of the value in the header determines the associated field. RAET overcomes this problem by defining a default value for each key-value pair. When a header is generated on the transmit side, the actual key-value pairs are compared against the default set. Any pair where the value matches the default is not included in the list of key-value pairs in the transmitted header. On the recieve side a default header is created with every key value pair set to the default. The received header's key-value pairs are used to update the default header with the non-defaulted values.  Because the optional fields are seldomly used by most packets the average header size is comparable to a fixed format header.  When viewing the header after expansion and update all the fields are present so there is no hidden information. All the meaning is apparently conveyed.

RAET header field defaults

```python
PACKET_DEFAULTS = odict([
                            ('sh', DEFAULT_SRC_HOST),
                            ('sp', RAET_PORT),
                            ('dh', DEFAULT_DST_HOST),
                            ('dp', RAET_PORT),
                            ('ri', 'RAET'),
                            ('vn', 0),
                            ('pk', 0),
                            ('pl', 0),
                            ('hk', 0),
                            ('hl', 0),
                            ('se', 0),
                            ('de', 0),
                            ('cf', False),
                            ('bf', False),
                            ('nf', False),
                            ('df', False),
                            ('vf', False),
                            ('si', 0),
                            ('ti', 0),
                            ('tk', 0),
                            ('dt', 0),
                            ('oi', 0),
                            ('wf', False),
                            ('sn', 0),
                            ('sc', 1),
                            ('ml', 0),
                            ('sf', False),
                            ('af', False),
                            ('bk', 0),
                            ('ck', 0),
                            ('fk', 0),
                            ('fl', 0),
                            ('fg', '00'),
                      ])
```

Any key value based schema standard specification may benefit from an intelligent default policy to greatly increase the expressive power of the schema.  This becomes even more important where security is concerned as the intelligent default might be the most secure set of options thus helping the user be more secure and more expressive. Moreover expressive power is about conveying meaning more simply which makes it easier to implement and incentivizes adoption.

### Essential vs. Optional Elements

Another related technique for increasing expressive power is to distinguish between essential and optional elements in a given representation. Any essential elements should be expressed as explicitly as possible (when not defaulted), that is, should not be looked up and should either not be indirected or have minimal indirecton. External lookups are expensive. Moreover, hiding essential elements behind multiple levels of indirection may make it harder to understand the conveyed meaning (adding dependencies and hence complexity). An important meaningful difference has occurred whenever an essential element is not set to a default value. This difference should not be hidden behind indirection.


#### Cryptographic Suite Representation

Best practices cryptography limits the option that user may choose from for the various cryptographic operations, such as, signing, encrypting, hashing to a suite of balanced and tuned set of protocols, one for each operation. Each member of the set should be the one and only one best suited to that operation. This prevents the user from making bad choices. In most key representation schemes each operation is completely free to be specified independent of the others. This is a very bad idea.  Users should not be custom combining different protocols that are not part of a best practices cypher suite. Each custom configuration may be vunerable to potential attack vectors for exploit. The suggested approach is to specify a cypher suite with a version. If an exploit is discovered for a member of a suite and then fixed, the suite is updated totally to a new version. The number of cypher suites should be minimized to those essential for compatibility but no more. This approach increases expressive power because only one element is needed to specify a whole suite of operations instead of a different element per operation.

The following article explains in more detail how standards such as JOSE expose vulnerabilities due to too much flexibility in how cryptographic operations are specified. 

https://paragonie.com/blog/2017/03/jwt-json-web-tokens-is-bad-standard-that-everyone-should-avoid 

Example cypher suites:

```bash
v1: Ed25519, X25519, XSalsa20poly1305, HMAC-SHA-512-256
v2: Ed448, X448, XChaCha20Poly1305, keyed BLAKE2b
v3: SPHINCS-256, SIDH, NORX64-4-1, keyed BLAKE2x
```

## Nested Signatures


## Data Canonicalization

Data canonization means that there is a universally defined way of serializing the data that is to be cyptographically signed.

The are few typical approaches to achieving data canonization.

1. Store the serialization and signature as a chunk.

The simplest is that the signer is the only entity that actually serializes the data. All other users of the data only deserialize. This simplifies the work to guarantee canonization. For example JSON is the typical data format used to serialize key:value or structured data. But the JSON specifcation for ser/deser treats whitespace characters as semantically unimportant as well as the order of appearance of keys. For a dictionary (key:value) data structure the typical approach is to represent it internally as a hash table. Most hash algorithms do not store data ordered in any predictable way (Python and other languages have support for Ordered Dicts or Ordered hashes which can be used to partially ameliorate this problem). But from the perspective of equivalence, key:value data structures are "dict" equal if they have the same set of keys with the same values for each key. Thus deserialization can produce uniform equivalent "dict equal" results from multiple but differing serializations (that differ in whitespace and order of appearance of fields). JSON only guarantees *dict* equivalent not serialization equivalence. Unfortunately the signatures for the differing but equivalent serializations will not match.

But in signed at rest data only the signer ever needs to serialize the data. Indeed, only the signer may serialize the data because only the signer has the private key. So deserialization and reserialization by others is of limited value. The primary value appears to be either schema completeness where signatures are included as fields in a wrapper object or the ability to nest signatures or signed data with signatures. Because is is simple to convert a JSON serialization to a coded serializaiton such as Base64. Nested coded JSON serialization without canonicalization can be trivially supported.  After expansion and decoding, readers of the data can see the uncoded underlying data in a *schema complete* representation.

The signer's serialization is always *canonical* for the signature. Users of the data merely need to use a "dict equal" deserialization which is provided by any compliant JSON deserializer. So no additional work is required to support it across multiple languages etc. If the associated data also needs to be stored unserialized then validation and extraction of the data is performed by first verifying the signature on the stored serialization and then deserializing it in memory.

2. Implement perfectly canonical universally reproducibly serialization.

In this approach all implementations of the protocol or service use the exact same serialization method that is canonical including white space and field order so that they can reproduce the exact same serialization that the original signer created when originally signing the data. This is difficult to achieve with something like JSON across multiple languages, platforms, and tool kits. Its usually more work to implement and more work to support because it usually means either using something other than JSON for serialization or writing from scratch conformant JSON implementations or at the very least having tight control of how white space and order occurs and ensuring accross updates that this does not change. Unfortunately many overly schematizied standards are based on this approach. This approach breaks virtually every web framework.

3. Use binary data structures

With binary data structures the canonical form is well defined but it is also highly inflexible. The advantages of compatibility, flexibility and modularity that come from using a key/value store serialization such as JSON usually makes 1) or 2) the preferred approach.

