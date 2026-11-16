# Handoff: remote-job-sites link and category audit, 2026-09-30

## Task

Audit every site listed in `GUIDE.md`, which is generated from `data/entries/*.yaml`.
Confirm that each URL still resolves and points at the site it claims to.
Check that each entry carries the right `status` and `category`.

Done means the audit verdicts are applied to the yaml files. `validate.py`, `dedupe.py`, and `build.py` are clean. The changes are committed and pushed, so the GitHub copy of `GUIDE.md` matches local.

## State

The audit ran and every verdict is applied to the working tree. Nothing is committed.

`git status` shows 89 changed files: 75 modified entries, 8 renamed entries, 5 deleted entries, and `GUIDE.md`. The `site/` folder is gitignored and built by automated checks, so it is not part of the commit.

Totals moved from 2326 entries (2067 live / 92 blocked / 167 dead) to 2321
entries (2054 live / 51 blocked / 216 dead).

Applied:

- 26 entries flipped `blocked` to `live`. In a real browser they return 200
  with real content, so the 403/429 was the request-based checker, not the site.
- 49 entries flipped to `dead`. Name lookup failures, expired secure connections, 404, 500, 522, and
  30s timeouts, all confirmed in a real browser rather than only by lychee.
- 8 entries repointed at the URL they now redirect to, with the file renamed to
  match the new host: `jobs.uber.com/en`, `aboutwayfair.com/careers`,
  `apartmenttherapymedia.com/jobs`, `campaignlive.co.uk/careers`, `jbs.dev`,
  `wearedevelopers.com/about/4scotty`, `jobs.solarwinds.com`, and
  `workatastartup.com/jobs`.
- 5 entries deleted as duplicates. Their host now redirects into a site already
  listed: `careerrookie.com`, `de.gigajob.com`, `joblift.com`,
  `about.meta.com/en/careers`, `careers.upwork.com/jobs/search`.
- `beefrii.com` to `dead`. Loads 200 but lands on
  `filter.clickpathworks.com/filter?...` with a 4-character body. Parked domain
  on an ad redirect.
- `citizenremote.com` to `category: resource`, `apply_type: unknown`. It sells
  visas, tax help, insurance and housing to remote workers. `/jobs` is a 404.

Verified with ground truth, not inference: `validate.py` ok (2321 entries),
`dedupe.py` 0 duplicate hosts, `check.py` 2052 of 2055 live URLs OK after the
fix.

## Decisions

- A real browser is better evidence than an HTTP checker. A 403 from lychee does
  not prove a site is unusable. A 200 does not prove it is usable (see `beefrii.com`) (2026-09-29).
- Repointing beats keeping a stale URL when a site has genuinely moved host
  (uber, wayfair, Upwork, Apartment Therapy, Campaign, `JBS`, WeareDevelopers,
  SolarWinds) (2026-09-29).
- Deleting beats repointing when the redirect target is already a listed host,
  because the repo rule is one entry per host (2026-09-29).
- A site that loads but is not a job board gets a `category` change, not a
  status change (`citizenremote.com`) (2026-09-30).
- Slug and `name` follow the host, so the 8 repointed entries were renamed,
  not left with filenames that no longer match their URL (2026-09-29).

## Files touched

- `data/entries/*.yaml` had 75 edits, 8 renames, 5 deletions as listed above.
- `GUIDE.md` was regenerated, 2321 entries, 2054 live / 51 blocked / 216 dead.
- `handoffs/audit-guide-md-20260930.md` is this doc.

Audit scratch files, all outside the repo and safe to lose:

- `/tmp/rjs-browser-audit.py` covered the CloakBrowser pass over the suspect URLs.
- `/tmp/rjs-apply-fixes.py` is the script that applied the verdicts.
- `/tmp/rjs-audit-findings.json` holds 139 verdicts split into buckets.
- `/tmp/rjs-browser-audit.jsonl` holds the raw per-URL browser result.
- `/tmp/rjs-lychee.json` holds the raw lychee result for all 2067 live URLs.
- `/tmp/rjs-audit-report.md` is the written report.

## Commands / environment

- Check the whole live set: `python3 scripts/check.py` (about 2 minutes,
  needs `lychee`, installed at `/opt/homebrew/bin/lychee`).
- Rebuild outputs: `python3 scripts/build.py`.
- CloakBrowser is installed but not on `PATH`. Use
  `~/.cloakbrowser/venv/bin/python` with `from cloakbrowser import launch` or
  `launch_context`. Wrapper 0.5.10, Chromium 151 pro binary, headless only,
  Free licence allows one concurrent session.
- The previous thread ran with `/caveman ultra` and `/i-have-adhd` active.
  Repeat those if the same output style is wanted.
- GitHub still shows a 2026-09-23 `GUIDE.md` because the latest commit is `353ac4c` from
  2026-09-23 and `origin/main` matches it. Local and remote are in sync at the
  old state, with all audit work uncommitted.

## Blockers / open questions

- Commit and push have not been requested yet, so the audit is invisible to
  anyone reading the repo on GitHub.
- `stackshare.io/match` loads 200 but resolves to a personal StackShare
  profile, not the old job-matching product. Left as-is. No correct URL is known.
- `teleport.org` returned an empty 200 and then `ERR_CONNECTION_RESET`.
  `popularpays.com` returned 200 and then timed out. Both are left `live` as flaky.
- A wider category audit was offered and not answered: browser-check every
  entry filed as `job-board` (about 900) and re-categorise the ones that are
  really resources or single-company pages.

## Next steps

1. Commit the data and regenerated `GUIDE.md` together, in the repo's style,
   then push so GitHub shows the current file.
2. Decide on `stackshare.io/match`, `teleport.org`, `popularpays.com`.
3. If wanted, run the wider `job-board` category sweep.
