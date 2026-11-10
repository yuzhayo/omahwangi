# WANGI Workspace Authority

This file is the local authority for work under `C:\WANGI`.

## Authority order

1. The operator's explicit instruction.
2. This file.
3. `agent_setup_guide.md` for the deployment procedure.
4. `catalog_website_plan.md` for product and technical requirements.
5. `SETUP-LOG.md` for current state and produced values.
6. `.memory\browser.md` for dated browser observations.

If two sources disagree, stop and report the conflict unless the operator has already
resolved it.

## Workspace shape

```text
C:\WANGI\
  AGENTS.md
  agent_setup_guide.md
  catalog_website_plan.md
  SETUP-LOG.md
  catalog-store\
  .memory\browser.md
  .opencode\agents\wangi.md

C:\RARA\BROWSER-AUTOMATION\
  Default\
  0Browser\
```

The external RARA path is one shared browser exception. WANGI may read its launcher
documentation and attach to its running Chrome on port 9333. WANGI must not modify the
profile, launcher files, other workspace tabs or RARA project data.

## Live surface boundary

| Surface | Allowed access |
|---|---|
| Registrar for `omahwangi.web.id` | Section 2.1: inspect Kelola DNS and Email Forwarding read-only. After that check: write only the Nameserver page. Keep Kunci Domain Aktif. Never open Kode EPP. Never modify DNS records, forwarding or contact details. |
| Cloudflare Free account | Create the zone, `catalog-photos` bucket, two Pages projects, one R2 custom domain, one secret and one binding. No purchase, upgrade or card entry. |
| Google Sheet `catalog` and its Apps Script | Create and deploy. Operator handles login, authorization prompts, passwords, 2FA and CAPTCHA. |
| Private GitHub repo `catalog-store` | Create and push. Never commit a secret. |
| Anything else | Do not open, read or modify it. |

## Browser rules

- Use only `C:\RARA\BROWSER-AUTOMATION` and attach only through port 9333.
- If port 9333 is down, the operator starts `0Browser\start-automation-chrome.bat`.
- Never launch a second Chrome, create a profile or change `--user-data-dir`.
- Never move, copy, rename or edit the shared browser profile.
- Never use `Stop-Process -Force` or `taskkill /F` on Chrome.
- Stop only on an explicit operator order through `0Browser\stop-automation-chrome.bat`.
- Pin target tabs by URL identity. Never use `pages[0]`.
- Do not touch tabs belonging to other workspaces.

## Deployment rules

- Follow guide sections in order and complete one step at a time.
- Before changing a live surface, state the guide step and exact action.
- Verify navigation before clicking and verify every write by read-back plus outcome.
- Stop for login, 2FA, CAPTCHA, payment, DNSSEC warnings, existing domain email, unknown
  failures or two failed attempts.
- Keep Cloudflare on Free. Do not substitute another host, storage service or backend.
- Keep the domain lock active.

## Secret rules

- `WRITE_TOKEN` may exist in `SETUP-LOG.md`, Apps Script and the Cloudflare Pages secret.
- `WRITE_TOKEN` must never exist in Git, public config, screenshots or browser memory notes.
- `READ_TOKEN` is public by design.
- Do not print secrets during checks.

## File rules

- ASCII for project instructions and code. UTF-8 without BOM and LF line endings.
- Search before creating a file. Do not create duplicate files for the same job.
- `SETUP-LOG.md` is local state and must remain git-ignored.
