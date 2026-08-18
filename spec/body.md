## DTG Credential Taxonomy

*This section is informative.*

This section provides a visual overview of the DTG Core Credential types and their formal type hierarchy. The three functional categories (edge, invitation, annotation) are descriptive aids only; they do not appear in credential schemas.

```mermaid
graph LR
    DTG[DTGCredential]

    DTG --> EC(Edge Credentials)
    DTG --> IC(Invitation Credentials)
    DTG --> AC(Annotation Credentials)

    EC --> VRC["VRC - RelationshipCredential"]
    EC --> VMC["VMC - MembershipCredential"]
    EC --> VDC["VDC - DelegationCredential"]
    IC --> VIC["VIC - InvitationCredential"]
    AC --> VPC["VPC - PersonaCredential"]
    AC --> VWC["VWC - WitnessCredential"]
    AC --> VEC["VEC - EndorsementCredential"]

    classDef parent fill:#f5f5f5,stroke:#555,stroke-width:2px,color:#000
    classDef cat fill:#eeeeee,stroke:#999,stroke-width:1px,color:#555
    classDef edge fill:#bbdefb,stroke:#1976d2,stroke-width:2px,color:#000
    classDef inv fill:#ffe0b2,stroke:#f57c00,stroke-width:2px,color:#000
    classDef ann fill:#e1bee7,stroke:#7b1fa2,stroke-width:2px,color:#000

    class DTG parent
    class EC,IC,AC cat
    class VMC,VRC,VDC edge
    class VIC inv
    class VPC,VEC,VWC ann
```

### Formal W3C Type Hierarchy

```text
VerifiableCredential
└── DTGCredential
    ├── MembershipCredential (VMC)
    ├── RelationshipCredential (VRC)
    ├── DelegationCredential (VDC)
    ├── InvitationCredential (VIC)
    ├── PersonaCredential (VPC)
    ├── EndorsementCredential (VEC)
    └── WitnessCredential (VWC)
```

> **Note:** The [[ref: r-card]] (relationship card) that appeared in earlier drafts of this specification is a [[ref: verifiable data structure]] (VDS), not a `DTGCredential` subtype. It will be defined in the planned **DTG Verifiable Data Structures** specification (see [Related Specifications](#related-specifications)).

## W3C Verifiable Credentials Version Support

This section is normative.

### Primary Standard: v2.0

This specification is written using **W3C Verifiable Credentials Data Model v2.0** syntax. All DTG implementations MUST support v2.0 credential verification and SHOULD support v2.0 credential issuance.

### Legacy System Compatibility: v1.1

Many existing [[ref: identity verification providers]] (IDVPs), [trust registries](https://glossary.trustoverip.org/#term:trust-registry), and community infrastructure may only support W3C VC Data Model v1.1. To ensure broad interoperability and avoid forcing costly system migrations:

- DTG implementations SHOULD accept and verify v1.1 credentials
- Existing credential issuers MAY issue DTG-compliant credentials using v1.1 syntax
- New implementations SHOULD prioritize v2.0 but MAY also issue v1.1 when required by ecosystem constraints

> **Design Intent:** This dual-version support enables:
>
> - Legacy IDVPs to issue [[ref: IDVCs]] (identity verification credentials) without system upgrades
> - Existing [[ref: VTCs]] to participate in the DTG using their current infrastructure
> - Gradual ecosystem migration from v1.1 to v2.0 without breaking trust relationships

### Property Mapping

The only differences between v1.1 and v2.0 DTG credentials are:

| Property | v1.1 | v2.0 |
| ---------- | ------ | ------ |
| **Context** | `https://www.w3.org/2018/credentials/v1` | `https://www.w3.org/ns/credentials/v2` |
| **Issuance** | `issuanceDate` | `validFrom` |
| **Expiration** | `expirationDate` | `validUntil` |

All DTG-specific schemas (types, issuer requirements, credentialSubject structure) are identical.

> **Implementation Note:** Verifiers supporting both v1.1 and v2.0 credentials MUST be able to process proof types commonly used in both versions. Issuers SHOULD use well-supported proof types and include all necessary contexts.

### Dual-Version Examples

**v2.0 (Primary):**

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://firstperson.network/credentials/dtg/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "type": ["VerifiableCredential", "DTGCredential", "MembershipCredential"],
  "issuer": "did:web:chess-club.example",
  "validFrom": "2026-01-06T10:00:00Z",
  "validUntil": "2027-01-06T10:00:00Z",
  "credentialSubject": {
    "id": "did:key:z6MkpTHR8VNs..."
  },
  "proof": {
    "type": "Ed25519Signature2020",
    "created": "2026-01-06T10:00:00Z",
    "proofPurpose": "assertionMethod",
    "verificationMethod": "did:web:chess-club.example#key-1",
    "proofValue": "z3FXQjecWJKT..."
  }
}
```

**v1.1 (Legacy Compatibility):**

```json
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://firstperson.network/credentials/dtg/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "type": ["VerifiableCredential", "DTGCredential", "MembershipCredential"],
  "issuer": "did:web:chess-club.example",
  "issuanceDate": "2026-01-06T10:00:00Z",
  "expirationDate": "2027-01-06T10:00:00Z",
  "credentialSubject": {
    "id": "did:key:z6MkpTHR8VNs..."
  },
  "proof": {
    "type": "Ed25519Signature2020",
    "created": "2026-01-06T10:00:00Z",
    "proofPurpose": "assertionMethod",
    "verificationMethod": "did:web:chess-club.example#key-1",
    "proofValue": "z3FXQjecWJKT..."
  }
}
```

> **Note:** All examples in this specification use v2.0 syntax unless explicitly labeled otherwise. When implementing v1.1 support, use the property mappings above.

## Base Structure

This section is normative.

All DTG credentials share this W3C VC structure (v2.0 shown; see [Legacy System Compatibility](#legacy-system-compatibility-v11) for v1.1 compatibility):

**Schema:**

- `@context` (array, REQUIRED): MUST include `"https://www.w3.org/ns/credentials/v2"` and `"https://firstperson.network/credentials/dtg/v1"`, plus any additional contexts required by the proof type
- `type` (array, REQUIRED): MUST include `"VerifiableCredential"`, `"DTGCredential"`, and exactly one concrete subtype
- `issuer` (string, REQUIRED): DID of the issuing entity ([[ref: C-DID]], [[ref: M-DID]], [[ref: R-DID]], or [[ref: P-DID]] as appropriate)
- `validFrom` (string, REQUIRED): ISO 8601 datetime (`issuanceDate` in v1.1)
- `validUntil` (string, OPTIONAL): ISO 8601 datetime (`expirationDate` in v1.1)
- `credentialSubject` (object, REQUIRED):
  - `id` (string, REQUIRED): DID of the subject
  - Additional type-specific properties
- `taskContext` (string, OPTIONAL unless a credential type requires it): identifier (`threadId`) of the [trust task](https://glossary.trustoverip.org/#term:trust-tasks) exchange in which this credential was issued. See [Trust Task Context Binding](#trust-task-context-binding).
- `proof` (object, REQUIRED): W3C VC proof object

**Example:**

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://firstperson.network/credentials/dtg/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "type": ["VerifiableCredential", "DTGCredential", "MembershipCredential"],
  "issuer": "did:example:vtcCommunityDid",
  "validFrom": "2026-01-06T10:00:00Z",
  "validUntil": "2027-01-06T10:00:00Z",
  "credentialSubject": {
    "id": "did:example:memberMdid"
  },
  "proof": {
    "type": "Ed25519Signature2020",
    "created": "2026-01-06T10:00:00Z",
    "proofPurpose": "assertionMethod",
    "verificationMethod": "did:example:vtcCommunityDid#key-1",
    "proofValue": "z3FXQjecWJKT..."
  }
}
```

## Edge Credentials

This section is normative.

Edge credentials establish relationships between existing entities (nodes) in the DTG: [[ref: VRCs]] attest to relationships between two entities, [[ref: VMCs]] attest to community membership, and [[ref: VDCs]] attest that one entity has appointed another to act in its name. In each case, a bi-directional pair of credentials forms a complete [[ref: DTG edge]].

### VRC (Verifiable Relationship Credential)

**Purpose:** Attests to a relationship between two entities; two VRCs (one each direction) form a complete [[ref: DTG edge]].

**Schema:**

- `type` (array, REQUIRED): MUST include `"RelationshipCredential"`
- `issuer` (string, REQUIRED): [[ref: R-DID]] or [[ref: M-DID]] of the source party
- `credentialSubject` (object, REQUIRED):
  - `id` (string, REQUIRED): R-DID or M-DID of the target party

**Example:**

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://firstperson.network/credentials/dtg/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "type": ["VerifiableCredential", "DTGCredential", "RelationshipCredential"],
  "issuer": "did:peer:2.Ez6LSbysKZ...",
  "validFrom": "2026-01-06T10:00:00Z",
  "credentialSubject": {
    "id": "did:peer:2.Ez6LSpSrLxn..."
  },
  "proof": { "//": "..." }
}
```

**Note:** R-DIDs are RECOMMENDED for privacy; M-DIDs are allowed for bootstrapping (see [Privacy Considerations](#privacy-considerations)).

#### Unilateral Relationship Identification

A [[ref: relationship DID]] (R-DID) generated by a controller for the explicit purpose of establishing a VRC serves as a globally unique identifier for that relationship edge from the perspective of the controller.

Therefore, a relationship within the DTG can be canonically identified by two independent identifiers:

- The Source R-DID (controlled by the Issuer)
- The Target R-DID (controlled by the Subject)

Semantic statements, metadata, or private context regarding the relationship MAY be anchored solely to the controller's own R-DID, without requiring the resolution or inclusion of the counterparty's identifier.

> **IMPORTANT**: The valid application of this specification requires that each entity MUST generate a new, unique R-DID for every single entity they connect with, even within the same community.

#### Pairwise Zero-Knowledge Proof

The holder of a VRC MAY construct a zero-knowledge proof that demonstrates possession of a valid VRC and selectively discloses chosen attributes, subject DIDs, or predicates over them. A common application is to disclose the parties' [[ref: P-DIDs]] (persona DIDs) while hiding the underlying [[ref: R-DIDs]] (relationship DIDs), enabling a public, verifiable claim that two known [[ref: personas]] have a relationship without exposing the private pairwise channel between them or enabling correlation across the holder's other presentations. This construction is available to any two parties who hold a VRC between them, regardless of whether they share membership in a [[ref: VTC]]. It supports selective disclosure and minimal correlation across contexts. It does not by itself confer any community-level assurance (e.g., personhood); whatever assurance it carries derives from the parties' own out-of-band context, the public reputation attached to any disclosed persona DIDs, and the cryptographic integrity of the VRC.

### VMC (Verifiable Membership Credential)

**Purpose:** Attests to the membership of an entity in a [[ref: VTC]] or [[ref: VTN]]; two VMCs (one each direction) form a complete [[ref: DTG edge]].

**Schema:**

- `type` (array, REQUIRED): MUST include `"MembershipCredential"`
- `issuer` (string, REQUIRED): [[ref: C-DID]] of the VTC or VTN
- `credentialSubject` (object, REQUIRED):
  - `id` (string, REQUIRED): [[ref: M-DID]] of the member (person/device/agent) OR C-DID (for VTN-to-VTC membership)

**Example:**

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://firstperson.network/credentials/dtg/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "type": ["VerifiableCredential", "DTGCredential", "MembershipCredential"],
  "issuer": "did:web:chess-club.example",
  "validFrom": "2026-01-06T10:00:00Z",
  "credentialSubject": {
    "id": "did:key:z6MkpTHR8VNs..."
  },
  "proof": { "//": "..." }
}
```

#### Community-Anchored Zero-Knowledge Proof

A VRC is a signed verifiable credential. It MAY be presented and verified using standard W3C VC presentation methods when privacy preservation is not required, and it SHOULD be presented using a zero-knowledge proof whenever privacy preservation is desired. Community membership is **not** a precondition for issuing, holding, or presenting a VRC; two entities that do not share (or do not hold) a [[ref: VMC]] can still exchange VRCs, and the resulting edges are valid trust attestations standing on their cryptographic signatures and on whatever real-world context the parties bring to them.

When both parties to a VRC hold VMCs from the same community, the holder MAY construct a community-anchored ZKP of the relationship. In such a proof, the holder demonstrates:

1. Possession of the VRC
2. Possession of the underlying VMC (proving membership in the community)
3. The VRC issuer possesses a VMC from the *same* [[ref: C-DID]] (community DID)

This allows the relationship's existence to be proven within a shared community's governance context without revealing the specific DIDs or other credential details. Whatever assurances the community's trust registry attaches to its VMCs (e.g., personhood, when the VMCs qualify as [[ref: PHCs]]) carry forward into the proof.

This is one proof construction available to relationships within a shared community. Detailed ZK protocols and registry-ZK interactions are out of scope for this specification (see [Zero-Knowledge and Selective Disclosure](#zero-knowledge-and-selective-disclosure)).

**Note:** Implementations SHOULD make ZKP presentation the default behavior so that users obtain privacy preservation without having to opt in. See [Privacy Considerations](#privacy-considerations).

### VDC (Verifiable Delegation Credential)

**Purpose:** Attests that one entity (the delegator) has appointed another entity (the delegate) to act in the delegator's name, for a bounded set of acts, for a limited period, revocably. Within the appointed `scope`, what the delegate does is attributable to the delegator. Two VDCs — a grant and a matching acceptance — form a complete [[ref: DTG edge]].

A VDC differs from every other credential in this specification in kind, not only in payload. The [[ref: VRC]], [[ref: VMC]], [[ref: VIC]], [[ref: VPC]], [[ref: VEC]], and [[ref: VWC]] all *attest* that something is true about the graph; a verifier evaluates each as a claim and asks whether it is true. A VDC establishes *representation*; a verifier asks a different question — whether this party may stand in for that one, for this act, at this moment — and answering it requires steps that evaluating a claim does not: scope containment, chain resolution, invocation binding, and timely revocation. That is why delegation is defined as its own concrete subtype rather than expressed through the payload of an existing type.

#### Grant and Invocation

*This subsection is informative.*

Delegation divides cleanly along the test in [Credentials versus Trust Task Artifacts](#credentials-versus-trust-task-artifacts):

- **The grant** — "this delegator has appointed this delegate to act in its name, for this scope, until this time" — is true standing alone and outlives the exchange in which it was issued. It is a **credential**, defined here.
- **The invocation** — "the delegate is acting in the delegator's name, now, to do this particular thing" — is meaningful only inside the exchange in which it occurs. It is a **trust task artifact**, correlated by `threadId` and defined in the planned DTG Core Trust Task Protocols specification.

This specification therefore defines what a delegation *is* and how a verifier establishes that it is valid and in force. It does not define how a delegation is exercised.

#### Delegation and Authority

*This subsection is informative.*

Delegation and authority are distinct, and a VDC expresses only delegation. Keeping the two apart is what tells an implementer which credential to reach for.

- **Authority** answers *may this party do this thing?* The party acts **as itself**. What it does is attributed to it, and it may do it because it has been permitted to.
- **Delegation** answers *may this party act in another's name?* The delegate acts **as the delegator**. What it does within `scope` is attributed to the delegator, and is bounded by what the delegator could have done itself.

| Question | Answered by | The act is attributed to |
| ---------- | ------------- | -------------------------- |
| May this party do this thing, as itself? | authority — not defined in this specification | the party itself |
| May this party act in another's name? | delegation — the VDC | the entity in whose name it acts |

Neither implies the other. A service granted access to a person's mailbox may read that mail as itself; it has not thereby been appointed to send mail in that person's name. Conversely, a delegate appointed to correspond in a person's name holds that appointment whether or not it has been given access to any particular mailbox — and where it has not, the appointment gets it nowhere. The first is authority without delegation; the second is delegation without authority. A credential that conflated them would leave a verifier unable to tell which of the two it had been shown.

**When to use a VDC.** Ask whose name the act is performed in.

- **The actor's own name** — the actor is doing something it has been permitted to do, and the act is attributed to it. This is a question of authority, and a VDC is the wrong credential. This specification does not currently define a credential for it.
- **Another entity's name** — the actor is standing in for that entity, and the act is attributed to that entity. This is delegation, and a VDC is the credential that establishes it.

#### How a Delegation Composes with Authority

*This subsection is informative.*

A VDC neither carries authority nor confers it on the delegate. When a delegate presents a VDC and asks to perform an act, the verifier does not ask what the *delegate* is permitted to do, and it does not treat the VDC as a permission the delegator has handed over. It substitutes the delegator for the delegate and then asks the question it would have asked of the delegator directly. **A VDC moves the question; it does not answer it.**

Three checks, each independent of the others:

1. **Is this the delegate, and may it act in the delegator's name for this act?** Established by the VDC, together with [Delegation Chains](#delegation-chains) and [Invocation Binding](#invocation-binding). This specification defines this check.
2. **May the delegator perform this act?** Established by whatever the act requires of the delegator — community membership, a governance framework, an [[ref: IDVC]], a permission credential, or the verifier's own policy. This specification does not define this check, and a VDC does not influence its outcome.
3. **Must the delegate independently qualify?** A governance determination. Some communities will require a delegate to hold a [[ref: VMC]] of its own, or to satisfy the same requirements as any other actor, before it may act for anyone; others will not.

The reach of a delegation is the **intersection** of what the delegator may do and what the VDC chain appoints the delegate for — never the union, and never more than either.

Three consequences follow, and they answer the question of what, exactly, has been handed over:

- **Nothing the delegator holds is copied to the delegate.** A credential issued to the delegator remains the delegator's. The issuer's assurance about the delegator does not extend to the delegate, and a delegator cannot re-issue to a delegate what it was itself issued. A delegation is a statement by the delegator about who may speak in its name; it is not a transfer of anything the delegator was given.
- **Withdrawing the delegator's own permission ends the delegate's ability to act immediately**, without revoking the VDC, because check 2 is evaluated at the time of the act rather than at the time of the appointment. Revoking the VDC and withdrawing the underlying permission are different remedies with different reach, and a delegator may need either.
- **A `scope` may exceed what the delegator itself may do.** This is not an error: a delegator's own permissions change over the life of a durable appointment. A verifier MUST NOT treat such a `scope` as conferring anything beyond what check 2 allows, and issuers SHOULD NOT issue one as a matter of hygiene.

Credentials expressing authority are out of scope here. Confining the VDC to delegation leaves the [[ref: DTGWG]] free to define one separately — a verifiable authority credential, say — without reinterpreting the VDC or contending with it for the same semantic ground.

**Schema:**

- `type` (array, REQUIRED): MUST include `"DelegationCredential"`
- `issuer` (string, REQUIRED): DID of the delegator — an [[ref: R-DID]] or [[ref: M-DID]] for a person, device, or agent, a [[ref: C-DID]] where a community delegates to a [[ref: VTA]] or other service, or a [[ref: P-DID]] where the delegation is made under a [[ref: persona]]
- `validUntil` (string, REQUIRED): ISO 8601 datetime (`expirationDate` in v1.1). Unlike the base structure, `validUntil` is REQUIRED for a VDC: an appointment with no expiry cannot be reasoned about by a verifier that cannot reach the delegator.
- `credentialStatus` (object, REQUIRED): a W3C VC status mechanism through which a verifier can determine whether the delegation has been revoked. Revocation of a delegation is the mechanism by which a delegator withdraws an appointment already made, and MUST therefore be checkable by a verifier without contacting the delegator. The status mechanism used is determined by the governing [[ref: VTC]] or [[ref: VTN]].
- `credentialSubject` (object, REQUIRED):
  - `id` (string, REQUIRED): DID of the delegate
  - `delegation` (object, REQUIRED):
    - `scope` (array of strings, REQUIRED): the acts the delegate may perform in the delegator's name. MUST contain at least one entry; the vocabulary is defined by the governing VTC or VTN, as for the `endorsement` structure of a [[ref: VEC]]. A VDC MUST NOT express an unbounded appointment by omitting or emptying `scope`.
    - `parent` (string, OPTIONAL): the digest of the VDC from which this delegation was derived, when the delegator is itself acting under a delegation. Encoded exactly as the [[ref: VWC]] `digest` property: the SHA-256 hash of the parent credential canonicalized per [JCS, RFC 8785](https://datatracker.ietf.org/doc/html/rfc8785), as the string `sha256:` followed by the lowercase hexadecimal digest. A VDC with no `parent` is a **root delegation**.
    - `maxDepth` (integer, OPTIONAL): the number of further re-delegations permitted below this one. A value of `0` prohibits re-delegation. When absent, re-delegation is prohibited.
    - `accepts` (string, OPTIONAL): present only in the acceptance direction; the digest of the grant being accepted, encoded as for `parent`. See [Delegation Edges](#delegation-edges).

**Example (grant):**

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://firstperson.network/credentials/dtg/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "type": ["VerifiableCredential", "DTGCredential", "DelegationCredential"],
  "issuer": "did:peer:2.Ez6LSbysKZ...",
  "validFrom": "2026-01-06T10:00:00Z",
  "validUntil": "2026-04-06T10:00:00Z",
  "credentialStatus": {
    "id": "https://chess-club.example/status/3#94567",
    "type": "BitstringStatusListEntry",
    "statusPurpose": "revocation",
    "statusListIndex": "94567",
    "statusListCredential": "https://chess-club.example/status/3"
  },
  "credentialSubject": {
    "id": "did:key:z6MkpTHR8VNs...",
    "delegation": {
      "scope": ["schedule:read", "schedule:propose"],
      "maxDepth": 0
    }
  },
  "proof": { "//": "..." }
}
```

**Example (acceptance):**

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://firstperson.network/credentials/dtg/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "type": ["VerifiableCredential", "DTGCredential", "DelegationCredential"],
  "issuer": "did:key:z6MkpTHR8VNs...",
  "validFrom": "2026-01-06T10:05:00Z",
  "validUntil": "2026-04-06T10:00:00Z",
  "credentialStatus": { "//": "..." },
  "credentialSubject": {
    "id": "did:peer:2.Ez6LSbysKZ...",
    "delegation": {
      "scope": ["schedule:read", "schedule:propose"],
      "accepts": "sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    }
  },
  "proof": { "//": "..." }
}
```

#### Delegation Edges

A VDC whose `delegation` object has no `accepts` property is a **grant**: the delegator states the appointment it has made. A VDC whose `delegation` object has an `accepts` property is an **acceptance**: the delegate acknowledges the appointment it has taken on, and thereby the accountability that attaches to acting in another's name. Together they form a complete [[ref: DTG edge]], consistent with the bidirectional pairing of [[ref: VRCs]] and [[ref: VMCs]].

- In an acceptance, `issuer` MUST be the `credentialSubject.id` of the grant, and `credentialSubject.id` MUST be the `issuer` of the grant.
- In an acceptance, `delegation.scope` MUST be identical to the `scope` of the grant identified by `accepts`, so that what the delegate consented to is legible without resolving the grant.
- A verifier that relies on the delegate having knowingly taken on the appointment MUST obtain and verify the acceptance. A grant alone establishes what the delegator appointed, not what the delegate agreed to.

#### Delegation Chains

A delegate MAY appoint a further delegate only where the VDC it holds permits this, and only for a subset of the acts it was itself appointed for. Where a VDC carries a `parent`, verifiers MUST evaluate the whole chain:

1. Every VDC in the chain MUST independently satisfy the verification requirements of this specification, including proof verification, validity period, and revocation status.
2. The `scope` of each VDC MUST be a subset of the `scope` of the VDC identified by its `parent`. A verifier MUST reject any chain in which a delegation broadens the appointment it derives from.
3. The `validUntil` of each VDC MUST NOT be later than the `validUntil` of its parent.
4. Each step below a VDC bearing `maxDepth` reduces the permitted remaining depth by one. A verifier MUST reject a chain deeper than the shallowest `maxDepth` along it, and MUST reject any re-delegation below a VDC that omits `maxDepth` or sets it to `0`.
5. The chain MUST terminate in a root delegation whose `issuer` is the principal — the entity in whose name the acts would ultimately be performed. A verifier MUST establish that this principal is the party it intends to deal with; a chain that cannot be resolved to such a root establishes no representation. A governing [[ref: VTC]] or [[ref: VTN]] MAY additionally restrict which entities may delegate which acts, published via the applicable [trust registry](https://glossary.trustoverip.org/#term:trust-registry).

As with the [[ref: VWC]] `digest`, a `parent` value is only as useful as the verifier's access to the credential it references. Holders presenting a derived VDC SHOULD make the full chain available alongside it.

#### Invocation Binding

A VDC is not a bearer token. A verifier MUST NOT accept a party as acting in the delegator's name unless that party demonstrates control of the verification method associated with `credentialSubject.id` at the time of the request. A VDC presented without such a demonstration is evidence that a delegation exists; it is not evidence that the party presenting it is the delegate.

#### Relationship to Capability Models

*This subsection is informative.*

Chaining, attenuation, and invocation are well-explored outside the W3C VC data model, notably in [ZCAP-LD](https://w3c-ccg.github.io/zcap-spec/) and [UCAN](https://github.com/ucan-wg/spec). The VDC reuses their mechanics — attenuation-only re-delegation, chains resolving to a recognized root, and binding to a demonstration of key control at invocation — rather than inventing a different set.

The semantics differ, and the distinction in [Delegation and Authority](#delegation-and-authority) is exactly the one at issue: those models chain *permissions*, whereas a VDC chains *representation*. The mechanics are shared because both must answer how a grant narrows as it passes down a chain and how it is bound to the party invoking it, not because the thing being passed is the same.

The VDC expresses those mechanics as a DTG credential, rather than referencing an external capability token, for two reasons. First, a delegation is a durable edge of the graph and is expected to be reasoned about alongside the other DTG edges. Second, this specification's schemas are kept minimal so that holders can satisfy predicates in zero knowledge; an opaque embedded token would place the one payload a verifier most needs to reason about — the scope — outside the reach of that machinery. Mappings between VDCs and these formats are left to future work.

## Invitation Credentials

This section is normative.

### VIC (Verifiable Invitation Credential)

**Purpose:** Authorizes a prospective member to join a [[ref: VTC]] or [[ref: VTN]] when presented to the [[ref: VTA]]/[[ref: PEP]]. The [[ref: DTG invitation credential]] has two functional variants distinguished by issuer and subject rules (not by separate type strings): the [[ref: VTC invitation credential]] and the [[ref: VTN invitation credential]].

**Schema:**

- `type` (array, REQUIRED): MUST include `"InvitationCredential"`
- `issuer` (string, REQUIRED):
  - For VTC invitation: VTC [[ref: C-DID]] OR authorized member's [[ref: M-DID]] (per policy)
  - For VTN invitation: VTN C-DID OR member VTC's C-DID (per policy)
- `credentialSubject` (object, REQUIRED):
  - `id` (string, REQUIRED):
    - For VTC invitation: prospective member's M-DID OR prospective VTC's C-DID
    - For VTN invitation: prospective VTC's C-DID

**Example (VTC member invitation):**

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://firstperson.network/credentials/dtg/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "type": ["VerifiableCredential", "DTGCredential", "InvitationCredential"],
  "issuer": "did:key:z6MkhaXgBZD...",
  "validFrom": "2026-01-06T10:00:00Z",
  "validUntil": "2026-02-06T10:00:00Z",
  "credentialSubject": {
    "id": "did:key:z6MkpTHR8VNs..."
  },
  "proof": { "//": "..." }
}
```

> **Editor's note — roles and access control:** Roles and access control policy details are primarily inferred from the issuer plus the [trust registry](https://glossary.trustoverip.org/#term:trust-registry). An open question for this Working Draft is whether any of this information should be embedded in the VIC itself.

## Annotation Credentials

This section is normative.

Annotation credentials **do not create graph structure**. They attach data to existing edges or parties.

### VPC (Verifiable Persona Credential)

**Purpose:** Links a [[ref: persona DID]] (P-DID) to an existing relationship, enabling the holder to control intentional correlation across relationships.

**Schema:**

- `type` (array, REQUIRED): MUST include `"PersonaCredential"`
- `issuer` (string, REQUIRED): [[ref: P-DID]] of the persona
- `credentialSubject` (object, REQUIRED):
  - `id` (string, REQUIRED): Counterparty's DID (typically [[ref: R-DID]] or [[ref: M-DID]] used in the relationship)

**Example:**

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://firstperson.network/credentials/dtg/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "type": ["VerifiableCredential", "DTGCredential", "PersonaCredential"],
  "issuer": "did:key:z6MkrKqT9pL...",
  "validFrom": "2026-01-06T10:00:00Z",
  "credentialSubject": {
    "id": "did:peer:2.Ez6LSpSrLxn..."
  },
  "proof": { "//": "..." }
}
```

### VWC (Verifiable Witness Credential)

**Purpose:** Third-party attestation that an edge was established under specific conditions. The witness may be a person or a [[ref: VTA]] applying the witnessing policies of a [[ref: VTC]] — for example, verifying that both parties were present at the same event, or provided proof of biometric liveness at the time of relationship formation.

Because the meaning of a witness attestation depends on the conditions under which the witnessing occurred, a VWC MUST be bound to the [trust task](https://glossary.trustoverip.org/#term:trust-tasks) exchange in which it was issued via the `taskContext` property (see [Trust Task Context Binding](#trust-task-context-binding)).

A witnessed exchange of a complete [[ref: DTG edge]] is bidirectional: two VRCs, one in each direction, are formed in a single witnessing event. For such exchanges the witness SHOULD issue one VWC per direction. In each VWC, `credentialSubject.id` MUST be the DID of the issuer of the VRC that the VWC attests (the VRC referenced by `digest`), so that the two VWCs of an exchange are unambiguously bound to their respective directions.

A VWC's `credentialSubject.id` and `taskContext` alone identify only the observed party and the trust task exchange, not the edge being witnessed. Binding a VWC to a specific edge therefore requires `digest`: a verifier holding the referenced VRC can recover both relationship endpoints (the VRC's `issuer` and `credentialSubject.id`) and confirm the exact credential the witness attested to. This binding is only as strong as the verifier's access to that VRC — a `digest` without the referenced VRC to hand is an opaque hash, not an identified edge. Issuers and holders presenting a VWC as evidence of a specific edge SHOULD make the referenced VRC available alongside it.

**Schema:**

- `type` (array, REQUIRED): MUST include `"WitnessCredential"`
- `issuer` (string, REQUIRED): DID of the witness — an [[ref: M-DID]], or the DID of a [[ref: VTA]] acting according to VTC policy
- `taskContext` (string, REQUIRED): `threadId` of the trust task exchange in which the witnessing occurred
- `credentialSubject` (object, REQUIRED):
  - `id` (string, REQUIRED): DID of the observed party
  - `digest` (string, REQUIRED): A cryptographic hash of the witnessed VRC, binding the VWC to the specific edge established. The hash MUST be computed as the SHA-256 hash of the credential's JSON representation canonicalized with the JSON Canonicalization Scheme ([JCS, RFC 8785](https://datatracker.ietf.org/doc/html/rfc8785)), and MUST be encoded as the string `sha256:` followed by the lowercase hexadecimal digest.
  - `witnessContext` (object, OPTIONAL): Context of the witnessing event
    - `event` (string, OPTIONAL): Human-readable event name
    - `sessionId` (string, OPTIONAL): Session or nonce identifier
    - `method` (string, OPTIONAL): Verification method used

**Example:**

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://firstperson.network/credentials/dtg/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "type": ["VerifiableCredential", "DTGCredential", "WitnessCredential"],
  "issuer": "did:web:witness-service.example",
  "validFrom": "2026-01-06T10:00:00Z",
  "taskContext": "thread-abc-123",
  "credentialSubject": {
    "id": "did:key:z6MkpTHR8VNs...",
    "digest": "sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
    "witnessContext": {
      "event": "EthDenver 2024",
      "sessionId": "session-abc-123",
      "method": "in-person-proximity"
    }
  },
  "proof": { "//": "..." }
}
```

### VEC (Verifiable Endorsement Credential)

**Purpose:** Attaches endorsements (skills, reputation) to a party. The verifiability applies to cryptographic assurance in the issuer's signature, not to the truth of the assertions, whose vocabulary is defined by the governing [[ref: VTC]] or [[ref: VTN]].

**Schema:**

- `type` (array, REQUIRED): MUST include `"EndorsementCredential"`
- `issuer` (string, REQUIRED): DID of the endorser
- `credentialSubject` (object, REQUIRED):
  - `id` (string, REQUIRED): DID of the endorsed party
  - `endorsement` (object, REQUIRED): Community/VTN-defined endorsement structure
    - Structure and fields determined by community policy

**Example:**

```json
{
  "@context": [
    "https://www.w3.org/ns/credentials/v2",
    "https://firstperson.network/credentials/dtg/v1",
    "https://w3id.org/security/suites/ed25519-2020/v1"
  ],
  "type": ["VerifiableCredential", "DTGCredential", "EndorsementCredential"],
  "issuer": "did:key:z6MkhaXgBZD...",
  "validFrom": "2026-01-06T10:00:00Z",
  "credentialSubject": {
    "id": "did:key:z6MkpTHR8VNs...",
    "endorsement": {
      "type": "SkillEndorsement",
      "name": "Software Development",
      "competencyLevel": "expert"
    }
  },
  "proof": { "//": "..." }
}
```

## Trust Task Context Binding

This section is normative.

DTG credentials are frequently issued during broader multi-step exchanges — [trust tasks](https://glossary.trustoverip.org/#term:trust-tasks) carried out through ceremonies governed by a [[ref: VTC]] or [[ref: VTN]]. A credential exchanged inside such a ceremony can be cryptographically valid as an artifact while still being insufficient evidence that the ceremony reached its intended terminal state. This section defines the mechanism that prevents such credentials from escaping their task context and being misinterpreted.

### Credentials versus Trust Task Artifacts

*This subsection is informative.*

The boundary between this specification and the planned DTG Core Trust Task Protocols specification is drawn by the following test:

- A **credential** is a durable claim about the graph that is true standing alone (e.g., VRC, VMC, VPC). It lives on after the exchange in which it was issued.
- An **artifact** is a work-product of a trust task (intermediate or completion), only meaningful within its exchange. It is carried as a Trust Task document, correlated by a shared `threadId`, with its terminal state expressed at the trust task layer — not as a new credential type.

**Test for any new thing:** true outside the exchange? → credential. Only meaningful inside? → artifact.

All seven credential types in this specification pass the credential side of this test. The [[ref: VDC]] is the boundary case that most clearly illustrates it: the delegation grant is durable and passes, while the invocation of a delegation does not and is left to the trust task layer (see [Grant and Invocation](#grant-and-invocation)). The structure of trust task completion artifacts (outcome evidence) is out of scope for this specification and will be defined in the DTG Core Trust Task Protocols specification.

### The `taskContext` Property

A credential whose meaning depends on a trust task completing MUST carry a `taskContext` property containing the `threadId` of the originating trust task exchange. This requirement is a property of the credential type, not a per-issuer choice:

- For credential types where this specification marks `taskContext` as REQUIRED (currently only the [[ref: VWC]]), issuers MUST include it.
- For all other DTG credential types, `taskContext` is OPTIONAL.
- A DTG credential without a `taskContext` property MUST be interpretable standing alone, independent of any exchange.

### Outcome Interpretability

A verifier MUST NOT interpret a `taskContext`-bearing credential as proof that the associated trust task or ceremony completed unless the matching trust task outcome evidence is also present and verified. That outcome evidence MUST be reachable by the verifier — either it travels with the presentation, or the `taskContext` value enables the verifier to locate it.

## Supporting Concepts

*This section is informative.*

### Personhood Credentials (PHC)

A [[ref: PHC]] is simply a [[ref: VMC]] issued by a [[ref: VTC]] whose governance enforces:

- Real human personhood
- Exactly one membership per person

No additional schema fields are required. PHC status is determined by governance and trust registries, not by credential structure. Issuers may optionally add `"PersonhoodCredential"` to the `type` array as a non-authoritative hint.

**Example:**

```json
{
  "type": [
    "VerifiableCredential",
    "DTGCredential",
    "MembershipCredential",
    "PersonhoodCredential"
  ],
  "issuer": "did:web:government-idv.example",
  "credentialSubject": {
    "id": "did:key:z6MkpTHR8VNs..."
  }
}
```

### Trust Registries

- **Authoritative source** for roles ([[ref: initiator]], [trust anchor](https://glossary.trustoverip.org/#term:trust-anchor), member, [[ref: IDVP]], etc.)
- Map DIDs to roles and policies
- Determine acceptable issuers
- Schema and APIs out of scope for this specification
- Handle revocations, etc.

### Identity Verification Credentials (IDVC)

- [[ref: IDVCs]] are **not** `DTGCredential` subtypes
- Any W3C VC satisfying a VTC/VTN's identity-proofing requirements
- Issuers, assurance levels, and requirements governed by VTC/VTN policy and trust registries

### Zero-Knowledge and Selective Disclosure

- This specification is **format-agnostic** (no binding to BBS+, SD-JWT-VC, etc.)
- Two ZKP constructions are defined for proving relationships: the [Pairwise Zero-Knowledge Proof](#pairwise-zero-knowledge-proof) (available to any two VRC holders) and the [Community-Anchored Zero-Knowledge Proof](#community-anchored-zero-knowledge-proof) (available when both parties hold VMCs from the same community)
- Schemas are kept simple to enable common predicates:
  - "Holder has valid VMC from recognized VTC"
  - "Issuer is authorized member"
  - "Two distinct VRCs exist"
  - "Holder has a valid, unrevoked delegation to act in the name of a member of a recognized VTC, covering act X"
- Detailed ZK protocols and registry-ZK interactions are left to future work

## Security Considerations

*This section is informative.*

1. **Proof verification.** Verifiers must cryptographically verify the `proof` of every DTG credential, including resolution of the issuer's DID and validation of the verification method, before relying on any claim in the credential.
2. **Validity period enforcement.** Verifiers must reject credentials outside their `validFrom`/`validUntil` window (or v1.1 equivalents) and should check applicable revocation status via the governing trust registry.
3. **Issuer authorization.** A cryptographically valid credential is not necessarily an authorized one. Verifiers must evaluate whether the issuer is authorized for the claimed role (e.g., a VMC issuer being a recognized VTC, a VIC issuer being permitted to invite) using the applicable trust registry or governance framework.
4. **Digest integrity (VWC).** A verifier relying on a VWC's binding to a specific edge must have the referenced VRC available, recompute the SHA-256 hash over its JCS (RFC 8785) canonical form, and confirm it matches `digest`; a mismatch invalidates the attestation. Without the referenced VRC in hand, `digest` cannot be resolved to an edge, and the VWC should not be treated as evidence of which edge was witnessed.
5. **Context collapse.** A credential presented outside the trust task exchange in which it was issued may be misinterpreted as evidence of a completed ceremony. The requirements of [Trust Task Context Binding](#trust-task-context-binding) exist to prevent this class of attack and must be enforced by verifiers.
6. **Replay of invitation credentials.** VICs should be issued with short validity periods and should be treated as single-use by the accepting [[ref: VTA]]/[[ref: PEP]], to prevent replay of an intercepted invitation.
7. **Key compromise.** Compromise of the private key controlling any DID used in a DTG credential (issuer or subject) undermines all credentials anchored to it. Key rotation and revocation procedures are governed by the applicable DID methods and trust registries.
8. **Misreading a VDC as a claim (VDC).** A [[ref: VDC]] establishes representation rather than asserting a fact, so a verifier that evaluates it with the logic it applies to the other DTG credentials will accept a party as standing in for another without having bounded what that party may then do. Verifiers must apply the scope, chain, and invocation requirements of [VDC (Verifiable Delegation Credential)](#vdc-verifiable-delegation-credential) in full, and must not accept a syntactically valid VDC as representation for anything outside its `scope`.
9. **Treating a delegation as a permission (VDC).** A VDC establishes that the delegate may act in the delegator's name; it says nothing about whether the act requested is one the delegator could perform. A verifier that treats a valid VDC as sufficient grounds to proceed lets a delegate do in the delegator's name what the delegator could not do itself, and a verifier that evaluates the delegate's own permissions instead of the delegator's answers the wrong question entirely. The checks are independent and must each be made, as set out in [How a Delegation Composes with Authority](#how-a-delegation-composes-with-authority).
10. **Delegation chain escalation (VDC).** Re-delegation that broadens scope, extends validity beyond the parent, or exceeds the permitted depth converts a narrow appointment into a wide one. Verifiers must evaluate every VDC in a chain, not only the one presented, and must reject any chain that does not resolve to a root delegation issued by the principal in whose name the acts would be performed.
11. **Delegation revocation latency (VDC).** Revocation is the delegator's only means of withdrawing an appointment already made, and its usefulness is bounded by how recently the verifier checked. Verifiers should check `credentialStatus` within a freshness window defined by the governing VTC or VTN, and delegators should keep `validUntil` as short as the delegated purpose allows rather than relying on revocation alone.
12. **Bearer use of a VDC.** A VDC that is presented without a demonstration of key control by the delegate proves only that a delegation exists. Verifiers must enforce [Invocation Binding](#invocation-binding); otherwise a captured VDC is usable by whoever holds a copy of it.
13. **Personhood laundering via delegation.** A [[ref: PHC]] asserts that its holder is a real person with exactly one membership. Because a delegate's acts are attributable to the delegator, a verifier that cannot distinguish the two may credit an agent with its principal's personhood, and may credit several agents of one person as several people. Verifiers must treat an act performed under a VDC as an act by the delegate in the delegator's name — never as an act by the delegator in person — and communities whose governance depends on personhood should state whether delegated acts are recognized at all.

## Privacy Considerations

*This section is informative.*

1. **M-DID reuse.** Reuse of an [[ref: M-DID]] across multiple relationships is allowed for bootstrapping, but implementers should carefully consider correlation risks. Migration from M-DID-based to [[ref: R-DID]]-based edges is recommended post-bootstrapping for enhanced privacy.
2. **R-DID uniqueness.** As required in [Unilateral Relationship Identification](#unilateral-relationship-identification), each entity must generate a new, unique R-DID for every entity it connects with. Reusing an R-DID across counterparties creates unintended correlation.
3. **Intentional correlation via personas.** Correlation across relationships should occur only through the holder's deliberate assertion of a [[ref: persona]] (via a [[ref: VPC]]) or an M-DID — never as a side effect of credential structure.
4. **Minimal disclosure.** DTG credential schemas are intentionally minimal so that holders can satisfy common predicates (membership, relationship existence) using zero-knowledge or selective disclosure mechanisms without revealing underlying DIDs or credential contents.
5. **Witness data.** The optional `witnessContext` of a [[ref: VWC]] may reveal information about where and when parties met. Issuers should include only what the witnessing purpose requires, and holders should be able to withhold `witnessContext` details when proving the attestation.
6. **ZKPs by default.** Implementations should use ZKP presentation by default so that privacy preservation does not require any extra effort on behalf of users.
7. **Delegation correlation.** A [[ref: VDC]] links a delegator and a delegate, and every invocation of it exposes that link to the verifier. Delegators should issue VDCs against an [[ref: R-DID]] scoped to the context in which the appointment will be exercised rather than against an [[ref: M-DID]] used across contexts, so that a delegate's activity in one context does not correlate its principal's activity in another.
8. **Scope terms as identifiers.** The `scope` of a VDC is community-defined text that may be narrow enough to identify the delegator, the delegate, or the underlying arrangement. Issuers should choose scope vocabularies that are no more specific than the appointment requires, and holders should be able to prove scope containment in zero knowledge rather than disclosing the full `scope` array.

## Governance Considerations

*This section is informative.*

This specification deliberately delegates most policy decisions to the governance frameworks of individual [[ref: VTCs]] and [[ref: VTNs]], consistent with the [ToIP Governance Metamodel](https://trustoverip.org/wp-content/uploads/ToIP-Governance-Metamodel-Specification-V1.0-2021-12-21.pdf):

1. Membership criteria, invitation policies, and identity-proofing requirements (including acceptable [[ref: IDVPs]] and [[ref: IDVCs]]) are defined by each community's governance framework and published via trust registries.
2. Whether a [[ref: VMC]] qualifies as a [[ref: PHC]] is a governance determination, not a schema property.
3. Endorsement vocabularies for [[ref: VECs]] and witnessing policies for [[ref: VWCs]] are defined by the governing VTC or VTN.
4. Delegation scope vocabularies for [[ref: VDCs]], the status mechanism used for their revocation, the freshness window within which verifiers must check it, and whether delegated acts are recognized at all where personhood is required, are defined by the governing VTC or VTN.
5. Whether a delegate must independently qualify — hold a [[ref: VMC]] of its own, or meet the same requirements as any other actor — before it may act in another's name is a governance determination, not a property of the VDC. A VDC establishes only that the appointment was made; see [How a Delegation Composes with Authority](#how-a-delegation-composes-with-authority).
6. New credential types proposed by higher-layer trust task protocol specifications are expected to be coordinated between the DTGWG task forces responsible for credentials and trust tasks.

## Internationalization Considerations

*This section is informative.*

Human-readable values in DTG credentials (e.g., endorsement names, witness event names) are community-defined and may appear in any language. Detailed internationalization guidance will be completed before this specification advances beyond Working Draft status.

## Accessibility Considerations

*This section is informative.*

This specification defines data structures rather than user interfaces. Accessibility guidance for implementations presenting DTG credentials to users will be completed before this specification advances beyond Working Draft status.

## Conformance

This section is normative.

This specification defines normative requirements, using the keywords defined in [Requirements Language](#requirements-language), for the following conformance targets:

### Conformance Targets

1. **Issuers** — entities that issue DTG credentials. A conforming issuer MUST produce credentials that satisfy the [Base Structure](#base-structure) and the schema of the concrete credential type, including the `taskContext` requirements of [Trust Task Context Binding](#trust-task-context-binding).
2. **Holders** — entities that store and present DTG credentials. A conforming holder MUST present credentials without altering their contents and MUST include reachable trust task outcome evidence when presenting `taskContext`-bearing credentials as evidence of task completion.
3. **Verifiers** — entities that verify DTG credentials and presentations. A conforming verifier MUST implement the verification requirements of the [Security Considerations](#security-considerations) and the outcome interpretability rule of [Trust Task Context Binding](#trust-task-context-binding), and MUST support W3C VC Data Model v2.0 verification per [W3C Verifiable Credentials Version Support](#w3c-verifiable-credentials-version-support).

### Conformance Tests

Conformance test suites for this specification have not yet been defined and are expected to be developed as the specification matures toward Working Group Approved Deliverable status.

## References

### Normative References

- [W3C Verifiable Credentials Data Model v2.0](https://www.w3.org/TR/vc-data-model-2.0/)
- [W3C Verifiable Credentials Data Model v1.1](https://www.w3.org/TR/vc-data-model/)
- [W3C Decentralized Identifiers (DIDs) v1.0](https://www.w3.org/TR/did-1.0/)
- [IETF RFC 2119: Key words for use in RFCs to Indicate Requirement Levels](https://datatracker.ietf.org/doc/html/rfc2119)
- [IETF RFC 8785: JSON Canonicalization Scheme (JCS)](https://datatracker.ietf.org/doc/html/rfc8785)
- [ISO 8601: Date and time format](https://www.iso.org/iso-8601-date-and-time-format.html)

### Informative References

- [ToIP Trust Registry Query Protocol](https://trustoverip.github.io/tswg-trust-registry-protocol/)
- [ToIP Governance Metamodel Specification V1.0](https://trustoverip.org/wp-content/uploads/ToIP-Governance-Metamodel-Specification-V1.0-2021-12-21.pdf)
- [Trust Tasks (ToIP Glossary)](https://glossary.trustoverip.org/#term:trust-tasks) and the community proposals at [trusttasks.org](https://www.trusttasks.org)
- [IETF RFC 7095: jCard: The JSON Format for vCard](https://datatracker.ietf.org/doc/html/rfc7095)
- [Agent2Agent (A2A) Protocol: AgentCard](https://agent2agent.info/docs/concepts/agentcard/)
- [Personhood Credentials (arXiv:2408.07892)](https://arxiv.org/abs/2408.07892)
- [Authorization Capabilities for Linked Data (ZCAP-LD)](https://w3c-ccg.github.io/zcap-spec/)
- [User Controlled Authorization Networks (UCAN)](https://github.com/ucan-wg/spec)
- [W3C Bitstring Status List v1.0](https://www.w3.org/TR/vc-bitstring-status-list/)
- [DTG Credentials v0.3 proposal draft](https://github.com/trustoverip/dtgwg-cred-tf/blob/main/dtg.md) (superseded by this specification)
