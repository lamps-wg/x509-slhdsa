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
    name: Stefan Gazdag
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
  FIPS205:
      title: TBD
  FIPS205-ipd:
      title: "Stateless Hash-Based Digital Signature Standard"
      date: 2023-08-24
      target: https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.205.ipd.pdf
      author:
        org: National Institute of Standards and Technology (NIST)

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

--- abstract

Digital signatures are used within X.509 Public Key Infrastructure such as X.509 certificates, Certificate Revocation Lists (CRLs), and to sign messages.  This document describes the conventions for using the Stateless Hash-Based Digital Signature Standard (SLH-DSA) in X.509 Public Key Infrastructure.  The conventions for the associated signatures, subject public keys, and private key are also described.

\[EDNOTE: This draft is not expected to be finalized before the NIST PQC Project has standardized FIPS 205 Stateless Hash-Based Digital Signature Standard.  The current FIPS draft was published August 24, 2023 for public review.  Final versions are expected by April 2024. This specification will use object identifiers for the new algorithms that are assigned by NIST, and will use placeholders until these are released.]

<!-- End of Abstract -->

--- middle

# Introduction

Stateless Hash-Based Digital Signatures (SLH-DSA) is a quantum-resistant digital signature scheme standardized in {{FIPS205}} \[EDNOTE: {{FIPS205-ipd}} until officially published] by the US National Institute of Standards and Technology (NIST) PQC project {{NIST-PQC}}. This document specifies the use of the SLH-DSA algorithm in Public Key Infrastructure X.509 (PKIX) certificates and Certificate Revocation Lists (CRLs).

SLH-DSA offers three security levels.  The parameters for each of the security levels were chosen to provide 128 bits of security, 192 bits of security, and 256 bits of security. There are small (s) or fast (f) version of the algorithm, and the option to use SHA-256 {{?FIPS180=NIST.FIPS.180-4}} or SHAKE256 {{?FIPS202=NIST.FIPS.202}}. The fast versions are optimized for key generation and signing speed, they are actually slower at verification than the small parameter sets. For example, id-alg-slh-dsa-shake-256s represents the 256-bit security level, the small version of the algorithm, and the use of SHAKE256.

Separate algorithm identifiers have been assigned for SLH-DSA at each of these security levels, fast vs small, and SHA-256 vs SHAKE256.

This specification includes conventions for the encoding of SLH-DSA digital signatures and public keys in the X.509 Public Key Infrastructure.

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

This specification uses placeholders for object identifiers until the identifiers for the new algorithms are assigned by NIST.

The AlgorithmIdentifier type, which is included herein for convenience, is defined as follows:

~~~
    AlgorithmIdentifier  ::=  SEQUENCE  {
        algorithm   OBJECT IDENTIFIER,
        parameters  ANY DEFINED BY algorithm OPTIONAL
    }

    |  NOTE: The above syntax is from [RFC5280] and matches the
    |  version used therein, i.e., the 1988 ASN.1 syntax.  See
    |  [RFC5912] for ASN.1 copmatible with the 2015 ASN.1 syntax.
~~~

The fields in AlgorithmIdentifier have the following meanings:

* algorithm identifies the cryptographic algorithm with an object identifier.

* parameters, which are optional, are the associated parameters for the algorithm identifier in the algorithm field.

The OIDs are:

~~~
   nistAlgorithms OBJECT IDENTIFIER ::= { joint-iso-itu-t(2)
     country(16) us(840) organization(1) gov(101) csor(3) 4 }

   sigAlgs OBJECT IDENTIFIER ::= { nistAlgorithms 3 }

   id-alg-slh-dsa-sha2-128s OBJECT IDENTIFIER ::= { sigAlgs TBD }

   id-alg-slh-dsa-sha2-128f OBJECT IDENTIFIER ::= { sigAlgs TBD }

   id-alg-slh-dsa-sha2-192s OBJECT IDENTIFIER ::= { sigAlgs TBD }

   id-alg-slh-dsa-sha2-192f OBJECT IDENTIFIER ::= { sigAlgs TBD }

   id-alg-slh-dsa-sha2-256s OBJECT IDENTIFIER ::= { sigAlgs TBD }

   id-alg-slh-dsa-sha2-256f OBJECT IDENTIFIER ::= { sigAlgs TBD }

   id-alg-slh-dsa-shake-128s OBJECT IDENTIFIER ::= { sigAlgs TBD }

   id-alg-slh-dsa-shake-128f OBJECT IDENTIFIER ::= { sigAlgs TBD }

   id-alg-slh-dsa-shake-192s OBJECT IDENTIFIER ::= { sigAlgs TBD }

   id-alg-slh-dsa-shake-192f OBJECT IDENTIFIER ::= { sigAlgs TBD }

   id-alg-slh-dsa-shake-256s OBJECT IDENTIFIER ::= { sigAlgs TBD }

   id-alg-slh-dsa-shake-256f OBJECT IDENTIFIER ::= { sigAlgs TBD }
~~~

The contents of the parameters component for each algorithm are absent.

# SLH-DSA Signatures in PKIX

SLH-DSA is a digital signature scheme built upon hash functions. The security of SLH-DSA relies on the presumed difficulty of finding preimages for hash functions as well as several related properties of the same hash functions.

Signatures are used in a number of different ASN.1 structures.  As shown in the ASN.1 representation from {{!RFC5280}} below, in an X.509 certificate, a signature is encoded with an algorithm identifier in the signatureAlgorithm attribute and a signatureValue attribute that contains the actual signature.

~~~
    Certificate  ::=  SEQUENCE  {
        tbsCertificate       TBSCertificate,
        signatureAlgorithm   AlgorithmIdentifier,
        signatureValue       BIT STRING  }
~~~

Signatures are also used in the CRL list ASN.1 representation from {{!RFC5280}} below.  In a X.509 CRL, a signature is encoded with an algorithm identifier in the signatureAlgorithm attribute and a signatureValue attribute that contains the actual signature.

~~~
    CertificateList  ::=  SEQUENCE  {
        tbsCertificate       TBSCertList,
        signatureAlgorithm   AlgorithmIdentifier,
        signatureValue       BIT STRING  }
~~~

The identifiers defined in {{sec-alg-ids}} can be used as the AlgorithmIdentifier in the signatureAlgorithm field in the sequence Certificate/CertificateList and the signature field in the sequence TBSCertificate/TBSCertList in certificates CRLs, respectively, {{!RFC5280}}.  The parameters of these signature algorithms are absent, as explained in {{sec-alg-ids}}.

The signatureValue field contains the corresponding SLH-DSA signature computed upon the ASN.1 DER encoded tbsCertificate {{!RFC5280}}.

Conforming Certification Authority (CA) implementations MUST specify the algorithms explicitly by using the OIDs specified in {{sec-alg-ids}} when encoding SLH-DSA signatures in certificates and CRLs.  Conforming client implementations that process certificates and CRLs using SLH-DSA MUST recognize the corresponding OIDs. Encoding rules for SLH-DSA signature values are specified {{sec-alg-ids}}.

When any of the id-alg-slh-dsa-* identifiers appear in the algorithm field as an AlgorithmIdentifier, the encoding MUST omit the parameters field. That is, the AlgorithmIdentifier SHALL be a SEQUENCE of one component, the id-alg-slh-dsa-* OID.

# SLH-DSA Public Keys in PKIX {#sec-pub-keys}

In the X.509 certificate, the subjectPublicKeyInfo field has the SubjectPublicKeyInfo type, which has the following ASN.1 syntax:

~~~
    SubjectPublicKeyInfo  ::=  SEQUENCE  {
        algorithm         AlgorithmIdentifier,
        subjectPublicKey  BIT STRING
    }
~~~

The fields in SubjectPublicKeyInfo have the following meanings:

* algorithm is the algorithm identifier and parameters for the public key (see above).

* subjectPublicKey contains the byte stream of the public key.

{{!I-D.draft-ietf-lamps-cms-sphincs-plus}} defines the following public key identifiers for SLH-DSA:

~~~
   pk-slh-dsa-sha2-128s PUBLIC-KEY ::= {
      IDENTIFIER id-alg-slh-dsa-sha2-128s
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-sha2-128f PUBLIC-KEY ::= {
      IDENTIFIER id-alg-slh-dsa-sha2-128f
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-sha2-192s PUBLIC-KEY ::= {
      IDENTIFIER id-alg-slh-dsa-sha2-192s
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-sha2-192f PUBLIC-KEY ::= {
      IDENTIFIER id-alg-slh-dsa-sha2-192f
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-sha2-256s PUBLIC-KEY ::= {
      IDENTIFIER id-alg-slh-dsa-sha2-256s
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-sha2-256f PUBLIC-KEY ::= {
      IDENTIFIER id-alg-slh-dsa-sha2-256f
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-shake-128s PUBLIC-KEY ::= {
      IDENTIFIER id-alg-slh-dsa-shake-128s
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-shake-128f PUBLIC-KEY ::= {
      IDENTIFIER id-alg-slh-dsa-shake-128f
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-shake-192s PUBLIC-KEY ::= {
      IDENTIFIER id-alg-slh-dsa-shake-192s
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-shake-192f PUBLIC-KEY ::= {
      IDENTIFIER id-alg-slh-dsa-shake-192f
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-shake-256s PUBLIC-KEY ::= {
      IDENTIFIER id-alg-slh-dsa-shake-256s
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   pk-slh-dsa-shake-256f PUBLIC-KEY ::= {
      IDENTIFIER id-alg-slh-dsa-shake-256f
      -- KEY no ASN.1 wrapping --
      CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign }
      -- PRIVATE-KEY no ASN.1 wrapping -- }

   SLH-DSA-PublicKey ::= OCTET STRING
~~~

Section 9.1 of {{FIPS205}} defines the raw octet string encoding of an SLH-DSA
public key as the concatenation of the PK.seed and PK.root values. The octet
string length is 2*n bytes, where n is 16, 24, or 32, depending on the parameter
set. When used in a SubjectPublicKeyInfo type, the subjectPublicKey BIT STRING
contains the raw octet string encodings of the public keys.

{{!I-D.draft-ietf-lamps-cms-sphincs-plus}} defines the SLH-DSA-PublicKey ASN.1 OCTET STRING type for encoding a public key
when not used in a SubjectPublicKeyInfo. The OCTET STRING is mapped to a
subjectPublicKey (a value of type BIT STRING) as follows: the most significant
bit of the OCTET STRING value becomes the most significant bit of the BIT
STRING value, and so on; the least significant bit of the OCTET STRING
becomes the least significant bit of the BIT STRING.

The id-alg-slh-dsa-* identifiers defined in {{sec-alg-ids}} MUST be used as the algorithm field in the SubjectPublicKeyInfo sequence {{!RFC5280}} to identify a SLH-DSA public key.

The following is an example of a \[TODO: pick an OID] public key encoded using the textual encoding defined in {{?RFC7468}}.

~~~
-----BEGIN PUBLIC KEY-----
TODO
-----END PUBLIC KEY-----
~~~

Conforming CA implementations MUST specify the X.509 public key algorithm explicitly by using the OIDs specified in {{sec-alg-ids}} when using SLH-DSA public keys in certificates and CRLs.  Conforming client implementations that process SLH-DSA public keys when processing certificates and CRLs MUST recognize the corresponding OIDs.

# Key Usage Bits

The intended application for the key is indicated in the keyUsage certificate extension; see Section 4.2.1.3 of {{!RFC5280}}.  If the keyUsage extension is present in a certificate that indicates an id-alg-slh-dsa-* identifier in the SubjectPublicKeyInfo, then the at least one of following MUST be present:

~~~
    digitalSignature; or
    nonRepudiation; or
    keyCertSign; or
    cRLSign.
~~~

Requirements about the keyUsage extension bits defined in {{!RFC5280}} still apply.

# Security Considerations

The security considerations of {{RFC5280}} applies accordingly.

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

When computing signatures, the same hash function SHOULD be used to
compute the message digest of the content and the signed attributes,
if they are present.

When computing signatures, implementations SHOULD include protections
against fault injection attacks [CMP2018],[SLotH].  Protections against these
attacks include signature verification prior to releasing the
signature value to confirm that no error injected and generating the
signature a few times to confirm that the same signature value is
produced each time.

# IANA Considerations

For the ASN.1 Module in the Appendix of this document, IANA is
requested to assign an object identifier (OID) for the module
identifier (TBD1) with a Description of "id-mod-x509-slh-dsa-2024". The
OID for the module should be allocated in the "SMI Security for PKIX
Module Identifier" registry (1.3.6.1.5.5.7.0).

--- back

# ASN.1 Module {#sec-asn1}

RFC EDITOR: Please replace TBD2 with the value assigned by IANA during the publication of [I-D.draft-ietf-lamps-cms-sphincs-plus].

~~~
<CODE BEGINS>
{::include X509-SLHDSA-2024.asn}
<CODE ENDS>
~~~

# Security Strengths

Instead of defining the strength of a quantum algorithm in a traditional manner using precise estimates of the number of bits of security, NIST has instead elected to define a collection of broad security strength categories.  Each category is defined by a comparatively easy-to-analyze reference primitive that cover a range of security strengths offered by existing NIST standards in symmetric cryptography, which NIST expects to offer significant resistance to quantum cryptanalysis.  These categories describe any attack that breaks the relevant security definition that must require computational resources comparable to or greater than those required for: Level 1 - key search on a block cipher with a 128-bit key (e.g., AES128), Level 2 - collision search on a 256-bit hash function (e.g., SHA256/ SHA3-256), Level 3 - key search on a block cipher with a 192-bit key (e.g., AES192), Level 4 - collision search on a 384-bit hash function (e.g.  SHA384/SHA3-384), Level 5 - key search on a block cipher with a 256-bit key (e.g., AES 256).

The parameter sets defined for NIST security levels 1, 3 and 5 are listed in {{tab-strengths}}, along with the resulting signature size, public key, and private key sizes in bytes.

| OID                       | NIST Level | Sig.  | Pub. Key | Priv. Key |
|---                        |---         |---    |---       |---        |
| id-alg-slh-dsa-sha2-128s  | 1          | 7856  | 32       | 64        |
| id-alg-slh-dsa-sha2-128f  | 1          | 17088 | 32       | 64        |
| id-alg-slh-dsa-sha2-192s  | 3          | 16224 | 48       | 96        |
| id-alg-slh-dsa-sha2-192f  | 3          | 35664 | 48       | 96        |
| id-alg-slh-dsa-sha2-256s  | 5          | 29792 | 64       | 128       |
| id-alg-slh-dsa-sha2-256f  | 5          | 49856 | 64       | 128       |
| id-alg-slh-dsa-shake-128s | 1          | 7856  | 32       | 64        |
| id-alg-slh-dsa-shake-128f | 1          | 17088 | 32       | 64        |
| id-alg-slh-dsa-shake-192s | 3          | 16224 | 48       | 96        |
| id-alg-slh-dsa-shake-192f | 3          | 35664 | 48       | 96        |
| id-alg-slh-dsa-shake-256s | 5          | 29792 | 64       | 128       |
| id-alg-slh-dsa-shake-256f | 5          | 49856 | 64       | 128       |
{: #tab-strengths title="SLH-DSA security strengths"}

# Acknowledgments
{:numbered="false"}

Much of the structure and text of this document is based on {{?I-D.ietf-lamps-dilithium-certificates}}. The remainder comes from {{!I-D.draft-ietf-lamps-cms-sphincs-plus}}. Thanks to those authors, and the ones they based their work on, for making our work earier. "Copying always makes things easier and less error prone" - {{?RFC8411}}.
