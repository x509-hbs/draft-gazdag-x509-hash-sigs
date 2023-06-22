---
title: "Internet X.509 Public Key Infrastructure: Algorithm Identifiers for Hash-based Signatures"
abbrev: "Hash-based Signatures for X.509"
category: info

docname: draft-gazdag-x509-hash-sigs-latest
submissiontype: IETF
v: 3
area: sec
workgroup: LAMPS - Limited Additional Mechanisms for PKIX and SMIME
keyword: Internet-Draft
venue:
  group: LAMPS
  type: Working Group
  mail: spasm@ietf.org
  github: x509-hbs/draft-gazdag-x509-hash-sigs

author:
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
    org:
    email: daniel.vangeest.ietf@gmail.com
-
    ins: S. Kousidis
    name: Stavros Kousidis
    org: BSI
    email: stavros.kousidis@bsi.bund.de

normative:
  RFC5911:
  RFC5280: #v3 cer, v2 crl
  RFC8391: #xmss
  RFC8554: #hsslms
  I-D.ietf-lamps-cms-sphincs-plus:
  SP-Sub:
    title: SPHINCS+ - Submission to the 3rd round of the NIST post-quantum project. v3.1
    author:
      -
        ins: J. Aumasson
      -
        ins: D. J. Bernstein
      -
        ins: W. Beullens
      -
        ins: C. Dobraunig
      -
        ins: M. Eichlseder
      -
        ins: S. Fluhrer
      -
        ins: S. Gazdag
      -
        ins: A. Huelsing
      -
        ins: P. Kampanakis
      -
        ins: S. Koelb
      -
        ins: T. Lange
      -
        ins: M. M. Lauridsen
      -
        ins: F. Mendel
      -
        ins: R. Niederhagen
      -
        ins: C. Rechberger
      -
        ins: J. Rijneveld
      -
        ins: P. Schwabe
      -
        ins: B. Westerbaan
    date: 2021-06-10

informative:
  RFC3279:
  RFC4086: #rng
  RFC4506: #xdr
  RFC8410:
  RFC8411:
  RFC8708: #hsslms in cms
  MCGREW:
    target: https://tubiblio.ulb.tu-darmstadt.de/id/eprint/101633
    title: State Management for Hash-Based Signatures
    author:
    -
      ins: D. McGrew
    -
      ins: P. Kampanakis
    -
      ins: S. Fluhrer
    -
      ins: S. Gazdag
    -
      ins: D. Butin
    -
      ins: J. Buchmann
    date: 2016-11-02
  NISTSP:
    target: https://doi.org/10.6028/NIST.SP.800-208
    title: Recommendation for Stateful Hash-Based Signature Schemes
    author:
    -
      ins: National Institute of Standards and Technology (NIST)
    date: 2020-10-29
  NSA:
    target: https://media.defense.gov/2022/Sep/07/2003071834/-1/-1/0/CSA_CNSA_2.0_ALGORITHMS_.PDF
    title: Commercial National Security Algorithm Suite 2.0 (CNSA 2.0) Cybersecurity Advisory (CSA)
    author:
    -
      ins: National Security Agency (NSA)
    date: 2022-09-07

--- abstract

This document specifies algorithm identifiers and ASN.1 encoding formats for the
Hash-Based Signature (HBS) schemes Hierarchical Signature System (HSS), eXtended
Merkle Signature Scheme (XMSS), and XMSS^MT, a multi-tree variant of XMSS, as
well as SPHINCS+, the latter being the only stateless scheme. This specification
applies to the Internet X.509 Public Key infrastructure (PKI) when those digital
signatures are used in Internet X.509 certificates and certificate revocation
lists.

--- middle

# Introduction

Hash-Based Signature (HBS) Schemes combine Merkle trees with One/Few Time
Signatures (OTS/FTS) in order to provide digital signature schemes that remain
secure even when quantum computers become available. Their security is well
understood and depends only on the security of the underlying hash function. As
such they can serve as an important building block for quantum computer
resistant information and communication technology.

The private key of HSS, XMSS and XMSS^MT is a finite collection of OTS keys, hence only a limited
number of messages can be signed and the private key's state must be updated and persisted after signing to prevent reuse of OTS keys. While the right selection of algorithm parameters would allow a private key to sign a virtually unbounded number of messages (e.g. 2^60), this is at the cost of a larger signature size and longer signing time. Due to the statefulness of the private key of HSS, XMSS and XMSS^MT and the limited number of signatures that can be created, these signature algorithms might not be appropriate for use in interactive protocols. However, in some use case scenarios the deployment of these signature algorithms may be appropriate. Such use cases are described and discussed later in {{use-cases-hbs-x509}}.

The private key of SPHINCS+ is a finite but very large collection of FTS keys
and hence stateless. This typically comes at the cost of larger signatures
compared to the stateful HBS variants. Thus SPHINCS+ is suitable for more
use cases if the signature sizes fit the requirements.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The parameter 'n' is the security parameter, given in bytes. In practice this
is typically aligned to the standard output length of the hash function in use,
i.e. either 16, 24, 32 or 64 bytes. The height of a single tree is typically
given by the parameter 'h'. The number of levels of trees is either called 'L'
(HSS) or 'd' (XMSS, XMSS^MT, SPHINCS+).

# Use Cases of HBS in X.509 {#use-cases-hbs-x509}

As many cryptographic algorithms that are considered to be quantum-resistant,
HBS have several pros and cons regarding their practical usage. On the positive side
they are considered to be secure against a classical as well as a quantum adversary,
and a secure instantiation of HBS may always be built as long as a cryptographically
secure hash function exists. Moreover, HBS offer small public key sizes, and, in comparison to other post-quantum signature schemes, the stateful HBS can offer relatively small signature sizes (for certain parameter sets). While key generation and signature generation may
take longer than classical alternatives, fast and minimal verification routines
can be built. The major negative aspect is the statefulness of several HBS.
Private keys always have to be handled in a secure manner, but stateful HBS
necessitate a special treatment of the private key in order to avoid
security incidents like signature forgery [MCGREW], [NISTSP]. Therefore, for stateful HBS, a secure environment MUST be used for key generation and key management.

Note that, in general, root CAs offer such a secure environment and the number of issued signatures (including signed certificates and CRLs) is often moderate due to the fact that many root CAs delegate OCSP services or the signing of end-entity certificates to other entities (such as subordinate CAs) that use stateless signature schemes. Therefore, many root CAs should be able to handle the required state management, and stateful HBS offer a viable solution.

HBS MAY also be used by subordinate CAs for issuing certificates, but special and careful consideration MUST be taken into account for proper key management. HBS MUST NOT be used for end-entity certificates.

They are also appropriate in non-interactive contexts such as code signing.
Some manufactures use common and well-established key formats like X.509
for their code signing and update mechanisms. In this case, a secure key
environment as required can usually be established. Also there are multi-party
IoT ecosystems where publicly trusted code signing certificates are useful.
Further information about the security can be found in section 8. Also see
[NSA] for a short comment e. g. on software and firmware signing.

# Public Key Algorithms

Certificates conforming to [RFC5280] can convey a public key for any public key
algorithm. The certificate indicates the algorithm through an algorithm
identifier. An algorithm identifier consists of an OID and optional parameters.

In this document, we define new OIDs for identifying the different hash-based
signature algorithms. An additional OID is defined in [RFC8708] and repeated
here for convenience. For all of the OIDs, the parameters MUST be absent.

## HSS Public Keys

The object identifier and public key algorithm identifier for HSS is defined in
[RFC8708]. The definitions are repeated here for reference.

The object identifier for an HSS public key is `id-alg-hss-lms-hashsig`:

    id-alg-hss-lms-hashsig  OBJECT IDENTIFIER ::= {
       iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1) pkcs9(9)
       smime(16) alg(3) 17 }

Note that the `id-alg-hss-lms-hashsig` algorithm identifier is also referred to
as `id-alg-mts-hashsig`. This synonym is based on the terminology used in an
early draft of the document that became [RFC8554].

The HSS public key identifier is as follows:

    pk-HSS-LMS-HashSig PUBLIC-KEY ::= {
       IDENTIFIER id-alg-hss-lms-hashsig
       KEY HSS-LMS-HashSig-PublicKey
       PARAMS ARE absent
       CERT-KEY-USAGE
          { digitalSignature, nonRepudiation, keyCertSign, cRLSign } }

The HSS public key is defined as follows:

    HSS-LMS-HashSig-PublicKey ::= OCTET STRING

See [RFC8554] for more information on the contents and format of an HSS public
key. Note that the single-tree signature scheme LMS is instantiated as HSS with
level L=1.

##  XMSS Public Keys

The object identifier for an XMSS public key is `id-alg-xmss-hashsig`:

    id-alg-xmss-hashsig  OBJECT IDENTIFIER ::= {
       itu-t(0) identified-organization(4) etsi(0) reserved(127)
       etsi-identified-organization(0) isara(15) algorithms(1)
       asymmetric(1) xmss(13) 0 }

The XMSS public key identifier is as follows:

    pk-XMSS-HashSig PUBLIC-KEY ::= {
       IDENTIFIER id-alg-xmss-hashsig
       KEY XMSS-HashSig-PublicKey
       PARAMS ARE absent
       CERT-KEY-USAGE
          { digitalSignature, nonRepudiation, keyCertSign, cRLSign } }

The XMSS public key is defined as follows:

    XMSS-HashSig-PublicKey ::= OCTET STRING

See [RFC8391] for more information on the contents and format of an XMSS public
key.

## XMSS^MT Public Keys

The object identifier for an XMSS^MT public key is `id-alg-xmssmt-hashsig`:

    id-alg-xmssmt-hashsig  OBJECT IDENTIFIER ::= {
       itu-t(0) identified-organization(4) etsi(0) reserved(127)
       etsi-identified-organization(0) isara(15) algorithms(1)
       asymmetric(1) xmssmt(14) 0 }

The XMSS^MT public key identifier is as follows:

    pk-XMSSMT-HashSig PUBLIC-KEY ::= {
       IDENTIFIER id-alg-xmssmt-hashsig
       KEY XMSSMT-HashSig-PublicKey
       PARAMS ARE absent
       CERT-KEY-USAGE
          { digitalSignature, nonRepudiation, keyCertSign, cRLSign } }

The XMSS^MT public key is defined as follows:

    XMSSMT-HashSig-PublicKey ::= OCTET STRING

See [RFC8391] for more information on the contents and format of an XMSS^MT
public key.

## SPHINCS+ Public Keys

The object and public key algorithm identifiers for SPHINCS+ are defined in
[I-D.ietf-lamps-cms-sphincs-plus]. The definitions are repeated here for
reference.

    id-alg-sphincs-plus-128 OBJECT IDENTIFIER ::= {
       TBD }

    id-alg-sphincs-plus-192 OBJECT IDENTIFIER ::= {
       TBD }

    id-alg-sphincs-plus-256 OBJECT IDENTIFIER ::= {
       TBD }

The SPHINCS+ public key identifier is as follows:

    pk-sphincs-plus-128 PUBLIC-KEY ::= {
       IDENTIFIER id-alg-sphincs-plus-128
       KEY SPHINCS-Plus-PublicKey
       PARAMS ARE absent
       CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign } }

    pk-sphincs-plus-192 PUBLIC-KEY ::= {
       IDENTIFIER id-alg-sphincs-plus-192
       KEY SPHINCS-Plus-PublicKey
       PARAMS ARE absent
       CERT-KEY-USAGE
         { digitalSignature, nonRepudiation, keyCertSign, cRLSign } }

    pk-sphincs-plus-256 PUBLIC-KEY ::= {
       IDENTIFIER id-alg-sphincs-plus-256
       KEY SPHINCS-Plus-PublicKey
       PARAMS ARE absent
       CERT-KEY-USAGE
          { digitalSignature, nonRepudiation, keyCertSign, cRLSign } }

The SPHINCS+ public key is defined as follows:

    SPHINCS-Plus-PublicKey ::= OCTET STRING

See [SP-Sub] for more information on the contents and format of a SPHINCS+
public key.

# Key Usage Bits

The intended application for the key is indicated in the keyUsage certificate
extension.

If the keyUsage extension is present in an end-entity certificate that
indicates id-alg-xmss-hashsig or id-alg-xmssmt-hashsig in SubjectPublicKeyInfo,
then the keyUsage extension MUST contain one or both of the following values:

    nonRepudiation; and
    digitalSignature.

If the keyUsage extension is present in a certification authority certificate
that indicates id-alg-xmss-hashsig or id-alg-xmssmt-hashsig, then the keyUsage
extension MUST contain one or more of the following values:

    nonRepudiation;
    digitalSignature;
    keyCertSign; and
    cRLSign.

[RFC8708] defines the key usage for id-alg-hss-lms-hashsig, which is the same
as for the keys above.

# Signature Algorithms

This section identifies OIDs for signing using HSS, XMSS, XMSS^MT, and
SPHINCS+. When these algorithm identifiers appear in the algorithm field as an
AlgorithmIdentifier, the encoding MUST omit the parameters field. That is, the
AlgorithmIdentifier SHALL be a SEQUENCE of one component, one of the OIDs
defined below.

The data to be signed is prepared for signing. For the algorithms used in this
document, the data is signed directly by the signature algorithm, the data is
not hashed before processing. Then, a private key operation is performed to
generate the signature value. For HSS, the signature value is described in
section 6.4 of [RFC8554]. For XMSS and XMSS^MT the signature values are
described in sections B.2 and C.2 of [RFC8391], respectively. For SPHINCS+ the
signature values are described in [SP-Sub]. The octet string representing the
signature is encoded directly in the BIT STRING without adding any additional
ASN.1 wrapping. For the Certificate and CertificateList structures, the
signature value is wrapped in the "signatureValue" BIT STRING field.

## HSS Signature Algorithm

The HSS public key OID is also used to specify that an HSS signature was
generated on the full message, i.e. the message was not hashed before being
processed by the HSS signature algorithm.

    id-alg-hss-lms-hashsig OBJECT IDENTIFIER ::= {
       iso(1) member-body(2) us(840) rsadsi(113549) pkcs(1) pkcs9(9)
       smime(16) alg(3) 17 }

The HSS signature is defined as follows:

    HSS-LMS-HashSig-Signature ::= OCTET STRING

See [RFC8554] for more information on the contents and format of an HSS
signature.

## XMSS Signature Algorithm

The XMSS public key OID is also used to specify that an XMSS signature was
generated on the full message, i.e. the message was not hashed before being
processed by the XMSS signature algorithm.

    id-alg-xmss-hashsig  OBJECT IDENTIFIER ::= {
       itu-t(0) identified-organization(4) etsi(0) reserved(127)
       etsi-identified-organization(0) isara(15) algorithms(1)
       asymmetric(1) xmss(13) 0 }

The XMSS signature is defined as follows:

    XMSS-HashSig-Signature ::= OCTET STRING

See [RFC8391] for more information on the contents and format of an XMSS
signature.

## XMSS^MT Signature Algorithm

The XMSS^MT public key OID is also used to specify that an XMSS^MT signature
was generated on the full message, i.e. the message was not hashed before being
processed by the XMSS^MT signature algorithm.

    id-alg-xmssmt-hashsig  OBJECT IDENTIFIER ::= {
       itu-t(0) identified-organization(4) etsi(0) reserved(127)
       etsi-identified-organization(0) isara(15) algorithms(1)
       asymmetric(1) xmssmt(14) 0 }

The XMSS^MT signature is defined as follows:

    XMSSMT-HashSig-Signature ::= OCTET STRING

See [RFC8391] for more information on the contents and format of an XMSS^MT
signature.

## SPHINCS+ Signature Algorithm

The SPHINCS+ public key OID is also used to specify that a SPHINCS+ signature
was generated on the full message, i.e. the message was not hashed before being
processed by the SPHINCS+ signature algorithm.

    id-alg-sphincs-plus-128 OBJECT IDENTIFIER ::= {
       TBD }

    id-alg-sphincs-plus-192 OBJECT IDENTIFIER ::= {
       TBD }

    id-alg-sphincs-plus-256 OBJECT IDENTIFIER ::= {
       TBD }

The SPHINCS+ signature is defined as follows:

    SPHINCS-Plus-Signature ::= OCTET STRING

See [SP-Sub] for more information on the contents and format of a SPHINCS+
signature.

# ASN.1 Module

For reference purposes, the ASN.1 syntax is presented as an ASN.1 module here.
This ASN.1 Module builds upon the conventions established in [RFC5911].

    --
    -- ASN.1 Module
    --

    <CODE STARTS>

    Hashsigs-pkix-0 -- TBD - IANA assigned module OID

    DEFINITIONS IMPLICIT TAGS ::= BEGIN

    EXPORTS ALL;

    IMPORTS
      PUBLIC-KEY, SIGNATURE-ALGORITHM
      FROM AlgorithmInformation-2009
        {iso(1) identified-organization(3) dod(6) internet(1) security(5)
        mechanisms(5) pkix(7) id-mod(0) id-mod-algorithmInformation-02(58)} ;

    --
    -- Object Identifiers
    --

    -- id-alg-hss-lms-hashsig is defined in [RFC8708]

    id-alg-hss-lms-hashsig OBJECT IDENTIFIER ::= { iso(1)
       member-body(2) us(840) rsadsi(113549) pkcs(1) pkcs9(9)
       smime(16) alg(3) 17 }

    id-alg-xmss-hashsig  OBJECT IDENTIFIER ::= { itu-t(0)
       identified-organization(4) etsi(0) reserved(127)
       etsi-identified-organization(0) isara(15) algorithms(1)
       asymmetric(1) xmss(13) 0 }

    id-alg-xmssmt-hashsig  OBJECT IDENTIFIER ::= { itu-t(0)
       identified-organization(4) etsi(0) reserved(127)
       etsi-identified-organization(0) isara(15) algorithms(1)
       asymmetric(1) xmssmt(14) 0 }

    id-alg-sphincs-plus-128 OBJECT IDENTIFIER ::= {
       TBD }

    id-alg-sphincs-plus-192 OBJECT IDENTIFIER ::= {
       TBD }

    id-alg-sphincs-plus-256 OBJECT IDENTIFIER ::= {
       TBD }

    --
    -- Signature Algorithms and Public Keys
    --

    -- sa-HSS-LMS-HashSig is defined in [RFC8708]

    sa-HSS-LMS-HashSig SIGNATURE-ALGORITHM ::= {
       IDENTIFIER id-alg-hss-lms-hashsig
       PARAMS ARE absent
       PUBLIC-KEYS { pk-HSS-LMS-HashSig }
       SMIME-CAPS { IDENTIFIED BY id-alg-hss-lms-hashsig } }

    sa-XMSS-HashSig SIGNATURE-ALGORITHM ::= {
       IDENTIFIER id-alg-xmss-hashsig
       PARAMS ARE absent
       PUBLIC-KEYS { pk-XMSS-HashSig }
       SMIME-CAPS { IDENTIFIED BY id-alg-xmss-hashsig } }

    sa-XMSSMT-HashSig SIGNATURE-ALGORITHM ::= {
       IDENTIFIER id-alg-xmssmt-hashsig
       PARAMS ARE absent
       PUBLIC-KEYS { pk-XMSSMT }
       SMIME-CAPS { IDENTIFIED BY id-alg-xmssmt-hashsig } }

    -- sa-sphincs-plus-128 is defined in [I-D.ietf-lamps-cms-sphincs-plus]

    sa-sphincs-plus-128 SIGNATURE-ALGORITHM ::= {
       IDENTIFIER id-alg-sphincs-plus-128
       PARAMS ARE absent
       PUBLIC-KEYS { pk-sphincs-plus-128 }
       SMIME-CAPS { IDENTIFIED BY id-alg-sphincs-plus-128 } }

    -- sa-sphincs-plus-192 is defined in [I-D.ietf-lamps-cms-sphincs-plus]

    sa-sphincs-plus-192 SIGNATURE-ALGORITHM ::= {
       IDENTIFIER id-alg-sphincs-plus-192
       PARAMS ARE absent
       PUBLIC-KEYS { pk-sphincs-plus-192 }
       SMIME-CAPS { IDENTIFIED BY id-alg-sphincs-plus-192 } }

    -- sa-sphincs-plus-256 is defined in [I-D.ietf-lamps-cms-sphincs-plus]

    sa-sphincs-plus-256 SIGNATURE-ALGORITHM ::= {
       IDENTIFIER id-alg-sphincs-plus-256
       PARAMS ARE absent
       PUBLIC-KEYS { pk-sphincs-plus-256 }
       SMIME-CAPS { IDENTIFIED BY id-alg-sphincs-plus-256 } }

    -- pk-HSS-LMS-HashSig is defined in [RFC8708]

    pk-HSS-LMS-HashSig PUBLIC-KEY ::= {
       IDENTIFIER id-alg-hss-lms-hashsig
       KEY HSS-LMS-HashSig-PublicKey
       PARAMS ARE absent
       CERT-KEY-USAGE
          { digitalSignature, nonRepudiation, keyCertSign, cRLSign } }

    HSS-LMS-HashSig-PublicKey ::= OCTET STRING

    pk-XMSS-HashSig PUBLIC-KEY ::= {
       IDENTIFIER id-alg-xmss-hashsig
       KEY XMSS-HashSig-PublicKey
       PARAMS ARE absent
       CERT-KEY-USAGE
          { digitalSignature, nonRepudiation, keyCertSign, cRLSign } }

    XMSS-HashSig-PublicKey ::= OCTET STRING

    pk-XMSSMT-HashSig PUBLIC-KEY ::= {
       IDENTIFIER id-alg-xmssmt-hashsig
       KEY XMSSMT-PublicKey
       PARAMS ARE absent
       CERT-KEY-USAGE
          { digitalSignature, nonRepudiation, keyCertSign, cRLSign } }

    XMSSMT-HashSig-PublicKey ::= OCTET STRING

    -- pk-sphincs-plus-128 is defined in [I-D.ietf-lamps-cms-sphincs-plus]

    pk-sphincs-plus-128 PUBLIC-KEY ::= {
       IDENTIFIER id-alg-sphincs-plus-128
       KEY SPHINCS-Plus-PublicKey
       PARAMS ARE absent
       CERT-KEY-USAGE
          { digitalSignature, nonRepudiation, keyCertSign, cRLSign } }

    -- pk-sphincs-plus-192 is defined in [I-D.ietf-lamps-cms-sphincs-plus]

    pk-sphincs-plus-192 PUBLIC-KEY ::= {
       IDENTIFIER id-alg-sphincs-plus-192
       KEY SPHINCS-Plus-PublicKey
       PARAMS ARE absent
       CERT-KEY-USAGE
          { digitalSignature, nonRepudiation, keyCertSign, cRLSign } }

    -- pk-sphincs-plus-256 is defined in [I-D.ietf-lamps-cms-sphincs-plus]

    pk-sphincs-plus-256 PUBLIC-KEY ::= {
       IDENTIFIER id-alg-sphincs-plus-256
       KEY SPHINCS-Plus-PublicKey
       PARAMS ARE absent
       CERT-KEY-USAGE
          { digitalSignature, nonRepudiation, keyCertSign, cRLSign } }

    SPHINCS-Plus-PublicKey ::= OCTET STRING

    END

    <CODE ENDS>

# Security Considerations

## Algorithm Security Considerations

The cryptographic security of the signatures generated by the algorithms
mentioned in this document depends only on the hash algorithms used within the
signature algorithms and the pre-hash algorithm used to create an X.509
certificate's message digest. Grover's algorithm [Grover96] is a quantum search
algorithm which gives a quadratic improvement in search time to brute-force
pre-image attacks. The results of [BBBV97] show that this improvement is
optimal, however [Fluhrer17] notes that Grover's algorithm doesn't parallelize
well. Thus, given a bounded amount of time to perform the attack and using a
conservative estimate of the performance of a real quantum computer, the
pre-image quantum security of SHA-256 is closer to 190 bits. All parameter
sets for the signature algorithms in this document currently use SHA-256
internally and thus have at least 128 bits of quantum pre-image resistance, or
190 bits using the security assumptions in [Fluhrer17].

[Zhandry15] shows that hash collisions can be found using an algorithm with a
lower bound on the number of oracle queries on the order of 2^(n/3) on the
number of bits, however [DJB09] demonstrates that the quantum memory
requirements would be much greater. Therefore a parameter set using SHA-256
would have at least 128 bits of quantum collision-resistance as well as the
pre-image resistance mentioned in the previous paragraph.

Given the quantum collision and pre-image resistance of SHA-256 estimated
above, the current parameter sets used by `id-alg-hss-lms-hashsig`,
`id-alg-xmss-hashsig` and `id-alg-xmssmt-hashsig` provide 128 bits or more of
quantum security. This is believed to be secure enough to protect X.509
certificates for well beyond any reasonable certificate lifetime.

## Implementation Security Considerations

Implementations MUST protect the private keys. Compromise of the private keys
may result in the ability to forge signatures. Along with the private key, the
implementation MUST keep track of which leaf nodes in the tree have been used.
Loss of integrity of this tracking data can cause a one-time key to be used
more than once. As a result, when a private key and the tracking data are
stored on non- volatile media or stored in a virtual machine environment, care
must be taken to preserve confidentiality and integrity.

The generation of private keys relies on random numbers. The use of inadequate
pseudo-random number generators (PRNGs) to generate these values can result in
little or no security. An attacker may find it much easier to reproduce the
PRNG environment that produced the keys, searching the resulting small set of
possibilities, rather than brute force searching the whole key space. The
generation of quality random numbers is difficult. [RFC4086] offers important
guidance in this area.

The generation of hash-based signatures also depends on random numbers. While
the consequences of an inadequate pseudo-random number generator (PRNG) to
generate these values is much less severe than the generation of private keys,
the guidance in [RFC4086] remains important.

# IANA Considerations

IANA is requested to assign a module OID from the "SMI for PKIX Module
Identifier" registry for the ASN.1 module in Section 6.

--- back

# Acknowledgments
{:numbered="false"}

Thanks for Russ Housley and Panos Kampanakis for helpful suggestions.

This document uses a lot of text from similar documents ([RFC3279] and
[RFC8410]) as well as [RFC8708]. Thanks go to the authors of those documents.
"Copying always makes things easier and less error prone" - [RFC8411].
