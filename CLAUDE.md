# music.worlds — project brain

This file is durable context for any Claude Code session working on this repo.
Read it before touching `index.html`. It reflects the code as it actually is,
not aspirationally — where the two diverge, trust the code and fix this file.

## What this is

music.worlds is a personal spatial model of music taste — not a genre
taxonomy, playlist manager, or conventional music library app. Tracks live
in "worlds" based on how they feel/behave. The atlas (the visual map of
worlds) is part of the product: classifying a track should visibly change
the world it lands in.

Core loop:

```
paste a link → Nomansland (inbox) → play → listen → classify/place → world reacts → next
```

The reward is watching the atlas evolve. No gamification — no XP, badges,
streaks, or confetti. Don't add any.

## Repo reality

- Single self-contained file: `index.html` (~4,540 lines: `<style>`, markup,
  one `<script>` block). No build step, no bundler, no package.json, no
  external JS/CSS dependencies except the YouTube IFrame API, which is
  loaded dynamically at runtime only when a YouTube source is played.
- `README.md` is a one-line stub — not a source of truth, don't rely on it.
- No framework. Plain DOM (`$ = document.getElementById`), template-literal
  HTML strings, manual re-render calls. Keep it that way (see rule 11 below).
- GitHub repo: `daasnomem/music-worlds`. Production: GitHub Pages at
  `https://daasnomem.github.io/music-worlds/`.
- Local dev must be served over `http(s)`, not `file://` — the YouTube
  IFrame API refuses to embed on `file://` (error 153). Owner runs
  `py -m http.server 8000` locally on Windows
  (`C:\Users\Leo\Desktop\musicworlds`). Never hardcode that Windows path
  into application logic.

## Data model (as implemented, `migrateTrack()` at index.html:936)

Track v4 shape:

```
{
  id,                 // string, e.g. "t_<base36 time><random>"; genId() at line 907
  title, artist,       // strings; may be blank — a URL alone is enough to exist
  primaryWorld,        // one of WORLD_IDS; defaults to 'nomansland' if unresolved
  secondaryWorlds: [],  // world ids, excludes primaryWorld and nomansland
  position: null,       // 0-100 or null; meaning is WORLD-SPECIFIC (continuum worlds only)
  energy: 50,           // 0-100, universal, independent of position and bpm
  tags: [],             // flat normalized strings (normalizeTag/normalizeTagList)
  bpm, genre, key, album, link,
  sources: [],           // [{ type: 'youtube'|'bandcamp'|'other', url, sourceId, key, meta? }]
  downloaded, notes,
  createdAt, updatedAt,
  originalWorld          // set only when an unrecognized legacy world string got dumped into Nomansland
}
```

Critical invariant, enforced by construction, never violate it:
**position ≠ energy ≠ bpm.** None is ever derived from another.

`migrateTrack()` is idempotent — running it on an already-current track
is a no-op. It is the single place old/foreign shapes get normalized
(old `mood`/`world` fields, old `intensity` field, string tag lists, etc.).
Any code that reads track data external to the app (import, legacy storage)
must go through it.

### Sources

`sources` is the generic multi-platform model (index.html:3374 onward).
`parseSourceUrl()` classifies a URL into `{ type, url, sourceId, key }`:
- `youtube` — `sourceId` is the video id, `key` is `youtube:<id>`
- `bandcamp` — `sourceId`/`key` derived from host+path (no numeric track id
  available without scraping)
- `other` — anything else; `key` is host+path+query

`key` is the de-duplication identity. `link` is never rewritten once set;
`sources` are derived from it, not the other way around.

## World system (`WORLDS` object, index.html:783)

15 worlds + Nomansland (inbox) + Strange Paradise (archived, retired —
never resurrect it as active; if legacy tracks reference it, preserve them
inert). `WORLD_IDS = Object.keys(WORLDS)`; `activeWorldIds()` excludes
archived and inbox.

| id | tagline | structure |
|---|---|---|
| kinetic-hypnosis | a state of motion | continuum: suspended → locked → accelerated |
| liquid-groove | fluid, slippery, flowing | continuum: drift → flow → rush |
| desolate | nobody is here | energy |
| badman | soundsystem attitude | energy |
| crusin | music that simply works | energy |
| memory-weather | warm, buttery, milky | energy |
| becoming | something becoming something else | energy |
| strange-attractor | controlled chaos | energy |
| ritual | a collective rhythmic state | energy |
| dream-ecology | inhabiting a landscape | regions (empty array — no subcategories implemented yet, don't invent them) |
| soft-circuits | soft machinery | energy |
| dubbed-horizons | distant, broken, far-out | energy |
| biome | alien ecosystems + cybernetics | field (axes: null — not implemented, don't force one) |
| bubblegum | cute, playful, sweet | energy |
| night-transit | forward momentum | energy |
| nomansland | awaiting classification | none (inbox) |
| strange-paradise | archived | none (archived) |

Only `continuum`-structure worlds (`kinetic-hypnosis`, `liquid-groove`)
expose a meaningful `position` axis in the UI (`isContinuum()`). Everything
else is `structure.type === 'energy'` — position stays null there by
design, not by oversight.

Each world's `core`/`boundary` text in `WORLDS` is the canonical
disambiguation — e.g. Badman (soundsystem attitude) vs. Dubbed Horizons
(distance/depth) vs. Night Transit (dub + forward motion); Ritual
(communal/ceremonial) vs. Kinetic Hypnosis (hypnosis/repetition itself).
Don't blur these boundaries when writing UI copy or classification logic.

`normalizeWorldId()` handles slugification and a small `WORLD_ALIASES` map
for legacy names (`just-cruisin` → `crusin`, `tunnel-vision` → `liquid-groove`,
etc.).

## Visual philosophy

The atlas is explicitly **not**: planets, stars, galaxies, fantasy
worldbuilding, decorative-icon cards, or conventional dashboard/SaaS UI.

Each world renders as its own **law of motion** — a physical/behavioral
animation, not an icon. These are implemented in `SPECIMENS`
(index.html:1822 onward, extended in the "SPECIMENS · PHASE 5.5" block at
~2505) and are **approved — don't casually redesign them**:

- Kinetic Hypnosis — repetition/recursion/locking
- Liquid Groove — viscous flowing ribbon
- Desolate — enormous absence / tiny distant event
- Soft Circuits — warm nodes/circuit signals
- Nomansland — unresolved holding brackets / loose points
- Ritual — synchronization / coupled oscillators
- Dream Ecology — branching organic organisms
- Strange Attractor — deterministic controlled chaos
- Dubbed Horizons — propagation + delayed fading returns
- Biome — membrane/cells/cybernetic implant
- Becoming — unresolved state reorganizing into form
- Bubblegum — soft elastic bodies
- Badman — heavy impulse + dub responses
- Night Transit — directional/parallax transit
- Memory Weather — atmospheric condensation/diffusion
- Crusin — effortless continuous trajectory

Track count can affect specimen density/complexity. Median energy can
affect world-appropriate behavior. **BPM must never be used as a proxy for
energy** — they're deliberately decoupled fields.

Aesthetic: large negative space, near-black field (`--bg: #0a0a0d`),
sparse technical shell, Georgia/serif typography, organic restrained
animation. Worlds stay spatially distinct — no melting together.

## Interaction principle

Every taxonomy action has an immediate perceptible consequence:
assign world → track moves; set position → track relocates; change energy
→ behavior changes; tag → relationships become discoverable; secondary
world → overlap becomes represented; add tracks → world grows.
Classification is feeding the atlas, not filling out a form. Preserve this
in any UI change.

## Capture & playback

A bare URL is sufficient to admit a track — metadata is enrichment
(`ENRICHMENT`, index.html:4045), never a requirement for capture.

Player adapters (index.html:3469 onward), one per source type:
- **YouTubeAdapter** — uses the official YouTube IFrame API, loaded lazily.
  Requires http(s) (fails on `file://`, error 153 → falls back to
  "external" status, opens on YouTube instead). Video tile must stay
  visible per YouTube's embed rules but can dock in any corner
  (`TILE_KEY` in localStorage remembers the corner).
- **BandcampAdapter** — no public per-track player API exists without
  scraping for a numeric track id, so it deliberately has zero inline
  playback capability (`canPlayInline: false`, etc.) and opens externally.
  This is an intentional, documented limitation, not a bug to silently
  "fix" by scraping.
- **ExternalAdapter** — fallback for everything else; opens externally.

Player state aims to preserve: play/pause, seek where supported, ±10s
where supported, volume (persisted via `PLAYER_VOL_KEY`), prev/next, and
queue context. Playback persists across navigation and classification —
classifying a track must not interrupt what's playing unless the user
acts on the currently-playing track itself.

## Storage (current, implemented)

Everything currently lives in `localStorage`. Keys (index.html:993-996):

- `musicWorlds_v3` — current library, `STORAGE_KEY`. Value shape:
  `{ app: 'music.worlds', version: 4, savedAt, tracks: [...] }` (note: the
  storage *key* says `v3`, the payload `version` field says `4` — this is
  correct as implemented, not a bug; don't "fix" the mismatch).
- `musicTracks_v2` — legacy store, `LEGACY_KEY`. Read once for migration
  on first load if `musicWorlds_v3` is absent. **Never written or deleted.**
- `musicWorlds_v3_backup` — `BACKUP_KEY`, snapshot taken before any
  "replace library" import.
- `musicWorlds_v3_pre45` — `PRE45_KEY`, one-time untouched snapshot taken
  the first time a pre-4.5 library (payload `version < 4`) is loaded,
  before sources/migration touch it. Preserves a pure pre-migration
  recovery point.
- `musicWorlds_v3_corrupt_<timestamp>` — written if JSON.parse of the
  current library ever fails, so a corrupt value is never silently lost.

Additional keys: `PLAYER_VOL_KEY` (last volume), `TILE_KEY` (video tile
docked corner).

`SEED_LIBRARY` (index.html, final block, `const SEED_LIBRARY = {...}`) is
an embedded 117-track restore catalog — the owner's last real export. It's
only offered when the live library is empty (recovery path). **Never
destroy or silently alter this data.**

JSON export/import (`IMPORT / EXPORT` section, index.html:1620) is the
portable backup path and must keep working.

## Deployment workflow

code change → test locally over `http://localhost` → inspect
`git status`/`git diff` → commit → push/merge to `main` → GitHub Pages
deploys automatically.

Never force-push, rewrite history, hard-reset, or delete recovery data
(the legacy/backup/pre45/seed keys above). Don't commit or push without
being explicitly asked to.

## Next phase (not yet implemented): Supabase cloud sync

There is currently **no Supabase code in `index.html`** — no client
script tag, no fetch calls, nothing. This section is a plan, not a
description of current behavior.

- Supabase project: `music-worlds`, URL
  `https://suwlonsqjpdobqyoeanj.supabase.co`.
- Browser-safe publishable key (fine to embed client-side):
  `sb_publishable_ZpT4w6cNNnvUCTI7Sfmorg_lGrXHJSb`. **Never** request or
  embed `service_role` keys, secret keys, or the database password. Never
  ask the owner to paste their Supabase password into Claude.
- `public.tracks` table already exists (approx. columns): `id text pk`,
  `title`, `artist`, `primary_world`, `secondary_worlds jsonb`, `position`,
  `energy` (constrained 0–100), `tags jsonb`, `bpm`, `genre`, `track_key`,
  `album`, `link`, `sources jsonb`, `downloaded`, `notes`, `created_at`,
  `updated_at`, `schema_version`. RLS enabled; authenticated CRUD policies
  exist but are currently global (single intended user) — add
  ownership/`user_id` isolation before allowing a second user.

Target architecture once built: GitHub Pages = app code · Supabase =
canonical cloud library · localStorage = local cache/fallback · JSON
export = portable backup.

Desired behavior: sign in → same library everywhere → add/classify/edit
→ optimistic UI update + write to Supabase → on another device, same
state.

Migration must be conservative:
- If cloud is empty and local has tracks → offer an explicit "move this
  library to cloud" action showing the track count; upsert preserving
  IDs; verify; **never delete the local copy**.
- Once migrated, Supabase is canonical; localStorage stays cache/fallback.
- Writes are optimistic: update UI + local immediately, attempt cloud
  write, and on failure keep local data with a visible unsynced/retry
  state. **Never treat a failed cloud fetch as an empty library.**
- Never overwrite a non-empty cloud library from stale local state.
- Build explicit `trackToRow()` / `rowToTrack()` converters. Field
  mappings: `primaryWorld ↔ primary_world`, `secondaryWorlds ↔
  secondary_worlds`, `key ↔ track_key`, `createdAt ↔ created_at`,
  `updatedAt ↔ updated_at`, `schemaVersion ↔ schema_version`.
- Add an explicit `DATA_SCHEMA_VERSION` + `migrateTrack()`-style strategy
  for future row/track shape changes rather than ad hoc mutation — the
  existing `migrateTrack()` at index.html:936 is the pattern to extend,
  not replace.
- Keep track IDs stable across the whole pipeline.
- Do not auto-upload the embedded 117-track `SEED_LIBRARY` if the cloud
  already has tracks.
- Realtime sync is optional for v1; refresh-on-focus is an acceptable
  fallback.

## Development rules

1. Inspect current code before changing architecture.
2. Preserve working functionality.
3. Make incremental changes.
4. Never silently destroy library data.
5. Never overwrite a non-empty cloud library from stale local state.
6. Never treat a network failure as "zero tracks."
7. Preserve stable track IDs.
8. Preserve YouTube playback behavior.
9. Preserve the approved atlas animations (`SPECIMENS`) — don't redesign
   them casually.
10. Do not turn the UI into generic SaaS/dashboard software.
11. Avoid unnecessary dependencies or framework rewrites — this is
    intentionally a single dependency-free HTML file.
12. Do not deploy/push automatically unless explicitly requested.
13. Before substantial changes, explain intended files/areas affected.
14. After changes, report: files changed, checks performed, `git status`,
    and a suggested commit command. Don't commit/push unless asked.
