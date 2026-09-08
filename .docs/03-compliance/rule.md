# AI Perfumery Engine — Legal & Compliance Rules (rule.md)

**Company / Product:** AI Perfumery Engine — ระบบผู้ช่วยคำนวณและออกแบบกลิ่นน้ำหอมด้วยฟิสิกส์เคมี
**Course:** 1305493 SE Case Studies, 1/2569 — Week 2 in-class case
**Status:** Living document. Read this before writing any code that touches user data or user actions.

---

## 0. Scope — what this product actually holds

The engine takes a chemical/physical dataset of 100 aroma materials (supplied by the domain expert),
applies interaction rules (synergy / masking / evaporation over time, thresholds, material groups),
and shows the predicted result on a dashboard. Around that engine there are **accounts, logins,
saved formulas, evaluation panels and agreements** — that is where the law bites.

| Class | Examples in this system | Governing rule below |
|---|---|---|
| **A. Personal data** | perfumer/user account (name, email, role), `created_by` / `edited_by` on a formula, client-brief contact, panel-tester identity, interview notes from DISCOVER | PDPA |
| **B. Sensitive personal data** | skin allergy / sensitisation records, patch-test results, pregnancy status, "alcohol-free / halal only" preference (**reveals religion**) | PDPA — sensitive |
| **C. Owner IP / trade secret** | the 100-material dataset, interaction logic, thresholds, group definitions, saved formulas | not PDPA, but §0.1 below |
| **D. Access / traffic logs** | login, export, formula edit, calculation run | Computer Crime Act §26 |
| **E. Agreements** | consent tick-box, IP-assignment acceptance by co-developers, "approve this formula for production" sign-off | Electronic Transactions Act |

### 0.1 Owner-IP rules (contract, not statute — but non-negotiable)

- The agent must **never** send the material dataset, interaction rules, thresholds or any saved formula
  to a third-party AI/API/cloud service that is not on the approved list, even for "just testing".
- The agent must **never** make the repository, database dump, or dataset public, and must not commit
  real dataset files to a public branch, a Gist, a screenshot, or a demo deployment.
- If a feature requires an external service to see class-C data, **stop and ask the project owner in
  writing first**. Silent adoption of a new SaaS/model provider is a breach.
- Every generated file must carry the ownership header agreed with the owner; the agent must not add
  its own licence file (MIT/Apache) to this repo.

---

## 1. PDPA (Personal Data Protection Act B.E. 2562)

**What it is (TH):** กฎหมายคุ้มครองข้อมูลส่วนบุคคล — ถ้าระบบเก็บ "คน" ไม่ใช่แค่ "สาร" เรามีหน้าที่ตามกฎหมายทันที
**What it is (EN):** Thailand's data-protection law. The moment this system stores something that identifies
a real person — a perfumer's email, a panel tester's name, an interview note — we become a data controller
with duties, and that person gets rights we must ship as **features**.

**What it requires:** lawful basis (consent) · purpose limitation · data minimisation ·
access / correct / delete · extra protection for sensitive data (health, religion, biometrics).

### Rules for the agent

**Collecting & storing**

1. If the system stores a person's name, email, phone, photo or free-text note about a person, it must first
   record a consent record, and the write must fail (not warn) if `consent_id` is null.
2. If the agent creates any table containing personal data, it must add these columns in the same migration:
   `purpose_code`, `consent_id`, `created_at`, `retention_until` — no personal-data table ships without them.
3. If the agent designs the user account, it must store **only** `display_name`, `email`, `password_hash`,
   `role`, `organisation`. It must not add date of birth, ID-card number, home address, or gender
   "for later" — if a field has no stated purpose today, do not create the column.
4. If a screen asks for a field that is not used by the engine or the dashboard, the agent must delete the
   field from the design instead of storing it. "Store everything in case" fails PDPA minimisation.
5. If the system records evaluation-panel results (people smelling a blend and rating it), it must store the
   rating against a **pseudonymous panelist code**, and keep the code-to-identity mapping in a separate table
   that only the `panel_admin` role can read.
6. If the system stores a client brief for a perfumer, contact details belong in a `client_contact` table
   linked by id — the agent must not copy customer names into the formula's free-text notes.

**Sensitive data (class B) — highest bar**

7. If a feature needs allergy, skin-reaction, patch-test, pregnancy or medical-restriction data, the agent must
   **stop and ask the owner before implementing it**, and must not create the column until an explicit written
   plan exists (Guardrail 4).
8. If the goal is only "this person must avoid material X", the agent must model it as a non-medical flag
   (`avoid_material_id`, `reason = 'user_preference'`) instead of storing a diagnosis or a test result.
9. If the system offers an "alcohol-free / halal" preference, it must treat that field as sensitive
   (religion-revealing): explicit separate consent, encrypted at rest, excluded from every export,
   report, CSV and analytics event.
10. Sensitive fields must never appear in logs, error messages, stack traces, crash reports, or LLM prompts.

**Rights as features**

11. If the system has user accounts, it must ship three real screens/endpoints in the same sprint as login:
    **`GET /me/data` (export as JSON/CSV), `PATCH /me` (correct), `DELETE /me` (erase)**.
    "Email the admin" is not an implementation.
12. If a delete request is executed, it must cascade to derived rows (panel entries, comments, session rows,
    search caches), and the agent must list in the PR description every table touched and every table
    deliberately kept.
13. If personal data exists in backups, the deletion routine must record a `pending_backup_purge` entry with a
    date; the agent must not silently leave the data in backups without documenting it.
14. A delete must **not** delete the §26 access log — see rule 30. The agent must instead strip the profile
    and keep the minimum identifier, and must document this exception in the privacy notice.
15. If consent is withdrawn, the system must stop the processing for that purpose within 24 hours and mark the
    consent row `withdrawn_at` — it must not hard-delete the consent row (that row is our evidence).

**Purpose, sharing, transfer**

16. If data was collected for "using the perfumery engine", the agent must not reuse it for marketing,
    newsletters, model training or ranking. Any new purpose = new purpose code + new consent.
17. If the agent adds analytics, telemetry, crash reporting or a third-party widget, it must not send
    `user_id`, email, or formula content, and the vendor must be named in the privacy notice first.
18. If any component (DB, storage, LLM API, hosting) is outside Thailand, the agent must list it in
    `docs/privacy/transfers.md` and flag it to the owner before deploying.
19. The agent must never paste real interview transcripts, user emails, or panel records into a public LLM.
    For development it must generate clearly-labelled **synthetic** records (`seed_demo_*`), and must never
    present synthetic people as real research participants.
20. If a bug requires production data to reproduce, the agent must use a masked copy — never a raw dump.

**Access control, breach, notice**

21. Every read of personal data must be role-checked server-side; a perfumer may read only their own profile
    and their own formulas. Hiding a button on the client is not access control.
22. If an admin views another user's personal data, the system must write an access record
    (who, whose data, when, why).
23. If the system detects a personal-data breach, it must alert the owner immediately and support notifying
    the PDPC **within 72 hours** — the agent must build an incident log, not rely on an ad-hoc email.
24. The privacy notice must live in the repo as a versioned file (`docs/privacy/notice-v{n}.md`).
    If the notice changes materially, the system must re-ask for consent, not silently update the text.
25. Retention: DISCOVER interview notes and recruitment data must have an explicit deletion date; the agent
    must implement a scheduled purge, not a manual promise.

---

## 2. Computer Crime Act B.E. 2550, Section 26

**What it is (TH):** ผู้ให้บริการต้องเก็บ "ข้อมูลจราจรทางคอมพิวเตอร์" (ใคร/เมื่อไหร่/จากไหน) ไม่น้อยกว่า 90 วัน
และเก็บข้อมูลที่ระบุตัวผู้ใช้ต่ออีก ≥90 วันหลังเลิกใช้บริการ — ไม่ทำ ปรับไม่เกิน 500,000 บาท
**What it is (EN):** If people log into our system, we are a "service provider" (§3 covers any app that stores
data for other people's benefit — not just ISPs). We must be able to say **who accessed what, and when**, and
keep that trail for at least 90 days.

**What it requires:** an access/traffic log tied to a real user, retained ≥90 days (extendable to 2 years on
official order), including ≥90 days after the user stops using the service.

### Rules for the agent

26. If the system has a login, it must create the `access_log` table in the **first** sprint that adds
    authentication — logging is a Must backlog item, not a later hardening task.
27. Every access-log row must contain: `event_id`, `actor_user_id` (or `role` + session id for pre-auth
    events), `source_ip`, `user_agent`, `occurred_at` (UTC, ISO-8601 with offset; displayed as Asia/Bangkok),
    `action`, `target_type`, `target_id`, `result` (success/fail).
28. The system must log at minimum: login success, login failure, logout, password change/reset, role or
    permission change, account creation/deletion, formula create/update/delete, **calculation run**,
    **every export or download** (CSV/PDF/JSON of formulas or the material dataset), API-key use,
    and any admin access to another user's data.
29. Retention must be **≥90 days** and configurable upward to 2 years. If the agent writes a TTL, cron job,
    log-rotation policy, S3 lifecycle rule or container log limit shorter than 90 days, that is a defect —
    the agent must refuse the config and say why. Saving storage cost is not a valid reason.
30. If a user deletes their account, the system must keep the identifying information linked to their past log
    entries for **at least 90 days after the account ends** (§26 paragraph 2) in a restricted
    `retained_identity` table, then purge it automatically.
31. The log is **metadata, not content**: the agent must log `formula_id` + `version`, not the formula body,
    the chat text, or the material percentages. Logging content is both unnecessary under §26 and a PDPA
    minimisation failure.
32. Logs must be append-only: the application database role gets INSERT and SELECT on `access_log`, and no
    UPDATE or DELETE. Purging runs under a separate scheduled role.
33. The agent must add tamper-evidence: a per-row hash chained to the previous row (or an external write-once
    log store), so an altered log is detectable.
34. Timestamps must come from the NTP-synced server clock. The agent must never write a client-supplied
    timestamp into the access log.
35. If authentication uses Firebase / Google OAuth / any external identity provider, the system must still
    write its own access log inside our system. Relying only on the provider's console is a §26 failure.
36. Reads of the access log must themselves be logged (who queried the logs, when, for what date range).
37. Logs must survive redeploy: never write them only to an ephemeral container filesystem, and include them
    in backup and restore testing.
38. The agent must build an admin query/export screen that can answer "what did user X do between date A and
    date B" within minutes, because that is the shape an official request takes.
39. Any environment with real users (production, and staging if real accounts log in) must have full logging
    enabled; the agent must not gate logging behind a flag that defaults to off.
40. The agent must record in `README.md` who the log custodian is, where logs are stored, and the current
    retention setting.

---

## 3. Electronic Transactions Act B.E. 2544, §9 / §26 / §28

**What it is (TH):** การกดยอมรับ/ติ๊ก checkbox มีผลทางกฎหมายเป็น "ลายมือชื่ออิเล็กทรอนิกส์" ถ้าพิสูจน์ได้ว่าใครเป็นคนกด
แสดงเจตนาอะไร และวิธีนั้น "น่าเชื่อถือพอ" กับมูลค่า/ความเสี่ยงของธุรกรรมนั้น
**What it is (EN):** An electronic action counts as a signature if it (1) identifies the signer and shows their
intent, and (2) uses a method reliable enough for the value and risk of that transaction (§9). Meet the four
control/tamper tests and the law presumes it reliable (§26). Issuing certificates to other people makes you a
Certification Authority with its own duties (§28) — we are not going to do that.

**What it requires:** identify + intent + reliability proportionate to risk (§9) · signer-linked,
signer-controlled, alteration-detectable records (§26) · CA duties if you ever issue certificates (§28).

### Rules for the agent

**Recording any agreement**

41. If the user clicks "I agree", "Accept", "Approve" or "Confirm" on **anything**, the system must record:
    `user_id`, the signer's name at that moment, `signed_at` (server clock, UTC + Asia/Bangkok),
    `document_type`, `document_version_id`, `document_hash` (SHA-256 of the exact text shown),
    `auth_method`, `ip`, `user_agent`. "The user clicked OK" with no timestamp, no text version and no user id
    fails §9's own reliability test.
42. The system must store an **immutable copy or hash of the exact text version the user saw** — never just a
    link to a page whose content can change later.
43. Consent and agreement texts must be versioned files in the repo. If the text changes, the agent must create
    a new version id and must not retro-apply it to existing signature records.
44. The tick-box must be unticked by default, the button must state the actual act
    ("I agree to assign IP in this project", not "Continue"), and intent must be a separate deliberate action —
    the agent must never infer agreement from scrolling, browsing, or continued use.
45. Signature records are append-only: the agent must never UPDATE or DELETE a signature row. A correction
    creates a new row that references the old one.

**Reliability must scale with risk (§9)**

46. Low-risk acts (free account sign-up, accepting the privacy notice) may be a tick-box plus a logged session.
47. High-risk acts must require **re-authentication at the moment of signing** (password re-entry or OTP), and
    the method used must be stored in `auth_method`. In this product, high-risk means:
    - accepting the **IP-assignment / confidentiality agreement** as a co-developer,
    - approving a formula for production or external release,
    - exporting or transferring the material dataset,
    - granting another account admin or dataset-read rights,
    - deleting a user account or a formula's version history.
48. The agent must not apply one signature standard to every feature; a per-action risk level belongs in the
    design doc before the code is written.

**Presumed-reliable tests (§26)**

49. The signing credential must belong to one identified person: no shared team accounts, no shared login on
    the lab machine, no "signed by the admin on behalf of X". If the design needs delegation, stop and ask
    the owner.
50. If an admin-impersonation ("view as user") mode exists, the system must block every signing action inside
    it and mark the session as impersonated in the log.
51. The system must make alteration detectable on both sides: hash the signature record **and** the signed
    document, and expose a verification view showing who signed, when, which version, and the hash.
52. The signer must be able to download or receive a copy of exactly what they signed, immediately after signing.
53. Withdrawing consent must not erase the signature evidence — mark it withdrawn and keep the record under its
    own retention rule (see rule 15).

**Certificates (§28)**

54. The agent must not build a Certificate Authority, issue digital certificates to outside parties, or design a
    "we issue signing keys to clients" feature. If a requirement seems to need one, stop and escalate — that
    triggers §28 duties plus §§32–34 registration/licensing.
55. If certificate-grade identity is ever required, the system must integrate an existing CA
    (Thai Digital ID, a bank, or DBD e-Certificate) rather than becoming one.
56. Self-signed certificates are allowed only for internal transport inside our own closed environment, never
    issued to an outside party as proof of identity.
57. If any PKI-based signing is added, the design must include a key-compromise report channel and an immediate
    revocation path, plus a revocation/expiry check before a signature is accepted.

---

## 4. Extra — AI-specific rules (ETDA principles; beyond today's three laws)

The engine *predicts* how materials interact. A wrong prediction can end up as a blend someone puts on skin,
so the Air Canada rule applies: **we own what our system says.**

58. Every calculated result must show which rule, threshold and source rows produced it — no unexplained score.
59. If the engine has no rule covering a material pair, it must return "insufficient data", never a guessed value.
60. Any generative/LLM feature must be labelled as a suggestion, must cite the dataset rows it used, and must
    never invent a material, a CAS number, a concentration limit, or a safety claim.
61. Every safety-relevant output must carry the disclaimer agreed with the owner and a human-override path.
62. The core workflow (browse materials, record a formula) must still work when the AI/model service is down.
63. Users must have a feedback channel to report a wrong result, and every report must be logged as a defect.

---

## 5. Legal requirements for the W3 backlog

| ID | Law | Requirement | Priority |
|---|---|---|---|
| **LR1** | PDPA | If we store a user's name/email or any panel-tester record, we take consent first and ship view / correct / delete as real features; sensitive fields need the owner's written plan. | Must |
| **LR2** | CCA §26 | Keep an append-only access log (who, when, from where, what action) ≥90 days, including ≥90 days after an account ends. | Must |
| **LR3** | ETA §9/26 | Every "I agree" / approval is recorded with user id, timestamp, text version and hash; high-value acts (IP assignment, production release, dataset export) require re-authentication. | Must |
| **LR4** | ETDA principles | Explainable results, "insufficient data" instead of guesses, human override, and a core workflow that works without the model. | Should |
| **LR5** | Owner IP | Dataset, rules and formulas never leave approved infrastructure; the repo stays private. | Must |

**Backlog note:** LR1–LR3 go into the same sprint as login and "save a formula". Compliance sits inside the core
workflow, not in a hardening sprint at the end.

---

*Refs: PDPA B.E. 2562 · Computer Crime Act B.E. 2550 §3, §26, §27 (fine ≤ 500,000 THB) ·
Electronic Transactions Act B.E. 2544 §9, §26, §28, §§32–34 · ETDA "Digital Thailand AI Ethics Guideline" (2019).*


note Athichon kaewla 6631503046 wrote it.
