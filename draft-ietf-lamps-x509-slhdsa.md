---
title: "Internet X.509 Public Key Infrastructure: Algorithm Identifiers for SLH-DSA"
abbrev: "SLH-DSA for X.509"
category: std
stand_alone: true # This lets us do fancy auto-generation of references
ipr: trust200902

docname: draft-ietf-lamps-x509-slhdsa-latest
submissiontype: IETF
v: 3
area: sec
workgroup: LAMPS - Limited Additional Mechanisms for PKIX and SMIME
keyword:
 - SLH-DSA
 - SPHINCS+
 - PQ Signatures
 - post-quantum X.509
venue:
  group: LAMPS
  type: Working Group
  mail: spasm@ietf.org
  arch: "https://mailarchive.ietf.org/arch/browse/spasm/"
  github: "x509-hbs/draft-x509-slhdsa"

author:
-
    ins: K. Bashiri
    name: Kaveh Bashiri
    org: BSI
    email: kaveh.bashiri.ietf@gmail.com
-
    ins: S. Fluhrer
    name: Scott Fluhrer
    org: Cisco Systems
    email: sfluhrer@cisco.com
-
    ins: S. Gazdag
    name: Stefan-Lukas Gazdag
    org: genua GmbH
    email: ietf@gazdag.de
-
    ins: D. Van Geest
    name: Daniel Van Geest
    org: CryptoNext Security
    email: daniel.vangeest@cryptonext-security.com
-
    ins: S. Kousidis
    name: Stavros Kousidis
    org: BSI
    email: kousidis.ietf@gmail.com

normative:
  FIPS205: DOI.10.6028/NIST.FIPS.205

informative:
  NIST-PQC:
    target: https://csrc.nist.gov/projects/post-quantum-cryptography
    title: Post-Quantum Cryptography Project
    author:
      - org: National Institute of Standards and Technology
    date: 2016-12-20
  CMP2018:
    author:
    - ins: L. Castelnovi
      name: Laurent Castelnovi
    - ins: A, Martinelli
      name: Ange Martinelli
    - ins: T. Prest
      name: Thomas Prest
    date: '2018'
    seriesinfo:
      Lecture Notes in Computer Science: vol 10786
      PQCrypto: '2018'
      Post-Quantum Cryptography: pp. 165-184
    target: https://link.springer.com/chapter/10.1007/978-3-319-79063-3_8
    title: 'Grafting Trees: A Fault Attack Against the SPHINCS Framework'
  SLotH:
    author:
    - name: M-J. Saarinen
    date: '2024'
    target: https://eprint.iacr.org/2024/367.pdf
    title: 'Accelerating SLH-DSA by Two Orders of Magnitude with a Single Hash Unit'
  X680:
    target: https://www.itu.int/rec/T-REC-X.680
    title: "Information Technology - Abstract Syntax Notation One (ASN.1): Specification of basic notation. ITU-T Recommendation X.680 (2021) | ISO/IEC 8824-1:2021."
    author:
      org: ITU-T
    date: February 2021
  Ge2023:
    author:
    - name: Aymeric Genêt
    target: https://doi.org/10.46586/tches.v2023.i2.80-114
    title: On Protecting SPHINCS+ Against Fault Attacks
    seriesinfo:
      TCHES: 2023/02

--- abstract

Digital signatures are used within X.509 Public Key Infrastructure such as X.509 certificates, Certificate Revocation Lists (CRLs), and to sign messages.  This document describes the conventions for using the Stateless Hash-Based Digital Signature Algorithm (SLH-DSA) in X.509 Public Key Infrastructure.  The conventions for the associated signatures, subject public keys, and private keys are also described.

<!-- End of Abstract -->

--- middle

# Introduction

The Stateless Hash-Based Digital Signature Algorithm (SLH-DSA) is a quantum-resistant digital signature scheme standardized in {{FIPS205}} by the US National Institute of Standards and Technology (NIST) PQC project {{NIST-PQC}}. Prior to standardization, the algorithm was known as SPHINCS+. SLH-DSA and SPHINCS+ are not compatible. This document defines the ASN.1 Object Identifiers (OIDs) and conventions for the encoding of SLH-DSA digital signatures, public keys and private keys in the X.509 Public Key Infrastructure.

SLH-DSA offers three security levels.  The parameters for each of the security levels were chosen to be at least as secure as a generic block cipher of 128, 192, or 256 bits. There are small (s) and fast (f) versions of the algorithm, and the option to use SHA-256 {{?FIPS180=NIST.FIPS.180-4}} or SHAKE256 {{?FIPS202=NIST.FIPS.202}} as internal hash functions. The fast versions are optimized for key generation and signing speed, they are actually slower at verification than the SLH-DSA small parameter sets. For example, id-slh-dsa-shake-256s represents the 256-bit security level, the small version of the algorithm, and the use of SHAKE256.

Separate algorithm identifiers have been assigned for SLH-DSA at each of these security levels, fast vs small, and SHA-256 vs SHAKE256.

SLH-DSA offers two signature modes: pure mode and predigest mode. SLH-DSA signature operations include a context string as input.  The context string has a maximum length of 255 bytes.  By default, the context string is the empty string.
This document only specifies the use of pure mode with an empty context string for use in the X.509 Public Key Infrastructure.

<!-- End of introduction section -->

# Conventions and Definitions

{::boilerplate bcp14-tagged}

<!-- EDNOTE: [DVG] I added this to my own (unpublished) draft based on draft-ietf-lamps-cms-sphincs-plus. draft-ietf-lamps-dilithium-certificates, on which we are basing this draft doesn't have such an extensive overview section. If draft-ietf-lamps-cms-sphincs-plus gets reduced in scope and refers more to this draft, we could add this text.-->
<!--
# SLH-DSA Hash-based Signature Algorithm Overview

SLH-DSA is a hash-based signature scheme which consists of a few time signature construction, namely Forest of Random Subsets (FORS) and a hypertree.  FORS signs a message with a private key.  The corresponding FORS public keys are the leaves in k binary trees.  The roots of these trees are hashed together to form a FORS root.  SLH-DSA uses a one-time signature scheme called WOTS+. The FORS tree roots are signed by a WOTS+ one-time signature private key.  The corresponding WOTS+ public keys form the leaves in d-layers of Merkle subtrees in the SLH-DSA hypertree.  The bottom layer of that hypertree signs the FORS roots with WOTS+. The root of the bottom Merkle subtrees are then signed with WOTS+ and the corresponding WOTS+ public keys form the leaves of the next level up subtree.  Subtree roots are consequently signed by their corresponding subtree layers until we reach the top subtree.  The top layer subtree forms the hypertree root which is trusted at the verifier.

A SLH-DSA signature consists of the FORS signature, the WOTS+ signature in each layer, and the path to the root of each subtree until the root of the hypertree is reached.

A SLH-DSA signature is verified by verifying the FORS signature, the WOTS+ signatures and the path to the root of each subtree.  When reaching the root of the hypertree, the signature verifies only if it hashes to the pre-trusted root of the SLH-DSA hypertree.

SLH-DSA is a stateless hash-based signature algorithm.  Stateful hash-based signature schemes require that the WOTS+ private key (generated by using a state index) is never reused or the scheme loses it security.  Although its security decreases, FORS which is used at the bottom of the SLH-DSA hypertree does not collapse if the same private key used to sign two or more different messages like in stateful hash-based signature schemes.  Without the need for state kept by the signer to ensure it is not reused, SLH-DSA is much less fragile.

SLH-DSA was designed to sign up to 2^64 messages and offers three security levels.  The parameters of the SLH-DSA hypertree include the security parameter, the hash function, the tree height, the number of layers of subtrees, the Winternitz parameter of WOTS+, the number of FORS trees and leaves in each.  The parameters for each of the security levels were chosen to provide 128 bits of security, 192 bits of security, and 256 bits of security.
-->

# Algorithm Identifiers {#sec-alg-ids}

The AlgorithmIdentifier type, which is included herein for convenience, is defined as follows:

~~~ asn.1
AlgorithmIdentifier{ALGORITHM-TYPE, ALGORITHM-TYPE:AlgorithmSet} ::=
        SEQUENCE {
            algorithm   ALGORITHM-TYPE.&id({AlgorithmSet}),
            parameters  ALGORITHM-TYPE.
                   &Params({AlgorithmSet}{@algorithm}) OPTIONAL
        }
~~~

<aside markdown="block">
The above syntax is from {{?RFC5912}} and is compatible with the 2021 ASN.1 syntax {{X680}}.
See {{?RFC5280}} for the 1988 ASN.1 syntax.
</aside>

The fields in AlgorithmIdentifier have the following meanings:

* algorithm identifies the cryptographic algorithm with an object identifier.

* parameters, which are optional, are the associated parameters for the algorithm identifier in the algorithm field.

The SLH-DSA OIDs are:

~~~ asn.1
   nistAlgorithms OBJECT IDENTIFIER ::= { joint-iso-itu-t(2)
     country(16) us(840) organization(1) gov(101) csor(3) 4 }

   sigAlgs OBJECT IDENTIFIER ::= { nistAlgorithms 3 }

   id-slh-dsa-sha2-128s OBJECT IDENTIFIER ::= { sigAlgs 20 }

   id-slh-dsa-sha2-128f OBJECT IDENTIFIER ::= { sigAlgs 21 }

   id-slh-dsa-sha2-192s OBJECT IDENTIFIER ::= { sigAlgs 22 }

   id-slh-dsa-sha2-192f OBJECT IDENTIFIER ::= { sigAlgs 23 }

   id-slh-dsa-sha2-256s OBJECT IDENTIFIER ::= { sigAlgs 24 }

   id-slh-dsa-sha2-256f OBJECT IDENTIFIER ::= { sigAlgs 25 }

   id-slh-dsa-shake-128s OBJECT IDENTIFIER ::= { sigAlgs 26 }

   id-slh-dsa-shake-128f OBJECT IDENTIFIER ::= { sigAlgs 27 }

   id-slh-dsa-shake-192s OBJECT IDENTIFIER ::= { sigAlgs 28 }

   id-slh-dsa-shake-192f OBJECT IDENTIFIER ::= { sigAlgs 29 }

   id-slh-dsa-shake-256s OBJECT IDENTIFIER ::= { sigAlgs 30 }

   id-slh-dsa-shake-256f OBJECT IDENTIFIER ::= { sigAlgs 31 }
~~~

The contents of the parameters component for each algorithm are absent.

# SLH-DSA Signatures

SLH-DSA is a digital signature scheme built upon hash functions. The security of SLH-DSA relies on the presumed difficulty of finding preimages for hash functions as well as several related properties of the same hash functions.

Signatures can be placed in a number of different ASN.1 structures.
The top level structure for a certificate is given below as being
illustrative of how signatures are frequently encoded with an
algorithm identifier and a location for the signature.

~~~ asn.1
  Certificate  ::=  SIGNED{ TBSCertificate }

  SIGNED{ToBeSigned} ::= SEQUENCE {
     toBeSigned           ToBeSigned,
     algorithmIdentifier  SEQUENCE {
         algorithm        SIGNATURE-ALGORITHM.
                            &id({SignatureAlgorithms}),
         parameters       SIGNATURE-ALGORITHM.
                            &Params({SignatureAlgorithms}
                              {@algorithmIdentifier.algorithm})
                                OPTIONAL
     },
     signature BIT STRING (CONTAINING SIGNATURE-ALGORITHM.&Value(
                              {SignatureAlgorithms}
                              {@algorithmIdentifier.algorithm}))
  }
~~~

<aside markdown="block">
The above syntax is from {{?RFC5912}} and is compatible with the 2021 ASN.1 syntax {{X680}}.
See {{?RFC5280}} for the 1988 ASN.1 syntax.
</aside>

The same algorithm identifiers are used for signatures as are used
for public keys.  When used to identify signature algorithms, the
parameters MUST be absent.

The data to be signed is prepared for SLH-DSA.  Then, a private key
operation is performed to generate the raw signature value.

Section 9.2 of {{FIPS205}} defines an SLH-DSA signature as three elements,
R, SIG_FORS and SIG_HT. The raw octet string encoding of an SLH-DSA
public key is the concatenation of these three elements, i.e. R || SIG_FORS || SIG_HT.
The raw octet string representing the signature is encoded
directly in the BIT STRING without adding any additional ASN.1
wrapping.  For example, in the Certificate structure, the raw signature
value is encoded in the "signatureValue" BIT STRING field.

# Subject Public Key Fields {#sec-pub-keys}

In the X.509 certificate, the subjectPublicKeyInfo field has the SubjectPublicKeyInfo type, which has the following ASN.1 syntax:

~~~ asn.1
  SubjectPublicKeyInfo {PUBLIC-KEY: IOSet} ::= SEQUENCE {
      algorithm        AlgorithmIdentifier {PUBLIC-KEY, {IOSet}},
      subjectPublicKey BIT STRING }
~~~

<aside markdown="block">
The above syntax is from {{?RFC5912}} and is compatible with the 2021 ASN.1 syntax {{X680}}.
See {{?RFC5280}} for the 1988 ASN.1 syntax.
</aside>

The fields in SubjectPublicKeyInfo have the following meanings:

* algorithm is the algorithm identifier and parameters for the public key (see above).

* subjectPublicKey contains the byte stream of the public key.

{{!I-D.draft-ietf-lamps-cms-sphincs-plus}} defines the following public key identifiers for SLH-DSA:

~~~ asn.1
   pk-slh-dsa-sha2-128s PUBLIC-KEY ::= {
      IDENTIFIER id-slh-dsa-sha2-128s
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-sha2-128f PUBLIC-KEY ::= {
      IDENTIFIER id-slh-dsa-sha2-128f
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-sha2-192s PUBLIC-KEY ::= {
      IDENTIFIER id-slh-dsa-sha2-192s
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-sha2-192f PUBLIC-KEY ::= {
      IDENTIFIER id-slh-dsa-sha2-192f
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-sha2-256s PUBLIC-KEY ::= {
      IDENTIFIER id-slh-dsa-sha2-256s
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-sha2-256f PUBLIC-KEY ::= {
      IDENTIFIER id-slh-dsa-sha2-256f
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-shake-128s PUBLIC-KEY ::= {
      IDENTIFIER id-slh-dsa-shake-128s
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-shake-128f PUBLIC-KEY ::= {
      IDENTIFIER id-slh-dsa-shake-128f
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-shake-192s PUBLIC-KEY ::= {
      IDENTIFIER id-slh-dsa-shake-192s
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-shake-192f PUBLIC-KEY ::= {
      IDENTIFIER id-slh-dsa-shake-192f
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-shake-256s PUBLIC-KEY ::= {
      IDENTIFIER id-slh-dsa-shake-256s
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-shake-256f PUBLIC-KEY ::= {
      IDENTIFIER id-slh-dsa-shake-256f
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   SLH-DSA-PublicKey ::= OCTET STRING
~~~

Section 9.1 of {{FIPS205}} defines an SLH-DSA public key as two n-byte elements,
PK.seed and PK.root. The raw octet string encoding of an SLH-DSA
public key is the concatenation of these two elements, i.e. PK.seed || PK.root. The octet
string length is 2*n bytes, where n is 16, 24, or 32, depending on the SLH-DSA parameter
set. When used in a SubjectPublicKeyInfo type, the subjectPublicKey BIT STRING
contains the raw octet string encoding of the public key.

{{!I-D.draft-ietf-lamps-cms-sphincs-plus}} defines the SLH-DSA-PublicKey ASN.1
OCTET STRING type to provide an option for encoding a public key in an
environment that uses ASN.1 encoding but doesn't define its own mapping of an
SLH-DSA raw octet string to ASN.1. To map an SLH-DSA-PublicKey OCTET STRING to
a SubjectPublicKeyInfo, the OCTET STRING is mapped to the subjectPublicKey
field (a value of type BIT STRING) as follows: the most significant
bit of the OCTET STRING value becomes the most significant bit of the BIT
STRING value, and so on; the least significant bit of the OCTET STRING
becomes the least significant bit of the BIT STRING.

The id-slh-dsa-* identifiers in {{sec-alg-ids}} MUST be used as the algorithm field in the SubjectPublicKeyInfo sequence {{!RFC5280}} to identify a SLH-DSA public key.

{{example-public}} contains an example of an id-slh-dsa-sha2-128s public
key encoded using the textual encoding defined in {{?RFC7468}}.

# Key Usage Bits

The intended application for the key is indicated in the keyUsage certificate extension; see {{Section 4.2.1.3 of RFC5280}}.  If the keyUsage extension is present in a certificate that indicates an id-slh-dsa-* identifier in the SubjectPublicKeyInfo, then the at least one of following MUST be present:

~~~
    digitalSignature; or
    nonRepudiation; or
    keyCertSign; or
    cRLSign.
~~~

If the keyUsage extension is present in a certificate that indicates an id-slh-dsa-* identifier in the SubjectPublicKeyInfo, then the following MUST NOT be present:

~~~
    keyEncipherment; or
    dataEncipherment; or
    keyAgreement; or
    encipherOnly; or
    decipherOnly.
~~~

Requirements about the keyUsage extension bits defined in {{!RFC5280}} still apply.

# Private Key Format

"Asymmetric Key Packages" {{!RFC5958}} describes how to encode a private
key in a structure that both identifies what algorithm the private
key is for and optionally allows for the public key and additional attributes
about the key to be included as well.  For illustration, the ASN.1
structure OneAsymmetricKey is replicated below.

~~~ asn.1
   OneAsymmetricKey ::= SEQUENCE {
      version Version,
      privateKeyAlgorithm PrivateKeyAlgorithmIdentifier,
      privateKey PrivateKey,
      attributes [0] IMPLICIT Attributes OPTIONAL,
      ...,
      [[2: publicKey [1] IMPLICIT PublicKey OPTIONAL ]],
      ...
   }

   PrivateKey ::= OCTET STRING

   PublicKey ::= BIT STRING
~~~

Section 9.1 of {{FIPS205}} defines an SLH-DSA private key as four n-byte
elements, SK.seed, SK.prf, PK.seed and PK.root.  The raw octet string
encoding of an SLH-DSA private key is the concatenation of these four
elements, i.e. SK.seed || SK.prf || PK.seed || PK.root.  The octet string
length is 4*n bytes, where n is 16, 24, or 32, depending on the SLH-DSA parameter
set.  When used in a OneAsymmetricKey type, the privateKey
OCTET STRING contains the raw octet string encoding of the private key.

When an SLH-DSA public key is included in a OneAsymmetricKey type, it is
encoded in the same manner as in a SubjectPublicKeyInfo type. That is, the
publicKey BIT STRING contains the raw octet string encoding of the public
key.

{{example-private}} contains an example of an id-slh-dsa-sha2-128s private
key encoded using the textual encoding defined in {{?RFC7468}}.

NOTE: There exist some private key import functions that have not
picked up the new ASN.1 structure OneAsymmetricKey that is defined in
{{RFC5958}}.  This means that they will not accept a private key
structure that contains the public key field.  This means a balancing
act needs to be done between being able to do a consistency check on
the key pair and widest ability to import the key.

# Security Considerations

The security considerations of {{RFC5280}} apply accordingly.

Implementations MUST protect the private keys.  Compromise of the
private keys may result in the ability to forge signatures.

When generating an SLH-DSA key pair, an implementation MUST generate
each key pair independently of all other key pairs in the SLH-DSA
hypertree.

A SLH-DSA tree MUST NOT be used for more than 2^64 signing
operations.

The generation of private keys relies on random numbers.  The use of
inadequate pseudo-random number generators (PRNGs) to generate these
values can result in little or no security.  An attacker may find it
much easier to reproduce the PRNG environment that produced the keys,
searching the resulting small set of possibilities, rather than brute
force searching the whole key space.  The generation of quality
random numbers is difficult, and {{?RFC4086}} offers important guidance
in this area.

Implementers SHOULD consider their particular use cases and may
choose to implement OPTIONAL fault attack countermeasures [CMP2018],[Ge2023].
Verifying a signature before releasing the signature value
is a typical fault attack countermeasure; however, this
countermeasure is not effective for SLH-DSA [Ge2023].  Redundancy by
replicating the signature generation process can be used as an
effective fault attack countermeasure for SLH-DSA [Ge2023]; however,
the SLH-DSA signature generation is already considered slow.

Likewise, Implementers SHOULD consider their particular use cases and
may choose to implement protections against passive power and
emissions side-channel attacks [SLotH].

# IANA Considerations

For the ASN.1 Module in the Appendix of this document, IANA is
requested to assign an object identifier (OID) for the module
identifier (TBD1) with a Description of "id-mod-x509-slh-dsa-2024". The
OID for the module should be allocated in the "SMI Security for PKIX
Module Identifier" registry (1.3.6.1.5.5.7.0).

--- back

# ASN.1 Module {#sec-asn1}

RFC EDITOR: Please replace TBD2 with the value assigned by IANA during the publication of [I-D.draft-ietf-lamps-cms-sphincs-plus].

~~~ asn.1
<CODE BEGINS>
{::include X509-SLHDSA-2024.asn}
<CODE ENDS>
~~~

# Security Strengths

Instead of defining the strength of a quantum algorithm in a traditional manner using precise estimates of the number of bits of security, NIST has instead elected to define a collection of broad security strength categories.  Each category is defined by a comparatively easy-to-analyze reference primitive that cover a range of security strengths offered by existing NIST standards in symmetric cryptography, which NIST expects to offer significant resistance to quantum cryptanalysis.  These categories describe any attack that breaks the relevant security definition that must require computational resources comparable to or greater than those required for: Level 1 - key search on a block cipher with a 128-bit key (e.g., AES128), Level 2 - collision search on a 256-bit hash function (e.g., SHA256/ SHA3-256), Level 3 - key search on a block cipher with a 192-bit key (e.g., AES192), Level 4 - collision search on a 384-bit hash function (e.g.  SHA384/SHA3-384), Level 5 - key search on a block cipher with a 256-bit key (e.g., AES 256).

The SLH-DSA parameter sets defined for NIST security levels 1, 3 and 5 are listed in {{tab-strengths}}, along with the resulting signature size, public key, and private key sizes in bytes.

| OID                   | NIST Level | Sig.  | Pub. Key | Priv. Key |
|---                    |---         |---    |---       |---        |
| id-slh-dsa-sha2-128s  | 1          | 7856  | 32       | 64        |
| id-slh-dsa-sha2-128f  | 1          | 17088 | 32       | 64        |
| id-slh-dsa-sha2-192s  | 3          | 16224 | 48       | 96        |
| id-slh-dsa-sha2-192f  | 3          | 35664 | 48       | 96        |
| id-slh-dsa-sha2-256s  | 5          | 29792 | 64       | 128       |
| id-slh-dsa-sha2-256f  | 5          | 49856 | 64       | 128       |
| id-slh-dsa-shake-128s | 1          | 7856  | 32       | 64        |
| id-slh-dsa-shake-128f | 1          | 17088 | 32       | 64        |
| id-slh-dsa-shake-192s | 3          | 16224 | 48       | 96        |
| id-slh-dsa-shake-192f | 3          | 35664 | 48       | 96        |
| id-slh-dsa-shake-256s | 5          | 29792 | 64       | 128       |
| id-slh-dsa-shake-256f | 5          | 49856 | 64       | 128       |
{: #tab-strengths title="SLH-DSA security strengths"}

# Examples

This appendix contains examples of SLH-DSA public keys, private keys and certificates.

## Example Public Key {#example-public}

An example of a SLH-DSA public key using id-slh-dsa-sha2-128s:

~~~
-----BEGIN PUBLIC KEY-----
MDAwCwYJYIZIAWUDBAMUAyEAK4EJ7Hd8qk4fAkzPz5SX2ZGAUJKA9CVq8rB6+AKJ
tJQ=
-----END PUBLIC KEY-----
~~~

~~~
  0  48: SEQUENCE {
  2  11:   SEQUENCE {
  4   9:     OBJECT IDENTIFIER '2 16 840 1 101 3 4 3 20'
       :     }
 15  33:   BIT STRING
       :     2B 81 09 EC 77 7C AA 4E 1F 02 4C CF CF 94 97 D9
       :     91 80 50 92 80 F4 25 6A F2 B0 7A F8 02 89 B4 94
       :   }
~~~

## Example Private Key {#example-private}

An example of a SLH-DSA private key without the public key using id-slh-dsa-sha2-128s:

~~~
-----BEGIN PRIVATE KEY-----
MFICAQAwCwYJYIZIAWUDBAMUBECiJjvKRYYINlIxYASVI9YhZ3+tkNUetgZ6Mn4N
HmSlASuBCex3fKpOHwJMz8+Ul9mRgFCSgPQlavKwevgCibSU
-----END PRIVATE KEY-----
~~~

~~~
  0  82: SEQUENCE {
  2   1:   INTEGER 0
  5  11:   SEQUENCE {
  7   9:     OBJECT IDENTIFIER '2 16 840 1 101 3 4 3 20'
       :     }
 18  64:   OCTET STRING
       :     A2 26 3B CA 45 86 08 36 52 31 60 04 95 23 D6 21
       :     67 7F AD 90 D5 1E B6 06 7A 32 7E 0D 1E 64 A5 01
       :     2B 81 09 EC 77 7C AA 4E 1F 02 4C CF CF 94 97 D9
       :     91 80 50 92 80 F4 25 6A F2 B0 7A F8 02 89 B4 94
       :   }
~~~

## Example Certificate

An example or a self-signed SLH-DSA certificate using id-slh-dsa-sha2-128s:

~~~
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            70:fb:96:41:c3:74:53:9b:27:cb:07:bc:d2:bb:4a:bc:
            8a:5d:7a:25
        Signature Algorithm: slhdsa_sha2_128s
        Issuer: C=FR, L=Paris, O=Bogus SLH-DSA-SHA2-128s CA
        Validity
            Not Before: Oct  7 12:51:29 2024 GMT
            Not After : Oct  5 12:51:29 2034 GMT
        Subject: C=FR, L=Paris, O=Bogus SLH-DSA-SHA2-128s CA
        Subject Public Key Info:
            Public Key Algorithm: slhdsa_sha2_128s
                slhdsa_sha2_128s public key:
                PQ key material:
                    2b:81:09:ec:77:7c:aa:4e:1f:02:4c:cf:cf:94:97:
                    d9:91:80:50:92:80:f4:25:6a:f2:b0:7a:f8:02:89:
                    b4:94
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                CD:59:36:AA:FE:C4:11:C7:A4:72:69:3F:0B:E8:B3:8B:
                21:7B:19:ED
            X509v3 Authority Key Identifier:
                CD:59:36:AA:FE:C4:11:C7:A4:72:69:3F:0B:E8:B3:8B:
                21:7B:19:ED
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Key Usage:
                Certificate Sign, CRL Sign
    Signature Algorithm: slhdsa_sha2_128s
    Signature Value:
        11:b2:e9:ee:da:b9:9b:da:da:db:eb:28:2f:9a:2d:31:e2:78:
        40:14:20:f6:67:ec:ca:3e:fa:db:c3:b9:ab:9e:fd:d1:b8:77:
        29:47:1b:40:c0:73:68:7e:89:de:ac:d8:9f:43:71:58:3f:3d:
        da:28:58:57:6d:e0:99:28:ae:30:e9:17:a7:20:c8:11:e1:fa:
        f6:d0:21:ef:01:60:79:93:ac:b4:94:63:97:af:71:14:4c:c5:
        cf:77:40:df:60:f5:8f:20:95:a4:c1:35:02:43:14:b8:1c:4a:
        01:1e:bd:17:01:b7:36:12:80:ab:29:58:98:bf:da:1d:52:8a:
        be:de:67:af:26:6f:61:46:cd:95:a4:de:4e:66:92:27:1a:e0:
        23:9f:66:18:d6:27:30:9f:d7:fe:b3:bb:28:3e:94:4a:88:20:
        93:18:07:cb:ce:1d:6b:10:f9:bc:b5:e8:c1:32:2a:1d:52:e0:
        39:ba:d5:e4:d6:01:a1:de:6f:11:90:5d:40:bb:73:fe:d9:7c:
        2c:ea:c7:05:85:d1:34:47:83:af:d8:43:37:94:ce:8d:89:ac:
        fc:0f:8c:2f:47:1d:ff:be:bd:7a:c5:e9:76:fc:9d:fc:95:19:
        45:a3:7d:80:1f:1d:c8:52:e6:05:4b:ed:f6:57:5e:82:92:78:
        dd:5d:bd:86:48:12:5f:ad:09:85:9e:16:aa:cc:23:9f:3f:60:
        b9:a6:e1:f1:76:41:2c:89:08:e4:b6:96:eb:20:93:bd:83:21:
        96:a4:55:fb:74:a9:11:2f:97:ec:76:02:4e:97:5c:d8:27:26:
        ee:78:03:1e:df:4b:ed:a6:d4:f9:70:6c:cb:04:29:50:3f:6c:
        a9:32:3e:08:55:89:71:e3:4e:dd:e3:cd:30:a0:5d:29:ff:40:
        d3:92:09:22:56:e2:31:fc:22:39:5b:46:c2:11:ed:93:04:1c:
        65:08:2d:4b:2f:ad:30:ef:ac:f9:b8:c1:a3:9f:5d:8c:c0:20:
        38:16:8c:43:75:ee:52:3b:20:a3:ec:8d:ea:90:5f:cd:77:86:
        42:fb:69:fe:a1:01:c4:4e:3b:55:b8:79:b4:36:1b:f4:be:a7:
        51:0f:ce:53:83:be:3a:68:0e:61:58:1b:18:cc:8b:e5:85:b5:
        3c:89:42:82:31:d4:00:d3:bd:b0:3e:9b:47:23:6a:b0:50:e9:
        37:66:17:d2:82:08:6d:26:07:e7:8a:a0:6e:07:62:4e:61:f9:
        a8:37:ce:91:9a:c9:a1:44:ee:73:a4:a1:b7:ae:6c:19:1d:d7:
        45:a2:15:4a:6f:83:25:c6:9c:0a:d8:78:24:ce:26:80:8d:b5:
        a5:71:09:ba:40:17:7b:43:be:53:84:2f:0f:1e:49:7a:a9:82:
        73:73:55:52:22:83:72:2b:87:5d:31:f3:ef:39:58:18:12:6a:
        8b:30:65:d9:24:85:7d:db:3e:95:9e:99:4a:88:75:bd:21:19:
        e3:bc:2f:a9:cb:75:3c:19:f9:69:c7:88:f8:34:c0:f4:4d:1e:
        1b:40:70:98:7a:d9:04:d8:b1:06:d1:18:21:5e:bb:d0:fc:18:
        e0:c8:85:5b:89:09:19:ad:9d:ee:d8:32:d5:94:ab:93:30:7f:
        52:b3:fc:9e:20:69:48:33:a5:55:7f:f5:d7:77:b0:e6:9f:44:
        09:9c:d4:d8:9a:39:e6:84:fd:77:8e:21:b8:f6:cb:da:a2:af:
        5d:45:0b:97:50:bb:0c:41:09:a3:3b:c6:dd:b5:db:a1:56:df:
        f6:e4:1e:a4:05:ff:30:cb:8b:c8:45:9a:c1:70:9d:25:b2:7c:
        25:36:4e:72:04:2c:0e:07:0c:19:5f:f5:e4:7b:32:3b:6f:87:
        8c:c2:7a:f4:b3:74:f4:63:10:13:1e:ca:a5:bd:4a:99:ca:ca:
        3a:68:c6:d2:7d:1c:86:8c:13:af:23:12:cd:94:91:91:f3:6f:
        5e:85:a3:00:4d:a0:6b:c8:b3:49:51:ef:27:b5:9b:3e:4e:9b:
        9f:98:8a:e1:d6:b0:f1:13:15:90:c7:29:af:9a:9c:ad:d9:0e:
        cb:a0:de:8d:3d:bd:49:27:40:a0:86:3e:be:33:d4:fa:6d:c4:
        22:6b:2d:93:aa:89:70:1a:c0:f7:cd:ed:f5:80:9d:6d:f2:11:
        9b:ce:14:c9:07:40:d5:ec:d2:d9:54:57:d7:31:ef:89:e4:21:
        ca:f6:b9:66:99:8d:d9:96:3b:80:39:c8:4a:e7:10:dd:d4:b6:
        73:85:55:18:6d:98:16:62:b8:3a:57:24:7b:d9:fb:9b:e4:3d:
        39:4e:6c:cf:30:a1:c4:4a:67:d1:e2:c7:84:bf:68:85:c8:08:
        86:ce:b1:e7:f5:2e:58:6d:81:00:09:43:dd:0d:c2:99:6e:e5:
        ec:1e:c5:be:1b:43:3c:15:b4:69:7c:b7:36:75:ec:d2:b0:f2:
        67:8c:0f:a2:c1:f1:04:61:09:3a:d7:04:38:90:99:5b:d2:fc:
        1a:66:89:16:6d:01:93:76:81:38:a0:88:a5:2c:e7:eb:f6:e6:
        6a:ee:bf:6a:a1:c2:d1:cf:58:9a:00:54:ed:88:a3:fb:94:70:
        8b:f4:93:29:d8:d6:82:96:52:cd:aa:49:19:28:04:4b:85:a9:
        11:96:01:83:4d:5b:b0:16:25:7f:ba:c6:f4:e0:33:58:78:36:
        d8:9f:10:26:98:6f:d8:1f:03:5e:16:01:8b:32:6f:22:88:6c:
        16:88:c4:62:aa:23:1f:88:68:65:2c:89:21:28:72:17:6b:49:
        22:39:51:81:99:76:f8:1f:e3:09:66:90:a0:7b:57:57:bf:e2:
        c1:e2:92:b0:ea:e1:35:55:a8:d4:0f:69:d7:9d:4a:7d:25:fd:
        0e:92:bb:27:b1:fd:2f:55:21:6e:1d:ca:6b:37:51:d1:3f:56:
        3b:d8:7d:3e:4c:5b:4b:59:9a:db:1b:75:29:ca:8f:22:fa:b7:
        b7:72:dc:47:4c:9f:d4:bb:fb:dc:a5:ef:d4:08:54:95:85:7a:
        8a:9a:03:b0:cf:6b:7a:a5:50:78:f8:87:59:38:b4:24:c9:e1:
        2c:16:b6:82:b4:5b:73:6f:78:e3:0a:ec:5e:9b:b1:74:87:74:
        80:8f:ea:c8:eb:f8:ec:ad:20:96:a4:ff:44:3a:7a:e1:ba:b6:
        ba:d7:b5:0c:34:89:e1:84:83:2c:1c:b8:4f:2b:62:ec:da:68:
        4b:3a:23:97:0e:6f:50:3d:4c:e1:1a:54:67:5a:6c:97:65:bd:
        3c:19:db:e8:15:fd:19:13:33:c4:e1:59:3c:62:e4:19:09:b5:
        0a:d4:72:2b:60:2b:e3:be:5f:51:d9:b7:c1:56:b2:05:de:0e:
        26:ec:fc:a6:bc:8e:34:82:5f:8b:9f:5c:4e:e0:0f:10:40:64:
        f0:8d:9c:69:20:3b:da:38:07:df:01:41:4f:50:c1:27:6b:cb:
        7a:3b:4c:25:18:b6:d4:fd:ca:ee:47:d1:14:25:d4:d7:e1:3f:
        1a:62:78:74:1e:82:83:84:d2:6c:dd:fb:4c:11:6f:17:1f:35:
        d8:33:df:89:31:63:b4:c9:33:37:0a:ba:d6:f4:9b:2d:95:ed:
        ed:f1:b8:c4:bc:e4:e3:7b:73:6d:02:05:f4:27:69:42:f9:f1:
        11:24:a0:21:fa:14:01:15:94:ea:3a:cb:f8:d8:bc:84:2e:54:
        13:84:59:ac:6b:4b:c4:02:02:a3:c2:cb:a5:f0:4b:c8:05:2a:
        fd:84:5a:dd:b7:d6:30:9a:3f:c2:a3:71:87:d8:f1:68:c0:d9:
        08:05:91:a0:cb:f2:81:af:b8:b5:be:88:07:a6:80:3a:6b:a5:
        9b:27:55:31:1f:33:cd:93:d8:c3:49:2d:f9:92:57:64:f0:a5:
        79:06:5b:48:a6:97:e0:b5:d4:6f:0c:27:84:b1:34:e6:a9:9c:
        2f:8e:9b:95:8b:ed:94:14:61:84:54:fc:09:b0:8e:62:9c:49:
        00:20:e2:55:bb:9e:53:0a:27:ff:47:59:ea:36:52:d5:fd:80:
        fd:a6:9d:1c:67:ac:53:18:4c:37:fa:4e:88:18:b5:24:f7:36:
        78:7b:c0:0c:b5:3d:65:f1:86:34:70:38:bf:19:b4:23:ab:72:
        bd:20:77:8a:58:4b:46:a6:3f:62:08:47:e5:58:57:d1:c2:0b:
        00:eb:5e:89:17:48:5e:a6:ec:c4:00:42:9d:42:6e:72:d7:2d:
        51:c9:0c:5f:a9:f6:80:a7:80:f1:1d:63:2e:8d:cf:29:12:bd:
        9b:c6:3f:f5:bf:34:f2:45:2a:cc:23:20:30:f9:a2:bc:d7:d0:
        b4:f5:1f:8e:ca:9c:9b:0c:58:00:f2:b7:a7:90:5c:72:e3:d5:
        20:fa:38:cc:e9:90:7e:09:57:51:48:0f:ab:ca:9c:b8:ca:d4:
        6b:0d:2f:0d:00:29:6b:5e:4a:e2:4f:c9:f4:98:fa:97:9e:6e:
        da:09:0d:00:ba:3b:97:79:b0:d6:6f:f7:66:87:75:c3:ca:df:
        ad:78:41:e6:51:fb:90:05:2e:aa:72:ed:e6:8d:69:c7:19:81:
        84:8a:65:f3:20:dc:97:f3:bd:a5:5f:d8:51:89:01:81:78:42:
        e5:38:ed:92:8e:d0:f3:5b:ff:cb:29:70:02:ec:d0:e1:cb:7e:
        9f:b5:7b:aa:6f:2d:1c:2a:cf:8e:69:7a:b2:b3:b4:0f:cb:1b:
        3b:3a:5f:c9:44:7b:06:cd:40:40:c0:b3:a5:b2:1d:97:43:90:
        14:c6:96:6c:c6:25:b7:fb:78:2b:fe:b7:db:1b:cb:9f:59:77:
        8e:0f:4f:d3:6d:37:56:11:01:b6:73:ec:a3:d1:27:12:13:0f:
        29:1e:74:c0:88:92:9a:62:65:e1:c2:bd:ea:79:04:d6:ef:71:
        41:43:99:c3:32:2a:f1:69:9f:53:10:5a:bf:93:8b:b9:91:75:
        b6:f9:cb:70:9a:02:87:c0:a8:2c:18:54:d4:b4:96:82:00:d0:
        b2:5b:89:1b:42:02:8a:56:35:7c:0a:40:02:7e:71:e8:3e:c9:
        75:55:c0:3e:18:e4:2d:40:ec:6b:93:c5:56:77:00:d5:c6:4b:
        f9:d7:95:9b:f0:96:67:5b:8d:25:23:64:0c:41:e3:68:3d:9a:
        11:5a:fa:7c:bc:62:30:ce:c3:52:c3:7c:6b:94:51:b6:b9:46:
        e8:1c:df:25:a5:e2:f7:3f:5d:5a:f5:a7:1d:25:24:5a:89:3c:
        83:38:57:7e:aa:62:2f:fb:69:0f:7b:a5:05:57:59:37:63:26:
        60:ed:b6:c4:35:fb:65:9b:1a:31:ff:f7:a3:69:2a:eb:bf:71:
        ef:b7:be:92:39:58:0f:9a:7f:b2:45:81:87:fd:e0:d4:80:e0:
        fb:67:ea:ad:9f:72:1a:d6:72:29:0b:67:f4:cf:be:f6:d2:a3:
        9d:c4:8a:df:c1:49:d2:7c:c7:e5:bc:a8:dc:35:6a:8b:bf:e1:
        8f:8a:03:88:8d:b2:8f:68:82:02:05:5a:71:32:c7:45:4c:d5:
        01:42:c6:1a:25:6d:66:22:33:a9:1a:cb:d9:c0:67:ac:7b:10:
        3c:7c:2b:db:25:26:c2:03:4f:cb:0b:10:47:96:66:8d:cd:5c:
        64:30:ba:65:51:5e:39:d3:7a:c8:60:8c:08:b2:26:1c:65:44:
        51:2a:cd:0a:49:59:c3:0e:de:8f:a0:49:d2:23:aa:2e:5f:9b:
        00:fa:ac:10:d5:bd:1b:ae:2f:d6:cd:e6:7b:27:f9:d4:47:32:
        c4:b9:dc:fd:85:e1:af:87:36:68:37:ad:f3:13:f1:b5:3d:a7:
        b5:5c:32:34:31:b4:84:83:76:1f:89:9c:10:99:8d:ec:c8:77:
        ec:ab:ab:fd:fd:3f:46:10:a7:8c:e8:2d:02:a4:e2:2a:ed:47:
        4f:ff:86:be:17:76:5e:d7:e6:f0:98:30:37:8e:87:f6:e4:40:
        d8:6e:3b:b6:20:13:b4:87:15:bf:63:25:f7:94:f9:bd:85:f9:
        8b:43:47:d8:70:da:86:d8:2e:ff:9b:1f:ec:c9:f2:79:55:33:
        71:47:1e:0d:00:39:09:aa:e3:d7:3d:53:cb:05:b4:86:11:1c:
        e4:f3:e9:f5:6d:3f:bb:7e:78:50:6e:e9:fc:79:39:94:45:24:
        f0:5f:4c:5c:e9:12:2e:8e:b9:32:05:b4:2f:6e:73:cb:86:71:
        c9:60:39:9f:4f:0b:8f:d0:95:8b:6a:46:d4:15:28:ac:8b:51:
        73:f3:6d:7e:23:ae:89:1e:9f:1b:9e:1d:cf:0a:14:1a:64:39:
        16:b3:66:84:fd:10:65:33:9d:1e:4d:c9:e6:7c:f9:fe:5f:29:
        57:58:73:e4:30:53:af:39:fa:63:71:1a:33:e0:d4:ce:fb:3a:
        14:56:1d:9a:87:49:a0:02:41:c4:80:5f:06:0e:68:45:4d:1c:
        ca:35:4e:22:dd:e0:16:3d:a5:ef:e6:52:0c:32:7a:6a:f4:36:
        60:5e:78:39:61:f8:1e:34:e3:9f:ef:a1:44:6e:8d:80:fb:56:
        05:e5:17:13:54:a7:25:be:78:5b:35:55:42:d7:be:5d:d9:22:
        d9:ec:fc:01:9d:4e:47:e7:34:fc:51:ce:41:b8:be:d2:af:c6:
        17:c6:49:40:9c:f5:b5:6d:39:d0:a0:f9:7d:1d:39:eb:ea:26:
        79:3f:aa:96:8e:45:cf:68:e5:b4:30:0e:82:d7:2c:46:5b:6c:
        1f:a4:37:dc:04:d6:f6:10:9a:5e:27:a6:d3:b6:9a:7a:d3:e0:
        44:f1:59:96:36:f1:35:6f:60:62:78:5c:27:93:80:7e:14:16:
        32:d2:99:0b:fa:b3:eb:79:bc:0e:06:7e:06:26:46:8d:03:6a:
        f7:e0:86:d2:a9:3d:af:3c:90:ad:1e:b1:e8:90:4f:e0:32:47:
        59:a6:15:8e:f5:c6:98:45:fe:a0:87:e4:0d:d8:dc:ae:51:18:
        a7:6d:7a:f7:b0:51:b9:e0:9f:4a:91:d4:98:b3:88:a4:68:d9:
        6b:8c:09:f4:90:2d:f0:bc:c0:4a:79:13:73:b1:6e:ef:bb:0e:
        9a:3e:db:ce:ce:aa:f2:c0:e4:9b:86:2a:9d:e1:3d:87:38:d8:
        7a:bc:d5:d6:2a:6c:91:2d:1d:0a:3c:4d:53:15:06:cf:98:90:
        8b:9f:93:a4:04:f7:48:17:f1:7b:cc:dd:5f:33:3a:f3:ca:28:
        85:57:8f:c5:77:aa:53:43:c1:70:c3:e2:23:0b:d1:20:64:58:
        e4:11:2f:b8:c3:1d:8f:d4:b9:72:92:00:ef:68:3f:81:9d:6e:
        25:ef:42:1c:47:f6:be:ab:ef:fe:f2:d4:ee:80:bf:01:5f:90:
        f9:c8:e0:e9:c4:5b:3c:ab:4e:e6:35:cf:4f:a1:99:db:8e:07:
        25:ee:65:31:14:a3:d1:6e:f6:4e:7d:fa:df:c5:59:c7:29:bc:
        4e:fa:4e:a6:9f:a0:92:ce:4d:50:e4:87:78:33:c5:ed:ae:c9:
        b2:8a:07:66:64:0a:0a:0b:09:5f:98:7f:b4:a5:2d:ad:7b:50:
        9e:dd:02:62:0c:2a:1c:28:3a:ef:67:b1:c3:bb:4d:7a:6b:42:
        12:43:19:eb:45:c5:c4:b4:9b:41:c5:ea:a9:7f:21:c5:ab:b4:
        93:39:09:4f:d4:86:90:c3:7f:04:f0:51:8a:c6:af:86:ed:fe:
        7f:96:c1:8e:8a:46:6a:73:1d:bf:3a:ce:ea:a8:98:f0:84:11:
        85:18:b5:8f:53:44:77:ac:1b:58:2c:e7:a8:92:a5:f0:9d:b5:
        56:4d:8b:8a:35:7e:45:7e:06:6e:39:fe:7c:e1:00:d8:b7:bc:
        92:1f:7c:41:08:93:ec:7c:3b:8e:76:c2:39:46:98:ee:72:75:
        d6:e6:2a:b8:df:0c:03:6e:12:3c:ec:80:ba:c3:dd:26:14:80:
        51:5d:ab:6a:cf:36:e9:69:15:e5:51:0e:d4:15:47:81:39:2b:
        17:b6:57:3c:a5:11:a0:94:f0:d6:48:2b:dc:4a:7b:2f:d2:8d:
        5a:a8:d7:2a:3d:0c:73:d3:57:d2:fd:ee:66:85:75:53:75:0f:
        2a:f4:e5:78:50:0d:c2:d2:13:86:87:f9:45:99:86:1c:0a:16:
        27:a3:17:37:38:20:4c:93:87:aa:5e:9d:c0:e4:5c:b4:db:07:
        3c:d8:64:f8:0f:bf:de:71:ae:90:5a:58:4f:6c:8e:29:e2:01:
        06:7b:fc:71:b1:38:6d:16:64:49:12:d9:27:83:a1:66:eb:d8:
        ac:07:4b:bf:4c:16:30:79:6e:18:27:98:50:d7:2b:87:14:67:
        fd:ea:57:6e:cc:cc:19:51:f9:56:10:d9:c9:be:ed:7a:21:43:
        cd:6e:79:6b:b6:a1:14:01:fd:4f:c7:de:e9:d3:1c:1a:fb:c0:
        23:11:5d:be:b2:a0:df:96:d1:db:a2:45:de:bd:b8:47:5c:97:
        22:88:d1:31:53:65:cf:68:2c:9e:1f:35:ae:45:20:57:bb:32:
        64:ff:3e:bd:8f:dd:3f:33:cd:b0:16:be:4b:ff:4d:22:e7:b7:
        ea:22:9a:6e:07:3d:49:b3:5a:4e:6a:49:d5:8b:4b:65:a9:03:
        da:dc:4e:fb:40:ea:ef:7e:9d:90:bd:82:77:4d:c5:66:07:73:
        2d:c4:98:c4:17:14:be:68:fb:57:f0:96:ab:08:8a:3a:ef:be:
        86:5d:42:57:bd:24:1f:e3:0f:b9:d3:38:19:af:e7:8b:d4:d1:
        19:53:fa:d0:58:06:70:d9:38:2f:f9:ad:11:ad:54:fd:28:65:
        fd:b6:fb:2a:b9:07:10:68:0b:55:ce:07:a6:ac:61:5b:b7:7c:
        5f:ea:75:98:53:96:fb:7a:f7:62:f6:be:d6:72:ed:9e:98:d5:
        cd:49:79:d9:3c:e1:87:cc:38:7d:dc:36:b4:74:1b:0e:d4:d8:
        be:07:c6:3c:e6:4c:ae:ac:22:9b:21:ba:f2:25:d0:a3:02:35:
        07:6a:fd:75:a5:ee:fa:77:77:fc:f5:e2:06:c5:88:c6:c5:b4:
        65:60:5e:9e:5a:f1:1d:1a:05:8b:6d:45:62:8b:9c:ee:76:04:
        11:93:4f:46:64:3b:76:d9:5f:ba:8d:74:4b:0f:00:ff:59:61:
        d5:60:8a:3d:ff:9c:aa:61:00:53:55:a8:56:64:86:4a:ce:5f:
        b0:95:75:96:8a:82:71:cb:05:5a:8c:47:a0:3c:a4:99:ce:a2:
        cf:f3:ed:69:9f:02:95:e7:c0:44:50:69:9f:42:e7:82:3b:68:
        5f:4f:ba:bb:21:b4:6e:4d:bd:ad:c0:8d:4d:a6:24:46:c3:0a:
        d6:0d:a0:96:7e:97:ff:81:30:dc:a9:09:69:d1:c4:b4:1b:0e:
        d5:b8:1d:5b:6f:62:56:66:0b:de:ff:6e:73:b1:e1:1f:fd:8c:
        55:b4:71:58:f8:c3:7e:61:29:a1:9c:7d:29:8e:49:54:6e:64:
        67:63:33:57:fd:d9:81:b0:91:0e:53:27:f1:b7:0c:08:de:c6:
        8c:34:9b:69:6c:63:e2:7e:41:6e:aa:fe:34:93:70:bf:ef:38:
        9d:29:ca:12:78:57:e2:6a:43:37:7b:9c:03:91:83:8a:7f:b3:
        23:d9:a7:e0:8c:3f:41:81:24:29:48:fe:fb:0f:7f:25:56:4d:
        a5:b3:4a:6d:19:73:a7:e0:c5:33:29:64:93:d2:3c:99:ee:d6:
        5e:33:31:71:ce:9f:be:50:d7:60:d0:92:40:1d:a3:40:32:58:
        cd:a3:38:8c:7d:2d:3e:70:02:ef:85:43:63:15:80:ee:3c:eb:
        41:99:0a:5b:18:06:6c:2e:2f:13:b6:0f:47:3f:6e:17:44:21:
        d3:6e:c6:50:09:54:00:48:71:c0:a0:5a:fa:26:de:78:0c:63:
        17:60:59:1e:6f:33:29:5a:08:81:5d:cb:4c:45:dc:4d:1c:a1:
        e3:e6:98:92:42:20:62:6a:4c:b2:f1:65:5b:ac:33:bd:3f:24:
        00:ae:44:d4:21:ee:5b:be:43:8f:df:88:75:22:7e:cd:8a:70:
        31:4b:77:b4:b7:c0:67:0d:45:ab:1c:30:f9:cd:de:12:f2:a5:
        14:41:8b:3f:74:f5:f3:43:ed:b7:0d:28:37:81:18:f9:f2:ef:
        18:e2:aa:f1:35:52:90:8d:9d:dd:11:1d:3b:0b:b5:96:41:76:
        70:c3:8f:14:aa:16:c7:90:6d:03:33:c0:eb:09:86:49:bf:7b:
        dc:ee:2e:0d:81:ca:e3:76:0d:31:b4:0b:3d:12:18:0a:6e:bf:
        fa:18:d7:21:b8:3b:ad:2f:c5:c2:03:72:4d:c9:04:6e:06:94:
        50:37:ea:b2:62:11:93:7f:ad:91:9d:af:24:37:61:85:d9:b8:
        41:9c:62:e9:17:12:3c:b3:08:41:cf:b0:13:6c:3b:d6:e9:11:
        60:bd:c0:aa:f7:8a:be:cc:7c:48:ce:ad:d4:83:a4:60:46:34:
        f4:c6:35:22:4c:22:76:38:66:2f:54:f9:a2:fc:6b:e2:cb:38:
        ff:3d:ad:18:a4:a5:78:8f:9c:d3:de:c2:70:2d:ac:fb:a3:1d:
        fd:89:33:c1:d2:5e:50:5a:b1:ff:6c:b2:87:e3:20:da:09:af:
        73:d9:a0:f0:07:53:3d:01:d4:63:32:5a:15:14:da:12:77:6c:
        17:27:67:13:8c:0b:74:6a:98:40:11:45:22:87:8a:8e:3e:dc:
        1d:ad:4f:6c:c1:cc:7e:fa:a3:18:f8:42:cd:a9:28:33:7e:59:
        c4:65:36:b1:47:cb:7b:f4:db:77:51:20:43:e9:b3:db:21:d6:
        5b:a8:67:89:60:c8:67:e6:e6:95:2a:5a:ec:50:4e:ae:ff:79:
        8d:37:3d:a7:5c:b9:a6:cd:a8:68:50:17:2e:12:47:0c:b3:24:
        d3:52:6e:5b:81:4c:06:a2:f1:06:f4:24:64:e3:c6:83:19:1c:
        a9:07:7e:02:bd:7a:ab:2c:49:94:39:9d:d2:de:69:6d:43:fa:
        9a:59:08:34:ce:5c:29:12:71:a0:24:81:c8:7e:d6:77:30:8e:
        57:0a:38:19:e0:e4:f1:6c:ad:03:f5:09:26:b9:fe:e2:55:df:
        69:00:73:25:27:0c:21:c5:e1:7f:5b:c9:fa:53:77:80:f7:ee:
        71:c4:41:92:bf:2f:10:19:51:7f:68:8c:35:b2:63:49:be:eb:
        b9:07:66:fe:51:2a:ae:28:69:00:59:ec:eb:3a:52:54:1f:da:
        91:ca:b0:5b:72:c0:44:6e:46:8c:17:e5:51:20:ad:03:a8:d5:
        e7:a6:c4:b9:21:a8:ac:88:26:d9:43:10:84:76:14:78:d9:a3:
        4c:2b:74:41:e2:32:3e:47:31:eb:fc:67:e1:74:57:f1:e3:9b:
        15:2c:eb:b7:ac:3b:f1:9a:1a:2c:ab:04:c1:57:25:7c:e5:bb:
        2c:8e:56:5e:e5:99:77:05:d4:50:95:01:3e:ca:d7:8b:fa:37:
        11:dd:c1:35:f1:c1:cb:87:ca:55:74:45:3a:7e:81:e7:4f:42:
        d6:cb:6b:29:8b:21:55:09:db:ac:8a:57:34:45:e7:66:e4:d2:
        4f:d2:66:fc:34:8d:61:69:9a:f3:0e:54:1a:d6:cd:08:d5:cd:
        36:c0:02:46:18:1b:d2:16:0c:f1:f4:7b:7d:c8:89:23:35:7a:
        00:5d:7e:21:9d:58:fb:32:b1:9d:af:87:a8:8d:bb:b9:cd:b0:
        94:b2:14:72:88:42:12:50:b6:fe:ae:95:8c:ff:25:84:70:0d:
        cb:33:1f:f1:e7:e4:4d:27:d0:29:79:e8:92:d2:56:0a:c1:0e:
        60:c1:0f:c8:66:49:71:77:49:88:c5:5a:e1:86:93:f5:1a:24:
        6f:f4:0e:d6:58:f7:5c:68:7b:8f:9e:ac:4f:f2:2a:a2:df:d8:
        52:6f:80:96:0a:c1:f5:6c:6d:e6:c5:6d:7d:ca:13:2b:77:15:
        f2:8a:e5:ef:a9:ff:1c:f1:77:c0:e8:c0:f7:3b:5b:d1:b8:44:
        f3:e1:e2:18:d7:f9:d0:5f:24:48:70:cc:d6:0c:dd:9c:99:34:
        96:b9:f4:1b:8c:4e:5b:44:5e:96:11:a4:3a:62:d7:17:2b:19:
        e1:6f:4c:0d:c2:1e:96:7f:de:4b:5f:af:c8:84:b4:6e:92:e3:
        f1:d4:9d:4b:dd:bc:6c:c5:09:78:2a:10:71:fe:06:f8:75:c3:
        61:27:2a:6b:c4:a2:39:68:a3:ea:21:9e:1f:41:14:93:d9:88:
        e5:e5:9d:19:e5:a5:40:4a:5d:e7:c7:e4:0f:e8:a2:ce:89:42:
        6f:2a:7c:db:06:90:27:22:b8:54:fa:9a:59:fb:17:68:c5:27:
        a9:4a:57:a5:9c:8d:58:bf:a0:62:3b:4d:df:18:28:31:7c:17:
        30:37:d5:39:f1:92:2f:49:93:36:42:76:b6:c1:a1:ea:ca:89:
        7f:c6:77:f6:ba:21:b7:83:f2:79:aa:9f:50:79:48:a7:87:d9:
        4f:35:18:c4:53:11:95:c6:00:8e:67:66:d5:cb:c9:c2:14:37:
        61:26:49:e0:f4:40:b8:42:94:d8:0c:7a:c5:1f:63:01:6c:7b:
        b6:54:a5:ff:f8:08:0b:d2:16:d1:b0:79:9d:73:2c:01:7f:df:
        e6:93:27:70:82:7e:44:bb:6e:74:90:ba:c6:58:12:c6:65:7e:
        f8:9f:9e:e8:6d:ff:3d:cc:dd:8f:aa:10:3e:fe:60:b3:e0:5e:
        74:c0:82:bb:2a:e4:f9:5c:35:6d:81:ac:84:38:9f:f9:af:dd:
        ab:a2:ee:c6:48:9d:d2:fd:98:5a:e9:ba:15:3a:91:1b:bd:0a:
        cc:6f:1b:6a:ce:9a:5d:45:1d:bf:6a:53:e9:bb:ca:3a:02:32:
        6d:b1:30:b6:6f:c3:28:6b:75:fa:8b:7c:9f:6a:de:ae:b6:bf:
        e8:11:a5:04:53:10:5d:65:f3:e5:03:0c:e3:c4:17:47:f1:25:
        87:73:3b:3e:c4:0c:c3:ac:d2:95:ab:39:ce:bb:1b:1a:c4:0c:
        32:c7:bf:30:f9:38:cf:46:b6:f8:ab:28:6a:ce:3f:5d:62:5c:
        85:58:61:c4:3d:53:0e:65:ca:f8:29:db:ed:53:4e:4b:b4:9b:
        87:4e:86:71:77:9a:35:4e:ca:bc:80:a1:5d:83:49:1a:24:99:
        5c:17:53:39:c4:7d:6b:65:a8:9c:79:7b:80:12:3a:6e:16:6c:
        32:53:4e:0a:9e:e6:b3:67:60:b6:f5:05:ca:a7:23:0a:23:df:
        80:c4:c5:da:ce:86:f1:6f:0b:0f:54:24:6b:e1:7b:84:a5:29:
        27:6c:f5:a1:a8:c1:47:1f:fd:01:c8:f6:22:1e:1b:65:52:ec:
        60:cf:b7:9f:4d:d3:a7:0a:db:ed:85:aa:c0:23:09:23:0c:59:
        c0:c4:4e:36:d4:0a:b2:ed:fb:ec:ed:ae:6d:1b:bb:5c:ec:a4:
        d7:c1:dc:74:d0:31:8a:be:74:cb:ba:b7:49:32:4b:6b:21:12:
        bd:a2:a1:fd:8e:95:91:09:f9:5e:c5:28:77:5d:60:6d:8c:f1:
        a0:53:52:47:4b:3e:89:c6:b0:8a:ac:d0:ba:7e:1c:d1:72:0d:
        38:73:fe:9b:56:48:8d:35:5e:2e:04:d1:f0:fd:d7:77:de:7f:
        af:3b:28:fc:ca:e0:bf:1e:de:12:48:14:65:21:bf:e1:ff:eb:
        d3:4b:c6:32:8f:cb:98:97:96:94:f0:ba:b4:7a:99:29:16:28:
        89:8d:07:5c:f7:b5:f4:bf:c3:cb:89:be:be:b6:04:b1:d4:cb:
        0a:39:79:ad:f2:6e:cd:82:f7:d2:85:8b:cf:79:6f:66:ad:df:
        48:16:a8:d7:24:50:f3:cc:ea:c8:db:2d:61:e1:81:90:f2:04:
        66:60:7a:03:2c:be:68:e9:23:8a:a7:89:eb:2d:f9:b1:30:28:
        04:93:b1:aa:0f:11:b4:0c:07:2f:7d:cb:fd:ca:2f:ba:2c:8a:
        2c:29:5b:af:91:55:32:66:31:16:a0:3e:f0:c1:34:77:18:14:
        65:55:fe:28:7c:4d:16:9e:88:1d:f2:0c:f5:ba:f3:01:ee:80:
        b1:2c:57:5f:9e:c9:af:aa:ba:5f:b7:29:a2:07:0b:4b:cf:59:
        79:de:6d:3e:1f:a2:ec:ff:47:1c:c6:91:35:64:31:1a:05:0f:
        57:dc:80:9c:c4:5d:32:26:e8:9b:80:58:25:64:85:03:1e:af:
        ee:68:fc:12:6a:ce:eb:81:84:80:59:c7:ee:49:0e:d4:e7:a3:
        23:2a:f3:3c:1f:4b:bf:8c:1b:19:83:b5:ea:5e:38:69:d8:87:
        42:97:28:9b:17:c2:42:89:47:34:16:42:a0:e3:4a:45:d2:75:
        61:d0:fe:40:b4:d7:f9:2d:cc:a8:52:c4:ea:45:f5:6f:b6:cf:
        79:1e:0a:e5:22:65:04:5d:6f:a4:56:59:11:22:69:e3:f4:44:
        fa:90:15:9a:7e:6b:45:1a:85:18:2f:02:2e:ec:5b:87:0e:11:
        6f:f4:48:19:40:b7:37:0b:ff:9e:51:8c:ee:89:ab:fb:73:c5:
        43:72:43:85:ca:b8:bd:0a:32:71:53:5e:8e:f6:12:c7:bf:74:
        be:4c:33:4b:5e:32:79:1c:8b:24:cf:7f:d0:ae:02:a8:0c:b2:
        18:91:10:83:ed:76:a0:64:1f:db:fa:ad:ec:2b:65:d6:f2:70:
        73:af:f0:cc:53:ed:ae:1d:00:67:1e:7c:3d:f2:d8:17:31:50:
        71:6f:d7:88:82:08:5d:6f:e0:21:af:14:c4:be:07:8b:c6:5c:
        00:81:0d:4a:d3:5b:ab:be:da:1e:a7:e3:68:df:16:9b:ec:16:
        0e:ae:2c:6f:69:27:5a:e2:97:b9:3b:ea:0a:b8:5c:77:a6:85:
        66:ab:be:23:4b:b0:24:74:72:65:04:c5:ef:36:ce:96:c2:40:
        c4:70:9d:42:2f:59:ea:93:a6:cd:71:e8:c3:41:d4:f6:b9:62:
        01:df:9a:46:28:d3:7f:cd:a6:f1:65:b4:23:4e:6a:e0:4a:29:
        38:35:fd:69:1a:c0:5a:d9:7f:a6:40:c1:df:c0:19:67:d1:f2:
        3b:85:e4:82:2b:d3:c0:42:cf:1a:28:fc:eb:b2:8b:9f:51:7c:
        8c:22:2c:ba:14:6c:14:46:31:d8:6e:c3:6e:08:92:f9:5a:a3:
        64:49:51:03:f8:47:37:3b:a5:37:b0:98:87:d3:b8:d1:11:a4:
        03:8b:5a:3b:aa:bd:41:6c:ea:70:e3:d9:1c:3d:cb:fe:bc:f6:
        e2:e1:77:78:6e:84:7b:de:22:98:90:09:55:5b:40:18:23:3a:
        17:46:26:40:42:e7:a0:d1:71:01:5c:2b:a6:1a:62:e5:df:93:
        34:8b:85:ec:b4:c5:6f:1e:8a:4e:a9:f1:a8:11:16:cb:04:ff:
        f8:05:ad:d7:2b:9a:21:37:a6:44:18:4c:08:37:96:d1:b1:d7:
        2b:2d:cd:65:9a:8c:7c:d2:45:9a:c5:a8:21:a7:d9:d6:7b:01:
        64:85:7c:11:bc:48:0e:fa:1c:db:95:8f:91:e9:df:aa:3e:8b:
        28:f8:1b:42:d3:77:0a:4d:14:d6:7a:67:6f:ea:bb:54:e6:0a:
        5e:14:1d:14:ce:d1:28:51:83:2f:9f:ff:d4:78:19:15:1c:a7:
        fc:13:89:78:6b:ec:d8:3b:3d:a4:64:a0:d3:63:f2:9a:78:4e:
        0a:ba:76:ff:6e:50:8c:df:89:ce:59:ee:11:fa:d6:e1:5d:79:
        6b:59:9a:85:b0:45:f3:a9:72:8e:50:bd:5b:cd:a1:18:ee:63:
        82:2e:b2:64:c4:9c:07:7b:b9:29:e1:f3:5c:56:e6:33:53:1d:
        5a:3e:4c:30:e6:13:cb:6e:fd:28:9d:f6:47:c3:d7:74:07:07:
        dd:a2:80:db:01:88:65:8e:ac:dc:65:61:60:86:84:75:6b:b1:
        18:81:fc:69:f1:47:8b:33:74:98:9d:77:f4:2a:07:27:13:53:
        74:52:e1:b7:f7:c1:ca:4a:d2:80:e6:ff:b8:e1:50:70:05:05:
        db:66:3b:fd:3e:24:23:f1:bc:f5:19:8e:eb:b1:75:9a:e3:7c:
        32:93:6b:33:7c:1f:06:bd:3b:aa:8c:4f:39:c4:76:1e:84:cd:
        3e:8b:f1:39:c4:70:5f:0f:40:67:9c:00:b9:41:e2:f2:9d:fe:
        bb:54:ef:82:28:5c:90:68:1d:d4:ba:6f:5b:5e:1f:14:8a:b1:
        d3:c7:db:99:3e:de:6f:85:cc:ae:e6:dc:0e:a9:e9:da:d2:ee:
        00:4b:cf:65:be:ec:94:56:c1:37:13:94:4d:71:a0:58:a2:84:
        ce:8a:86:02:15:34:2f:01:71:79:04:87:28:dc:12:81:9e:0d:
        ee:7d:28:cf:69:d7:af:d1:a1:32:c6:55:11:c7:a2:8a:33:9f:
        0c:f9:e1:20:40:71:73:5b:60:dc:28:d0:b8:6c:06:8a:ed:74:
        4c:8e:29:5d:30:5b:d2:2d:c2:2d:b6:dc:d4:ff:ab:87:a6:6f:
        b1:c1:4e:91:76:29:02:2f:70:c6:b6:ab:3a:18:5c:7b:99:c0:
        fa:58:66:bb:1b:ac:af:34:1d:cf:5e:cc:78:80:d0:4c:39:f1:
        0c:45:74:99:8a:8b:8f:e6:2c:5f:6f:da:49:7f:80:e7:85:6e:
        ba:75:0a:ce:f0:2f:bc:5a:0c:b2:23:a3:81:dc:3e:48:cf:1b:
        0d:ea:3b:f5:29:ce:53:05:8f:3a:81:04:dd:fa:cf:10:df:41:
        2e:45:17:7f:f4:1e:31:6e:64:48:6e:31:a1:d2:ec:61:fd:8b:
        99:ed:1d:db:15:32:11:8e:36:97:96:1a:28:fd:6d:a9:88:d7:
        ff:21:bd:47:93:27:96:2b:ce:6a:4d:c3:f7:9b:ae:9b:35:e7:
        ae:4f:e3:47:f3:b7:ea:6b:79:31:f1:c7:53:fb:1f:83:1c:73:
        e3:eb:a9:19:d1:c8:3a:dc:ff:b0:39:cc:b3:5f:0e:a8:1c:82:
        b8:9f:4c:7c:35:32:d3:da:88:4b:d6:fc:c3:dd:d4:13:ad:38:
        04:a8:3b:c4:4a:8d:f8:69:85:2b:bc:94:96:f0:c7:9c:d1:84:
        11:a6:2f:eb:f5:6a:7a:d4:a9:44:cc:dc:cb:52:78:83:ee:8f:
        f5:60:49:a2:c4:b0:3c:6b:0e:78:c9:f6:87:6a:1b:6c:f2:20:
        72:18:cd:9d:05:da:3d:a2:b7:5b:39:90:7b:12:b4:d6:0b:eb:
        d0:98:8a:80:62:45:c0:8e:a5:a8:94:83:25:e9:20:8d:a4:e1:
        3c:ca:83:f7:9c:e4:49:4c:94:50:e1:e8:c7:bb:b4:e2:99:9c:
        72:86:4a:e7:20:cd:61:ff:10:a9:6a:83:92:cb:d8:21:d6:c1:
        c0:d7:2d:c1:45:47:85:28:05:56:67:33:56:ff:f5:bd:fb:c9:
        75:f3:ac:f0:5f:d8:c1:35:58:de:ca:98:ea:f0:03:fd:10:35:
        92:f7:b1:36:d4:b5:f6:2d:32:59:f3:6a:fe:88:d1:6e:ca:65:
        fd:fe:a4:f8:a7:97:b8:e2:85:ac:3e:94:43:39:2a:0b:1c:d1:
        bc:80:fe:81:a1:f6:1b:58:7c:51:b2:3d:c3:74:93:db:5a:31:
        96:d1:87:db:57:99:05:3d:d4:6e:0d:7a:8b:5e:d8:b7:8b:b4:
        0b:71:5d:f2:7f:44:73:19:68:11:19:7c:09:e0:ae:29:76:59:
        e1:7e:27:17:f9:14:5e:c2:ab:89:e1:4c:20:ff:ce:ff:b8:55:
        59:dc:6a:59:a6:76:bd:66:67:e9:47:de:5c:70:18:ca:b3:96:
        be:c8:0d:54:48:6c:04:20:98:c1:ca:2a:46:c2:e5:70:56:25:
        1d:63:ef:9a:f0:0a:d0:f0:f1:26:79:da:67:41:55:0b:fe:ca:
        36:11:51:04:57:69:46:a8:c1:a6:d7:81:ab:2d:b4:45:b7:f0:
        69:cb:13:95:5a:55:8b:f7:f6:29:cc:2e:3e:75:e0:63:d6:72:
        c6:01:bb:2c:3d:22:0c:f8:e0:fb:67:41:32:5a:c8:9c:c3:e6:
        8c:91:46:29:2e:5e:3e:63:83:be:9a:b6:2f:42:4d:29:9e:1a:
        a5:f0:ed:74:65:b7:95:ce:bc:37:96:4f:9d:bf:f2:ff:65:76:
        63:77:30:fd:6c:cf:57:45:b1:e3:f0:72:52:78:00:d4:9d:d4:
        12:34:66:55:be:11:8d:c3:10:e9:d9:2d:db:36:d6:28:3e:93:
        db:cc:3c:cc:11:96:60:e7:2d:de:9e:90:02:47:dd:aa:9b:d7:
        bf:e6:7f:9e:6f:4d:88:fd:7d:e1:3f:52:88:ff:d4:9d:34:b8:
        74:ac:6d:13:d1:44:b3:e0:01:90:bd:e1:e3:94:aa:f3:9b:32:
        db:e6:91:8a:6a:7f:f4:cd:df:a4:83:8b:7d:4f:fd:e3:28:d7:
        9f:c6:64:92:05:03:22:38:17:b5:0e:03:87:73:90:82:bc:5d:
        e5:83:f8:9d:25:f6:1a:13:a4:40:f7:48:61:a9:f1:dd:07:99:
        74:0a:cd:6a:30:38:00:f2:0a:5a:c8:fe:cc:4c:c4:4d:67:6d:
        3f:04:24:d1:a4:f0:87:45:65:45:45:c6:7f:c7:ca:f9:90:79:
        7b:68:ce:cb:7a:d2:47:5e:06:82:e9:53:92:6b:7c:36:82:7b:
        15:67:fd:4a:6b:59:1f:a3:f4:e0:ed:c7:7f:94:fd:40:3a:bf:
        31:b7:98:53:8e:2e:4e:15:9a:8d:8c:16:17:b6:8a:61:08:ca:
        90:f1:7e:95:10:ae:08:c7:05:87:c8:e9:a9:cc:ac:2c:71:5e:
        2a:a8:5b:6a:e3:2c:85:ec:1e:9c:e3:c0:08:1b:17:2e:72:b1:
        44:e3:aa:b4:f0:5a:be:93:96:ad:da:84:52:bc:02:82:be:b8:
        36:88:61:32:06:5d:cf:20:c0:df:07:a1:14:f8:f4:8c:b5:5b:
        2c:d7:af:2b:1e:91:b6:d5:12:2b:68:23:12:8e:f0:c0:38:38:
        56:b4:93:65:f1:e5:80:43:3a:c4:90:21:ce:3f:e4:5e:e7:2e:
        4f:45:c3:e0:0a:81:37:5c:18:00:83:86:97:f3:6a:6b:7a:20:
        a8:4b:04:74:fa:e4:b8:90:84:da:5d:d6:b4:4c:fb:f1:98:43:
        04:8c:56:86:22:b1:73:06:31:76:8a:3d:35:e4:39:9e:b8:22:
        54:66:de:86:78:9a:13:24:56:52:bd:b5:c3:e9:65:c9:06:2e:
        bf:5e:ab:c9:d6:65:93:c5:a9:c0:48:b5:bb:b0:73:23:35:8a:
        f1:a6:6f:f8:0e:28:33:dd:6d:c2:71:7d:a9:7b:18:2c:f4:ef:
        72:c5:46:32:65:a9:80:91:e6:8e:57:dd:fa:38:88:8c:89:e1:
        50:76:7e:a0:98:a2:8b:1e:5a:f6:4a:34:3f:f9:70:05:b2:ba:
        6e:62:55:29:62:1a:31:3b:bf:71:f5:69:d9:73:fe:e4:1b:b9:
        46:aa:9e:db:fc:c5:36:9d:4d:06:9c:54:18:cf:e2:b4:af:2e:
        b3:5a:af:32:96:0f:89:ce:da:6e:13:58:0b:10:d3:84:af:f9:
        22:a7:cb:88:67:b4:b0:b4:24:82:1a:7a:7d:b0:32:5b:8a:73:
        07:90:b8:59:13:25:56:b5:ec:ec:56:e5:8e:ba:97:61:23:69:
        71:13:be:b2:c9:53:a1:78:26:fc:5c:fe:c3:02:d7:50:95:a8:
        8f:6e:81:b2:a2:de:a3:da:5a:66:e2:52:9f:6f:19:7e:49:28:
        f5:b4:96:04:c2:a8:56:a3:1f:b8:6a:61:ec:12:25:e3:64:2a:
        f6:e1:9e:fb:d2:7e:5a:ce:f1:2c:22:81:c8:7c:ae:fb:69:a4:
        48:13:18:77:6d:c1:fb:da:0f:69:8d:1e:6e:93:75:23:ef:ae:
        a8:f1:75:45:16:5b:13:1d:12:64:e4:5a:b2:57:af:80:e0:c3:
        22:c8:e9:62:b8:8f:a7:eb:c7:c4:cd:c8:64:8b:e9:de:cb:e1:
        52:81:05:9c:16:77:4b:df:4e:aa:7f:0f:a8:ed:1e:ac:84:4d:
        96:86:5f:93:7f:7f:f6:2a:b4:b1:4f:94:5d:b3:8e:c5:ee:e4:
        59:33:6a:b4:2f:29:65:08:6a:af:6e:2f:80:c9:ff:90:a6:e9:
        3b:37:04:2d:66:b0:bd:45:78:d3:de:d4:ce:f4:67:95:2e:4f:
        bc:ef:00:85:cf:0a:a6:f5:c8:7f:12:58:09:cd:52:8f:f4:0c:
        b5:c1:40:7b:7b:46:c2:ac:19:76:1e:dc:ab:69:93:88:9b:4e:
        0f:fd:25:c5:7d:7d:18:3c:5c:62:96:0c:9a:e4:af:5c:4b:81:
        1c:ec:30:bc:a0:5c:45:3b:d7:8e:30:a8:a4:84:0e:12:58:68:
        90:ef:a1:09:0c:5b:32:11:1d:cd:12:94:d6:3e:37:03:4d:5a:
        cd:14:c1:2a:17:75:20:90:26:6a:ba:1d:78:83:11:fc:94:4d:
        08:ee:0d:09:75:21:47:36:f5:ff:57:42:15:19:eb:33:10:d7:
        bf:ea:94:97:41:d2:c5:67:32:e1:8e:97:c1:58:6b:c0:3c:45:
        26:d6:71:06:76:d2:fd:6c:07:f1:4e:ef:18:45:a3:b4:16:5f:
        42:0f:e2:dd:5d:e0:2f:34:c2:8e:72:b8:69:8d:a8:11:1b:3a:
        a9:13:8b:a4:62:af:73:3a:28:dc:c6:83:e7:c1:82:33:e6:c2:
        52:ff:ba:1d:78:61:d7:15:e0:b4:ad:90:5a:f3:1e:f4:cd:92:
        29:22:f9:7f:ed:99:44:ae:e4:67:a2:2d:75:4f:77:e2:0b:cd:
        29:80:2e:ae:5e:a5:85:a3:a2:09:31:51:82:98:0b:2c:7a:6b:
        96:ef:8d:c0:f5:1f:98:b4:f6:22:b6:21:6e:36:e3:bb:18:da:
        1d:24:46:0d:65:28:b6:6a
~~~

~~~
-----BEGIN CERTIFICATE-----
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
-----END CERTIFICATE-----
~~~

# Acknowledgments
{:numbered="false"}

Much of the structure and text of this document is based on {{?RFC8410}} and {{?I-D.ietf-lamps-dilithium-certificates}}. The remainder comes from {{!I-D.draft-ietf-lamps-cms-sphincs-plus}}. Thanks to those authors, and the ones they based their work on, for making our work earier. "Copying always makes things easier and less error prone" - {{?RFC8411}}.
