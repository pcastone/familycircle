# Business Process & Philosophy — Eldercare Coordination Platform

Status: Living document · Companion docs: `platform-spec.md` (build), `adr.md` (decision record)
Audience: humans — founders, grant writers, partner orgs. Agents read this for *why*, not *what to build*.

---

## 1. The problem, stated honestly

When adult children support an aging parent, the incumbent solution is a group text plus a shared drive folder. It is free, already installed, and real. It fails anyway, for one reason: **it still requires a single human to be the scheduler, the memory, the reminder system, and the dispatcher.** One sibling becomes the unpaid project manager of their parent's decline, on top of a job, a family, and a life. The tool doesn't fail; the person does.

The second failure is sharper: **in an emergency, the information isn't there.** Parent unresponsive, ER hallway, no signal in the building, and the med list / allergies / DNR / insurance card are in a drawer in another state, or in the head of the sibling who isn't answering.

The third failure is slow: nobody holds the aggregate. The decision to reduce a parent's independence — the car keys, the checkbook, eventually their own record access — gets made by whichever sibling witnessed the scariest single event, argued against by the ones who weren't there. There is no shared evidence, so there is only shared conflict.

## 2. What we are

Three things, in priority order:

1. **Crisis mode** — the emergency payload, offline, on every primary caregiver's phone, readable in seconds. This is the hook and the reason to install.
2. **The broadcast** — one action informs the whole family and the extended circle, so the person in the hallway isn't also the switchboard.
3. **The incident log** — every emergency, fall, ER visit, and missed check-in, timestamped, accumulating. This is the accruing value: it converts the hardest family conversations from memory-against-memory into a shared record. It is also our outcomes instrument (§6).

The escalation continues past the worst day. Sometimes emergencies end in death, and the family's need doesn't stop at the hospital door — it runs from *seconds* (DNR, POA) through *hours* (organ donation, funeral instructions) to *days* (prepaid plan, attorney, executor) to *weeks* (insurance claim, probate). The product follows the family through that entire arc. In v1 the estate layer is **pointers, not documents**: where the will is, who the lawyer is, which carrier and policy number — because the family's actual problem is that nobody knows where anything is.

## 3. What we are not

- **Not a nicer Google Drive.** Passive storage is the incumbent; we don't compete with free by being free-with-more-fields.
- **Not a medical device or clinical system.** We hold family-entered information; we do not diagnose, dose, or integrate with EHRs in MVP.
- **Not a surveillance product.** The elder is a participant with standing, not a monitored object (§4).
- **Not a scoreboard.** See §5. This is a design principle with teeth in the spec.

## 4. The elder's dignity is a structural commitment

The elder is the subject of the circle and, with permission, a limited user: check in, add appointments, update their information. Over time those permissions will be reduced — that is a stated purpose of the product, and it is the most emotionally violent moment we will ever mediate.

Commitments, in priority order:

1. **The elder can always see their own record and the incident log about themselves**, even with zero write access. Transparency without control is still dignity. This is an invariant in the spec, not a preference.
2. **Reduction of access is evidence-based, not vibes-based.** The incident log exists so the family conversation is "here are eleven incidents in four months," not "I just feel like Mom's slipping."
3. **Onboarding asks almost nothing of the elder** — scan a code, done — but "the family sets everything up" must never curdle into "everything was done about them." The record is *about* them and visible *to* them from day one.
4. Elder-facing screens are held to the strictest accessibility bar in the product: one-tap check-in, no gestures, large targets, honest fonts.

We also acknowledge an uncomfortable truth: elder financial abuse is overwhelmingly committed by family, and the relative who volunteers to control the information is not always volunteering out of love. Our mitigations are structural: no single-owner circles (two primaries minimum), executor changes are logged where the family can see them, and every read of sensitive data leaves an audit trail visible to all primaries. We cannot prevent abuse; we can refuse to be a convenient instrument for it, and we can make it visible.

## 5. Family conflict: designed for, not wished away

Siblings fight about eldercare. This product makes labor visible, and visible labor becomes a ledger, and ledgers become weapons. Our policy:

- **Forward-looking visibility, yes.** Unclaimed needs ("Mom needs a ride Thursday — no one has claimed this") in front of every sibling is coordination pressure, and it is the honest version of the product's promise.
- **Backward-looking per-person totals, never.** "You: 47 tasks. Him: 3" is not a feature; it is an exhibit in a future estate contest and an accelerant for every existing resentment. The platform never computes or displays historical per-person contribution counts. (Enforced in the spec's CI gates, because a well-meaning dashboard request will eventually ask for it.)
- **The log survives death and can be disclosed** by the immediate family to hospitals, police, insurers. We therefore write it like it will be read by an adversary someday, because it will: user-asserted event times are kept separate from immutable record times, corrections are amendments rather than edits, and nothing in it editorializes.

## 6. Grant funding shapes the operation

The goal is help, not profit. Funding is grant-based. Consequences we accept deliberately:

- **No growth hacking.** No viral loops, no engagement mechanics, no dark patterns to expand the circle. Families invite whom they trust.
- **Evidence is a deliverable.** Renewals require outcomes. The incident log doubles as the instrument: time-to-information in emergencies, ER-visit patterns, check-in adherence. We add a validated caregiver-burden measure (e.g., Zarit short form) at onboarding and intervals, so year-one produces a pre/post story a program officer recognizes. Measurement is designed in at month one, not bolted on at renewal time. All reporting aggregates are per-circle/per-period — never per-person (§5).
- **Distribution will likely be partner-mediated** — Area Agencies on Aging, senior centers, possibly health systems. That implies an eventual caseworker/admin surface (onboard 40 families, not 1) and it implies the compliance watch-point below.

## 7. Compliance posture: build BAA-shaped, sign nothing

- Today we are outside HIPAA: families recording information about their own parent makes us a personal record keeper, not a covered entity or business associate.
- The trigger to watch is **partner data flow**: the moment a health system or clinically-integrated agency enters or transmits patient information *through us on its behalf*, we become a business associate and need a BAA — with breach-notification duties, direct OCR liability, subprocessor agreements, and a documented risk analysis.
- That trigger most often arrives **silently, in our own grant language** ("discharge planners will onboard patients into the platform"). Every grant proposal is reviewed against this sentence-pattern before submission.
- Strategy: the architecture already carries the expensive parts (encryption at rest, tier-4 enclave separation, read auditing, telemetry redaction), so becoming a BA later is a contracts-and-process exercise, not a rebuild. We defer the paperwork, not the engineering.
- Not-HIPAA is not unregulated: the FTC Health Breach Notification Rule covers consumer health apps, and state law (e.g., California's CMIA) can exceed HIPAA. Michigan's medical-records provisions apply to us as a Michigan-based effort.

## 8. Honest promises (privacy-policy commitments)

These go in user-facing copy in plain language, because trust is the product:

1. **What we can see.** The service can read the data families store (this enables recovery when a phone is lost, family notifications, and anonymous outcome reporting for our funders). Sensitive estate/identity items live in a separated, additionally encrypted enclave with every access logged and visible to the family's primaries.
2. **What leaving the circle means.** When someone is removed, their device is instructed to erase its copy the next time it connects, and they receive nothing new from that moment. We are honest that a device that never reconnects keeps what it already had — no app on any phone can truly reach into a device it no longer controls. We say this out loud instead of implying otherwise.
3. **What we never do.** No ads, no data sale, no per-person contribution scoreboards, no content of your family's records in any notification, log, or report.
4. **What happens at death.** The circle persists as an estate-coordination space; the named executor's role becomes central; the incident log remains available to the immediate family, who control any disclosure to hospitals, police, or insurers.

## 9. Open business decisions

| # | Decision | Needed by | Notes |
|---|---|---|---|
| 1 | Grant target: federal (ACL/NIA/AHRQ) vs. private foundation vs. state/AAA pass-through | Before first submission | Determines partner-org exposure (§7) and reporting format (§6) |
| 2 | Two-primary quorum for revoking a primary: confirm as policy | Before v1 | Schema already supports it |
| 3 | Clinician hand-off view + printed wallet-card fallback | PoC scoping | "Who holds the phone in the ER?" |
| 4 | Extended family: native app vs. web-only feed | v1 | Web-only halves mobile surface |
| 5 | Enclave security-design session (keys, ceremonies, consent copy, document storage) | Before tier-4 GA | Dedicated session; engineering contract already frozen in spec §9 |
| 6 | Post-death log visibility default for extended family | Before v1 | Immediate-family control confirmed; default view state isn't |
| 7 | Name, brand, and the one-sentence pitch | Grant submission | Current working pitch: *"Everything your family needs on the worst day, and the shared record that makes the hard decisions before it."* |
