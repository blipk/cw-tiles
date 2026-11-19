# cw-tiles

Map tile pyramids for [Cosmic Weather](https://github.com/blipk/cosmic-weather),
served straight from this repo by [jsDelivr](https://www.jsdelivr.com).

Nothing here is hand-authored. Every file is build output, produced by the
generators in the app repo (`scripts/assets/`, documented in
`developer/BUILD.md`). Do not edit tiles in place — rebuild and re-tag.

## Layout

Pyramids sit at the repo **root**, so a tile's URL is its path:

```
https://cdn.jsdelivr.net/gh/blipk/cw-tiles@v1/day/3/5_2.webp
                                         └tag┘ └── path in this repo ──┘
```

| directory | what it is |
| --- | --- |
| `day/`, `day-01/` … `day-12/` | surface imagery — a base plate and one per month |
| `night/` | night lights |
| `height/` | terrain elevation (Terrarium-encoded) |
| `worldcover/` | land-cover classes |
| `lightpollution/` | estimated night-sky brightness |

Imagery is `.webp`; the DATA pyramids (`height/`, `lightpollution/`) are `.png`,
because lossy WebP would quantise numbers that are values rather than colours.

The pyramids are **sparse on purpose**. A tile that is nothing but ocean or empty
sky is never written, and the app treats a 404 as "the global plane already draws
this" rather than as an error.

## Releases are TAGS, and the tag is the version

The app points at one tag, baked in at build time:

```
VITE_TILES_BASE_URL=https://cdn.jsdelivr.net/gh/blipk/cw-tiles@v1
```

That tag is the ONLY version marker anywhere in the system. Tile paths carry no
version of their own, so a rebuilt tile keeps its old URL — which means a new tag
is what tells every cache that anything changed.

**To publish a rebuild:**

```bash
cd packages/web/public/tiles         # this repo, nested in the app's working tree
git add -A
git commit -m "Rebuild <what changed>"
git tag v2
git push origin main v2
```

Then bump the app's `VITE_TILES_BASE_URL` to `…/cw-tiles@v2` and redeploy.
Old tags keep serving, so a rollback is pointing the base back at `@v1`.

### ⚠️ Never overwrite a published tag

Move a tag that is already in use and caches disagree with each other: jsDelivr
holds a tagged path for a year, browsers hold their own copy, and the app keeps
cached tiles in IndexedDB for 30 days. A viewer then gets a **mix of two builds**
with nothing on screen to say so — a map that is half one dataset and half
another, which reads as a rendering bug rather than a stale cache.

For the same reason, point the app at a **tag**, never at a branch.

### Pushing is chunked

The full set is several GB and GitHub caps a single **HTTPS** push at 2 GB, so
push one pyramid (or one level) at a time. Over **SSH** there is no per-push cap.
If a push fails partway, re-run just that `git push` — only that commit's objects
re-send. Full notes in the app repo's `developer/DEPLOY-CF+JSD.md`.

## Sources and terms

These are derived products. Each pyramid inherits the terms of the data it was
built from — chiefly NASA Blue Marble / Black Marble, ESA WorldCover, and, for
`lightpollution/`, an estimate convolved from VIIRS annual nighttime lights. It
is an estimate of sky brightness, not a measurement of it. See `LICENCE.md` and
`developer/BUILD.md` in the app repo for the per-source attribution.
