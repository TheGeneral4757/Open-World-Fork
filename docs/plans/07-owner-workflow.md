# 07 — How the Owner Keeps Updating the Live Game

**Rule: the owner's workflow never gets harder.** He edits the code, pushes to GitHub, and the
live game updates. That's true today and stays true after every phase. Only what happens
*after* the push changes, and that part is automatic.

## Today

```
Owner edits index.html ──push to main──▶ GitHub Actions ──▶ GitHub Pages (live)
```

## After the transition

```
                         ┌──▶ Build client (Vite) ──▶ GitHub Pages            (live client)
Owner pushes to main ──▶ CI checks (typecheck, tests)
                         └──▶ Build server image ──▶ GHCR ──▶ Boss's box pulls it  (live server)
                                                              runs DB migrations, restarts
```

- **The client** still deploys to GitHub Pages, exactly as now. The only addition is a build
  step inside the existing Action.
- **The server** auto-updates. The box checks GHCR every few minutes (Watchtower or a small
  cron script), pulls the new image, runs migrations, and restarts. It's pull-based, so
  nothing on the home network has to accept connections from GitHub.
- **If CI fails, nothing deploys**, and the live game stays on the last good version.
- **Rollback:** redeploy the previous image tag (one command for Boss), or revert the commit
  (the owner can do this himself from GitHub).

## What the owner needs to know

| He wants to… | He does… |
|---|---|
| Change UI, visuals, text, balance numbers | Edit the files and push. Same as today. |
| Test before pushing | `pnpm dev` (one command, hot reload), or keep pushing straight to main if he prefers |
| Change game rules (prices, production, combat) | Edit `packages/shared`. Client and server both pick it up. |
| Add something that stores new data | Ask Boss/Claude for a migration, or follow the `apps/server/README` recipe |
| Ship a release note | Same as today: README changelog + `RELEASE_NOTES` |

## During the transition (before the server exists)

Nothing changes for him. The fork does its work on branches. Each phase reaches his repo as a PR
he reviews and merges. Until he merges the Vite PR, he keeps editing `index.html` exactly as now.

## Guardrails (light, not bureaucratic)

- CI runs on every push, and a red build blocks deploys, not pushes.
- No branch protection forced on him unless he wants it (Q11).
- `main` = live. Boss and Claude work on branches and merge via PR, so his direct pushes never
  collide with half-finished rebuild work. This also fixes the "work got clobbered" problem.
