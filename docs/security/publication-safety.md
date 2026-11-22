# Public documentation: security and confidentiality boundaries

**Audience:** anyone with access to the **public** QDIP documentation repository. This document is a publication rule, not evidence of security certification. Policy status: `PUBLIC_CONTENT_REVIEW_REQUIRED`.

## Allowed vs restricted content

| Publicly publishable | Requires private handling or explicit clearance |
|---|---|
| Generic SDK/API contracts, example synthetic inputs, conceptual system architecture, installation instructions for distributed artifacts | Real Worker/caller/admin credentials, bearer tokens, OAuth secrets, Zitadel/DNS challenges, signing keys, database credentials |
| Sanitized error semantics and capability scopes | Customer data, user identities, email/phone/address lists, organization IDs bound to real accounts, private endpoints, unredacted logs |
| Public product descriptions and approved grant claims | Internal threat findings, deployment identifiers, permission inventories, financial or personnel details not explicitly approved, unpublished invention claims |
| Clearly marked, non-sensitive research hypotheses | Private research datasets, proprietary coefficients or other unreleased IP |

Never assume `private` means safe to copy to a public Markdown page, example or generated DOCX. **Source links into private Core/Internal Docs are not a usable substitute for an independent SDK guide.**

## Before publishing or exporting

1. Classify **every changed file and binary attachment** as `PUBLIC`, `REVIEW_REQUIRED` or `RESTRICTED`. DOCX documents must be checked for tracked changes, comments, hidden text, core properties, embedded objects and body content, not just their titles.
2. Scan the changed content for credential-like values and private keys, authenticated URLs, real identifiers and personal data. Static regex scans provide **bounded** coverage; missing a pattern is not proof of absence.
3. Compare the publication text with current **source-pinned** API contracts; never promise public self-service, independent admission, hosted deployment or solver guarantees without completed real acceptance.
4. Review grants, IP strategy, commercialization, team and budget files for information disclosure and the owner's explicit permission. Generic templates are not automatically confidential, but actual names and negotiated amounts can be.
5. Check link destinations and Markdown layout at the intended renderer. Keep code blocks fenced and avoid wide 4–5-column tables in user-facing quickstarts.
6. Record reviewer, revision and decision in the PR. On finding exposed live credentials: stop distribution, rotate/revoke affected credentials, inspect access and incident scope; merely deleting text from the latest branch does **not** remove Git history or previous copies.

## 2026-11-22 bounded read-only findings

- GitHub repository metadata reports this repository as **public** and `forks_count=0` at inspection time. This does not rule out downloads, caches or external copies.
- GitHub code search for typical token fragments, private-key banners and common credential URI patterns returned **no indexed code matches** in eight targeted queries. GitHub workflow-runs and releases list endpoints each returned **0 items** for this repository during the same review; removed historical artifacts or other external copies are not ruled out. Code search does not inspect all binary documents or prove that historical commits are clean.
- Historical public content includes grant, IP, budget and team documents. A small set of Markdown documents was read manually; obvious literal credentials were not observed in that sample. **The entire 86-DOCX binary corpus has not been reviewed for embedded content or metadata.**
- Collaborator listings, GitHub App grants, deploy keys, organization access and all Actions artifacts cannot be enumerated with the connected repository read API. Do not claim an access-control audit passed.
- This policy intentionally does not disclose any internal organization/account IDs, tokens, deployment addresses or private source snippets.

**Result:** `BOUNDED_PUBLIC_MARKDOWN_SCAN / DOCX_PRIVACY_NOT_VERIFIED / ACCESS_GRANTS_NOT_VERIFIED`. Until a content/metadata review of public DOCX and history is completed, **do not assert NO_INFORMATION_LEAK**.
