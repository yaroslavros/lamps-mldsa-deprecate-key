---
title: "Deprecating Expanded ML-DSA and ML-KEM Private Key Encodings"
abbrev: "Deprecating Expanded ML-DSA and ML-KEM Private Key Encodings"
category: std
updates: 9881, 9935

docname: draft-rosomakho-lamps-mldsa-mlkem-seed-only-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Limited Additional Mechanisms for PKIX and SMIME"
keyword:
 - mldsa
 - milked
 - expanded
 - deprecate
venue:
  group: "Limited Additional Mechanisms for PKIX and SMIME"
  type: "Working Group"
  mail: "spasm@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/spasm/"
  github: "yaroslavros/lamps-mldsa-deprecate-key"
  latest: "https://yaroslavros.github.io/lamps-mldsa-deprecate-key/draft-rosomakho-lamps-mldsa-mlkem-seed-only.html"

author:
 -
    fullname: Yaroslav Rosomakho
    organization: Zscaler
    email: yrosomakho@zscaler.com

 -
    fullname: David Benjamin
    organization: Google LLC
    email: davidben@google.com

normative:

informative:

...

--- abstract

This document updates RFC 9881 and RFC 9935 to require the seed representation
when encoding Module-Lattice-Based Digital Signature Algorithm (ML-DSA) and
Module-Lattice-Based Key-Encapsulation Mechanism (ML-KEM) private
keys for interchange. It deprecates the expanded and combined
representations, while permitting their continued support for legacy
imports. These changes simplify implementation and improve
interoperability by aligning the private-key representation with that
used by JSON Object Signing and Encryption (JOSE) and CBOR Object Signing
and Encryption (COSE). Internal private-key representations are unaffected.


--- middle

# Introduction {#intro}

{{!X509-MLDSA=RFC9881}} and {{!X509-MLKEM=RFC9935}} define, respectively, three
representations for Module-Lattice-Based Digital Signature Algorithm (ML-DSA)
Module-Lattice-Based Key-Encapsulation Mechanism (ML-KEM) private keys: a 32-octet seed, an expanded
private key, and a combination of the seed and expanded private key. The
seed deterministically yields both the expanded private key and the public
key. Although {{Section 8.1 of X509-MLDSA}} and {{Section 6 of X509-MLKEM}}
recommend the seed representation, they permit all three representations for private-key
interchange.

Multiple representations complicate interoperability between
implementations that support different subsets of these formats. An
implementation that supports only seeds cannot import an expanded private
key without its seed. The combined representation also introduces a
consistency check between the seed and expanded private key, which can
require additional implementation support solely for importing that
representation.

{{?JOSE-COSE-MLDSA=RFC9964}} and {{?JOSE-COSE-MLKEM=I-D.ietf-jose-pqc-kem}}
specify only the seed representation for ML-DSA and ML-KEM private keys
in JSON Object Signing and Encryption (JOSE) and CBOR Object Signing and
Encryption (COSE). Using the seed representation consistently across these
specifications simplifies private-key interchange and avoids the need to
support multiple representations and their associated validation logic.

This document updates {{X509-MLDSA}} and {{X509-MLKEM}} to deprecate the
expanded and combined representations, limiting them to legacy compatibility
where applicable.


# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Updates to ML-DSA Private Key Encoding {#updates}

This section updates the private-key encoding requirements and format
recommendations in {{Section 6 of X509-MLDSA}}, {{Section 6 of X509-MLKEM}}, and
{{Section 8.1 of X509-MLDSA}}. These updates apply to ML-DSA-44, ML-DSA-65,
ML-DSA-87, ML-KEM-512, ML-KEM-768, and ML-KEM-1024.

Implementations that decode ML-DSA or ML-KEM private keys for interchange using
the structures defined in {{X509-MLDSA}} and {{X509-MLKEM}} MUST support their
respective `seed` representations.

Implementations MAY support the `both` and `expandedKey` representations for
compatibility with legacy systems, but this is optional and not required
for conformance with this specification. If decoding the `both` representation,
implementations MUST perform the consistency check described in
{{Section 8.2 of X509-MLDSA}} and {{Section 8 of X509-MLKEM}}.

When decoding a private key in either the `seed` or `both` representation,
and when generating a new ML-DSA or ML-KEM key, implementations MUST retain the
seed, unless the private key will never be encoded for interchange.

Implementations that encode ML-DSA or ML-KEM private keys for interchange MUST
use the `seed` representation, unless the key is a legacy key whose seed has
been discarded. If (and only if) the seed was discarded, implementations MAY use
the `expandedKey` representation. However, this encoding will not be
interoperable with all implementations of this specification.

Implementations MUST NOT output the `both` representation.

# Compatibility and Migration Considerations {#migration}

Existing private keys can be re-encoded in the `seed` representation when
the original seed is available, including from the `both` representation.
This conversion does not require replacing the key pair or reissuing
certificates.

Expanded private keys can be computed from seeds, so the `seed` representation
can be imported into a legacy implementation that expects expanded private keys.
When doing so, operators SHOULD ensure that seeds are retained for keys that may
need to be exported or restored into another implementation.

As noted in {{Section 8.1 of X509-MLDSA}} and {{Section 7 of X509-MLKEM}}, a discarded seed cannot be
recovered from the expanded private key. If no copy of the seed is
available, migration to an implementation that accepts only seeds requires
generating a new key pair and replacing any associated certificates or
other public-key bindings.

However, such existing keys MAY continue to be used by implementations that
support this representation. This document still optionally permits the
`expandedKey` encoding for such cases. It also places no constraints on keys
that are never exported for interchange.


# Security Considerations {#security}

The security considerations of {{X509-MLDSA}} and {{X509-MLKEM}} continue to apply. The seed
is sufficient to reconstruct the private key, so
it requires the same protection as the expanded private key, including
when retained in backups or transferred between implementations.


# IANA Considerations {#iana}

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
