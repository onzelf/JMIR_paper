# What Counts as an FCaC Instantiation

**An invariant conformance criterion for Federated Computing as Code**

Enzo Fenoglio — Department of Computer Science, University College London
v0.2 — August 2026

---

## 0. Purpose and scope

Federated Computing as Code (FCaC) [1] states an architectural position: sovereignty-critical
constraints are enforced at execution boundaries by verifying artifacts carried with the request,
rather than by interpreting policy at runtime against shared state. Since publication, the
vocabulary has begun to circulate independently of the mechanism — *constitutional governance*,
*envelope*, *capability*, *admission* — including in work that addresses adjacent problems by
entirely different means.

Vocabulary in circulation is not a complaint; it is the normal fate of a useful distinction. It does
create a specific difficulty: without a stated test, "FCaC-based" becomes a claim that cannot be
checked, and any system using the words inherits expectations about assurance that its
architecture may not be intended to satisfy. Both directions of that confusion are damaging —
to FCaC, which acquires responsibility for properties it did not enforce, and to the borrowing
system, which acquires obligations it never accepted.

This note fixes the test. It states the invariant an architecture must satisfy to be an FCaC
instantiation, the conditions that disqualify one, and the observable checks by which either can be
established. It is deliberately narrow. It says nothing about whether FCaC is the right choice for
a given deployment; that question is answered by the deployment taxonomy in [1, §9], which is
explicit that a cohesive federation with a shared control plane may need none of this.

The criterion is stated once and applies to every presentation of the architecture. Descriptions of
FCaC written for particular audiences — technical, clinical, executive — are valid only where they
can be checked against the invariant stated here.

---

## 1. Definitions

**Boundary.** A point at which an operation passes from one sovereign administrative domain into
another. Sovereignty is the operative property, not network topology: two services inside one
trust domain do not form a boundary in this sense, however they are deployed.

**Participant.** An entity that may hold authority within a federation. Participants are either
*accountable in their own right* — humans and organisations, which are moral and legal agents —
or *accountable only by sponsorship* — automated actors, including AI agents, whose actions are
attributable to a sponsoring participant. Sponsorship transfers no accountability to the sponsored
actor and creates no autonomous agency. It follows that admission of a sponsored actor must be
correct *ex ante*: there is no downstream party to whom responsibility can devolve.

**Admission.** The decision to permit an operation to cross a boundary, taken before the operation
occurs. Admission is not authorisation: authorisation is a local access-control decision inside an
executing environment, taken under that environment's own authority and state.

**Envelope.** A governance object created under the federation's constitutive rules, fixing the
participants, the effective policy, and the period within which capabilities may be minted and
exercised.

**Federation.** A set of participants under shared constitutive rules with a gatekeeper issuing
ALLOW/DENY. The minimum federation has quorum = 1. A singleton federation — one constitutive
participant admitting others — is a full instance, not a degenerate one; it is the shape of a
provider admitting subscribers, and of a subscriber admitting its own members under transferred
accountability. Multi-party quorum is a property of a particular federation, not a requirement of
the architecture.

---

## 2. The invariant

> **An architecture is an FCaC instantiation if and only if every operation crossing a boundary is
> admitted by a prior decision satisfying C1–C7 below, and no operation crosses a boundary
> otherwise.**

The quantifier is universal and the second clause is not decoration. An architecture in which
*some* paths are admitted this way and others bypass the gatekeeper does not partially conform; it
does not conform. The value of the invariant lies entirely in there being no other way in.

*Prior* is likewise load-bearing. A decision taken while the operation is in flight, or reconstructed
after it, may be locally determined, deterministic and signed, and still not be an admission: there
was no point at which the operation could have been refused on the strength of a record that already
existed.

**C1 — Local determination.** The decision is taken at the boundary where execution will occur,
from material carried by the request together with trust anchors already held locally. It does not
consult an online policy service, an authorisation graph, a shared relationship store, or any
party other than the verifier itself.

**C2 — Deterministic and fail-closed.** The same request, the same presented artifacts and the same
trust anchors yield the same outcome at any verifier, at any site. A missing, malformed, expired or
unverifiable artifact yields DENY. There is no discretionary path, no default-allow, and no
operator override at the boundary.

**C3 — Fixed capability scope.** Permission is carried by a signed artifact whose bounds are set at
issuance and cannot be widened afterwards — not by the holder, not by the receiving service, not by
reinterpretation at the point of use. The artifact states which operations over which resources
under which constraints; anything absent is denied by absence rather than by a special rule.

**C4 — Holder binding.** Exercise of a capability requires a request-bound demonstration that the
presenter controls the key to which the capability was issued. Possession of the artifact alone is
insufficient. This is what distinguishes a capability from a bearer token, and its absence collapses
the architecture into the failure mode it exists to prevent.

**C5 — Governance-state continuity.** The capability is bound to the governance state under which
it was approved — the envelope, and the exact policy that envelope pinned. A capability approved
under policy A does not remain exercisable once policy B is in effect. Adding a new constitutive
rule changes the pinned state and therefore requires a new envelope; this cost is intrinsic, not an
implementation defect.

**C6 — Adjudicable evidence.** Every decision, ALLOW and DENY alike, emits a record signed under a
key traceable to the federation's trust anchors, carrying the outcome, the reason, the request and
artifact bindings, and the governance state in force. The record is verifiable by a party who
trusts none of the participating systems. Evidence reconstructible only from platform logs or
mutable configuration does not satisfy C6.

**C7 — Non-transfer under composition.** Where an admitted participant transforms a governed
resource into a derivative — the *rebind* operation of [3] — authority to perform the transformation
does not carry authority to release the result. Release to a recipient is a further boundary
crossing, admitted on the *recipient's* own capability and not on the transformer's. Permission to
consume a rebound derivative implies no permission to consume its source. A participant operates a
transformation under its own authority; it cannot lend that authority to whoever invoked it.

Three capabilities are therefore distinct and separately admitted: authority to perform the
underlying operation, authority to rebind its result into a derivative representation, and authority
to consume that derivative. A system that collapses any two of them fails C7.

---

## 3. What the invariant does not cover

Conformance to C1–C7 establishes that authority at the boundary is correct. It establishes nothing
else, and three exclusions are worth stating plainly because they are routinely read as claims.

**Containment is not admission.** Once an actor is admitted, what it does within its bounds is a
separate question answering to a separate mechanism. This is not a limitation to be noted and set
aside: admission governance is *necessary but not sufficient* for a complete governance plane, and
containment is a mandatory future requirement of the architecture rather than someone else's
concern. It is not currently implemented. An FCaC instantiation is a conforming admission layer, not
a complete governance plane.

**Procedural controls remain local.** Workflow state, quotas, consent checks, rate limits, drift
monitoring, incident response and the rest stay inside the executing domain under its own
authority [1, §4.2]. They may reject an admitted request on local grounds. Their presence is not
evidence of conformance and their absence is not evidence against it.

**Lifecycle beyond bounded validity is open.** Immediate revocation of a still-valid capability, and
withdrawal of a delegation link before its expiry, require revocation evidence distributed out of
band. Bounded validity and key rotation are within the base design; the rest is stated future
work [1, §6.4].

Cumulative disclosure across separately admitted requests is likewise outside the criterion. Each
admission is decided against the governance state in force, not against the history of prior
decisions; per-recipient evidence under C6 is what a history-dependent control would be built on,
not a substitute for one.

---

## 4. Disqualifiers

Each of the following is sufficient on its own to establish that an architecture is **not** an FCaC
instantiation, whatever vocabulary it uses.

**D1.** Admission is decided by a policy decision service, an identity provider, or any component
requiring shared runtime state across the boundary. *(Violates C1.)*

**D2.** Presentation of a valid token is sufficient to act — no request-bound proof of key control.
*(Violates C4.)* This is the OpenClaw pattern: a trust boundary established once at connection
time, after which the actor holds ambient authority.

**D3.** The receiving service may widen, reinterpret or supplement the granted scope at runtime.
*(Violates C3.)*

**D4.** Scope is communicated to the actor rather than enforced against it — an instruction in
configuration, documentation, or prose, with no gatekeeper positioned to refuse the request.
*(Violates C1 and C2.)* Stating a constraint to a system that is capable of exceeding it is not
enforcement.

**D5.** The effective policy can change without invalidating capabilities issued under its
predecessor. *(Violates C5.)*

**D6.** Decision evidence is unsigned, or exists only as application logs, or can be reconstructed
only by a party trusted by all participants. *(Violates C6.)*

**D7.** Some cross-boundary path bypasses the gatekeeper — an administrative interface, a
management API, a co-located service, a debugging route. *(Violates the universal quantifier.)*

**D8.** A participant's authority to transform or rebind a resource is treated as authority to
release the result to whoever requested it. *(Violates C7.)* The transformation is admitted; the
release is a separate crossing and is not.

**D9.** The decision is taken at execution time, or reconstructed afterwards, with no record in
existence before the operation. *(Violates the invariant.)* Runtime evaluation may be strong and
still not be admission: dynamic verification of a request in flight answers a different question
from whether the request should have been permitted to occur.

**D10.** Federation policy is expressed as versioned, machine-readable configuration, but no signed
prior admission record is produced per boundary-crossing operation. *(Violates the invariant.)*
Policy-as-code fixes the governance state; it does not decide admission. The "as code" paradigm is
how FCaC represents constitutive rules — it is not what makes an architecture an instantiation of
them.

---

## 5. Checking conformance

The criterion is falsifiable, and cheaply. Each clause corresponds to an observable behaviour of a
running system.

| Clause | Check |
|---|---|
| C1 | Sever the verifier's network path to every service except the requester. Admission decisions continue unchanged. |
| C2 | Replay an identical request against independently deployed verifiers: identical outcomes. Strip or corrupt each artifact in turn: DENY in every case. |
| C3 | Present a request one step outside the granted scope — a neighbouring operation, a wider cohort, a different resource class. DENY, with the reason naming the absent capability. |
| C4 | Present a valid capability with a proof generated under a different key. DENY on holder binding. |
| C5 | Change the effective policy. Capabilities issued under the prior policy are no longer admitted. |
| C6 | Take a decision record to a party holding only the federation's public trust anchors. They can verify the signature, read the outcome and reason, and confirm the artifact was not modified after issuance. |
| C7 | Have an admitted participant rebind a resource into a derivative, then request release of that derivative to a recipient holding no capability over it. DENY, with the reason naming the recipient's absent capability rather than the transformer's. Separately: confirm that a recipient admitted to consume the derivative is still denied the source. |
| Prior | For any admitted operation, the decision record exists and is verifiable independently of the operation's own artifacts, and predates them. |
| All | Enumerate every ingress to the protected domain. Each one either passes the gatekeeper or is not a boundary. |

A conforming implementation therefore exhibits a characteristic defect profile: findings take the
form *the invariant is stated and not enforced at point X*, and each is closable by a bounded
change. An architecture that generates unbounded findings of the form *this property is claimed and
no mechanism exists to enforce it* is not a conforming implementation with defects; it is a
different architecture.

---

## 6. Governance Questions

The questions FCaC addresses are not proprietary. Who may request an operation, which data and code
are authorised, what may leave a node, and how the resulting artefact can be reproduced and
challenged — these are the governance questions of any federated system, and many research
programmes address them. Shared questions are common ground, and work that arrives at them
independently or by way of FCaC is entitled to say so.

Shared questions are not shared architecture. A system that answers them through runtime policy
evaluation, ambient service identity, or unsigned provenance metadata is addressing the same
problem by different means. It is not an FCaC instantiation, is not a specialisation of one, and
should not be described as based on or derived from FCaC — for its own protection as much as for
FCaC's. The description creates expectations of admission artifacts, holder binding, and adjudicable
evidence that the system will be asked to produce and, having never intended to, cannot.

The correct statement of such a relationship is that the work is informed by governance questions
also articulated in FC and FCaC, while pursuing a different architecture and implementation
approach. Where a stronger relationship is intended, this note states the test.

---

## 7. Status

FCaC is implemented. The proof of concept accompanying [1] demonstrates envelope issuance,
boundary verification and envelope-triggered training; the session-admission instantiation in [2]
adds decision records as run artifacts, negative tests for capability and possession mismatch, and
sub-millisecond per-request admission cost. The OpenHealth demonstrator extends the model to
envelope-bound capabilities, signed governance evidence, policy-hash continuity, sponsored
non-human participants, and the composition case stated in C7.

Conformance claims made against this criterion should cite the version of this note in force at the
time of the claim.

---

## References

[1] E. Fenoglio and P. Treleaven. *Federated Computing as Code (FCaC): Sovereignty-Aware Systems by
Design.* arXiv:2603.17331, 2026.

[2] E. Fenoglio and P. Treleaven. *Auditable Session Admission for Cross-Silo Federated Learning.*
2nd International Conference on Federated Learning and Intelligent Computing Systems (FLICS 2026),
IEEE. https://ieeexplore.ieee.org/document/11621917

[3] E. Fenoglio and P. Treleaven. *Federated computing: information integration under sovereignty
constraints.* Royal Society Open Science, 13(2):251318, 2026.
