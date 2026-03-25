# Project-reviewer Skill References

This directory contains reference documentation for the project-reviewer skill.

---

## known-gap-patterns.md

# Known Gap Patterns — Project Reviewer Skill 2.0

This file is read at the start of every review session (Step 0).
Each entry describes a recurring gap type discovered during a past review.
Append new entries after every session that surfaces a genuinely new gap.

---

## GitHub pre-sets overridden when user switches to Remote endpoint path

**Check group:** New — not yet in a check group (candidate for G11 / X check)

**Gap signals:**
- STDIO = No and Local = No were set on a server whose input was a GitHub URL
- Hosting Provider GitHub = No was set even though a GitHub repo was confirmed
- Occurred after the user chose "Answer 1 — Remote endpoint" in the Endpoint Discovery prompt
- The Remote Endpoint pre-set rules (Remote=Yes, Local=No) were applied as a full override

**Root cause:**
Step 0 says "switch to Remote path / pre-set Transport/Deployment/TLS/Hosting" — the word "switch" implies all deployment attributes are replaced, causing the GitHub-origin pre-sets (STDIO=Yes, Local=Yes, GitHub=Yes) to be silently overridden to No.

**Recommended check addition:**
Add check: "If the input was a GitHub URL and any of STDIO, Local, or Hosting Provider GitHub is No — flag as FAIL. These three values are permanent regardless of path chosen."

**First seen:** 2026-03-24  |  **Skill:** mcp-server (JustCall report correction)

---

## Pricing: Free trial not counted as Free = Yes; Paid marked Yes alongside Free

**Check group:** New — candidate for G6 (Yes/No discipline) strengthening

**Gap signals:**
- Free = Yes AND Paid = Yes were both set for a service with a 14-day free trial
- Paid = Yes was not removed despite a confirmed free trial path existing

**Root cause:**
The old rule said "Paid = Yes if the platform has a mandatory paid plan" — which is technically true even when a free trial exists. The rule didn't distinguish between "paid plan available" and "no free access at all."

**Recommended check addition:**
Add check: "If Free = Yes, Paid must be No — they cannot both be Yes. Paid = Yes is only valid when there is zero free access path (no trial, no tier, no OSS)."

**First seen:** 2026-03-24  |  **Skill:** mcp-server (JustCall report correction)

---

## TLS assumed Yes for remote endpoints without curl verification

**Check group:** S11 (TLS = transport layer only) — extension

**Gap signals:**
- TLS 1.3 = Yes and TLS 1.2 = Yes were pre-set for a remote endpoint without running curl
- macOS LibreSSL `--tlsv1.3` flag returns curl error 4 (build limitation), which is not a server failure

**Root cause:**
The original rule said "Remote endpoint → TLS 1.3 = Yes, TLS 1.2 = Yes" as a blanket assumption rather than a verified result.

**Recommended check addition:**
Add check: "TLS values for remote endpoints must be verified with curl or openssl, not assumed. When macOS LibreSSL error 4 occurs on --tlsv1.3, fall back to openssl s_client or Cloudflare infrastructure evidence before marking TLS 1.3."

**First seen:** 2026-03-24  |  **Skill:** mcp-server (JustCall TLS verification)

---

## Hosting Provider: Both SaaS Vendor and GitHub marked Yes (dual-deployment violation)

**Check group:** G7 (Hosting Provider — single provider rule) — new

**Gap signals:**
- SaaS Vendor = Yes AND GitHub = Yes were both set for a server with vendor endpoint + GitHub proxy
- Occurred after rule change removed dual-deployment rule (2026-03-24)
- Developer believed "dual deployment" meant marking both hosting providers as Yes

**Root cause:**
Old guidance said "mark BOTH as Yes for dual-deployment servers." New rule (2026-03-24) changed to mutually exclusive: only ONE hosting provider should be Yes. Hosting Provider describes source/primary distribution, not connection method availability.

**Recommended check addition:**
Add check: "Hosting Provider must have exactly one Yes value. If SaaS Vendor = Yes, then GitHub, GitLab, Bitbucket, etc. must all be No. Only exception: if input was GitHub URL, pre-set GitHub = Yes AND pre-set Deployment/Transport — but mark SaaS Vendor = No unless vendor infrastructure is primary."

**First seen:** 2026-03-24  |  **Skill:** mcp-server (rule update — preventive documentation)
