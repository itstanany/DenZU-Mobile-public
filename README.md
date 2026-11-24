# DenZU — public content host

DenZU is the clinic companion for dental students, interns and freelance dentists — your clinics,
rotations and shifts in one calm place.

This repository holds **only the public content the app fetches at runtime**: the landing page, the
privacy policy, the terms of use, and the versioned clinic-schema payload. It contains no application
source and no user data.

🌐 Landing page: https://itstanany.github.io/DenZU-Mobile-public/denzu/

## Layout

| Committed at | Cloudflare Worker | GitHub Pages | Contents |
|---|---|---|---|
| `denzu/content/` | `/content/` | `/DenZU-Mobile-public/denzu/content/` | `manifest.json` + the immutable payload `content-vN.json` |
| `denzu/privacy/` | `/privacy/` | `/DenZU-Mobile-public/denzu/privacy/` | Privacy policy |
| `denzu/terms/` | `/terms/` | `/DenZU-Mobile-public/denzu/terms/` | Terms of use |
| `denzu/` | `/` | `/DenZU-Mobile-public/denzu/` | Landing page and the appointment PDF |

The two hosts publish different roots for the same committed file — the Worker's asset root is
`denzu/`, Pages publishes the repository root — so a payload at `denzu/content/` answers at
`/content/` on the Worker and `/denzu/content/` on Pages. The app therefore holds **two full
directory bases**, not one hostname, and tries the Worker first.

## Cloudflare Worker

`wrangler.toml` describes it: static assets only, asset root `./denzu`.

- Name: `denzu-mobile-public` → `https://denzu-mobile-public.ahmadalymohamad.workers.dev`
- Preview: `*-denzu-mobile-public.ahmadalymohamad.workers.dev`
- Root directory `/`, build command **empty**, deploy command `wrangler deploy`

The name *is* the host: the app pins it in `ContentHosts.WORKER`, so renaming the Worker changes an
app constant and needs a release. Keep the name, or move both together.

Probe that proves the asset root, the path and the content type at once:

```bash
curl -sI https://denzu-mobile-public.ahmadalymohamad.workers.dev/content/manifest.json
```

## GitHub Pages

Pages serves the same commit as the fallback host, at
`https://itstanany.github.io/DenZU-Mobile-public/denzu/…`. `.nojekyll` is committed so the tree is
served as-is. Pages sends an `etag`, so the app's conditional request answers 304 when this base is in
use; the Worker sends none.

## Publishing content

Content is **data, not code**: the app downloads `manifest.json`, verifies the payload against the
`bytes`/`sha256` it names, then stages it. Publishes are made from the private app repository with
`scripts/publish-content.sh`, which validates the payload with the app's own seed gate, prints the
diff against what is live, and commits `content-vN.json` + `manifest.json` here.

Two rules this host must not break:

- **`content-vN.json` is immutable.** Three caches key on the file name, so a correction is `N+1`,
  never an overwrite.
- **`manifest.json` carries no URL.** The host list is compiled into the app, so a moved host is a
  release, not a manifest field.

## Legal pages

`denzu/privacy/index.html` and `denzu/terms/index.html` are generated from `docs/privacy-policy.md`
and `docs/terms-of-use.md` in the private app repository, which are the source of truth. Both must
stay reachable from inside the app: Apple 5.1.1 requires the policy in the app, and the terms name
the policy as part of themselves.
