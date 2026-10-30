---
name: cauldron
description: Generate images and work with a Cauldron studio from the terminal via the Cauldron MCP. Use when asked to generate/create an image, make a visual, search the Cauldron Library (including by describing what's in an image, filtering by tag, status, rating, person, collection or date, or finding look-alikes of an asset), look up a brand kit or a template, list or compare image models, show a recent generation, generate from a reference image, upscale/extend/resize an image or remove its background, upload a file or logo into the Library, change an existing asset's tags, title or description, list or delete the studio's tags, create collections and file assets into them, save or run a saved Library view, share a collection, project or asset by link (create, list, change or turn off share links), import or migrate a library (Dropbox, pics.io, or files on disk), read or change brand or studio settings (name, Slack channel, prompt prefix/suffix, banned terms), polish a prompt, or when Cauldron/the studio is mentioned. Also use before any generate_image call to get model choice, brand scoping, and spend rules right.
---

# Cauldron

The Cauldron MCP exposes a Cauldron studio as 48 tools, plus 5 saved-view
tools when the server has saved views turned on and 5 pics.io tools when it
has pics.io import turned on (58 in all). Everything is scoped to the (user,
studio) pair the connection was made for — no argument reaches another
studio's data.

## Setup

If the tools below aren't available, the MCP server isn't connected. This skill
can be installed without it, so walk the user through connecting instead of
guessing at tool calls.

1. **Connect.** Pick whichever fits the user's client:
   - Claude Code plugin: `/plugin marketplace add cauldronstudio/skills`, then
     `/plugin install cauldron@cauldron`. Nothing to fill in.
   - Claude Code without the plugin:
     `claude mcp add --transport http cauldron https://app.cauldron.studio/api/mcp`
   - Any other MCP client: a streamable HTTP server at
     `https://app.cauldron.studio/api/mcp`, no header.
2. **Sign in.** In Claude Code: `/mcp` → **cauldron** → **Authenticate**. A
   browser opens Cauldron; the user signs in, picks the studio, and clicks
   **Allow**. One connection = one studio; to switch, `/mcp` → **cauldron** →
   **Clear authentication**, then authenticate again and pick the other one.
3. **Clients without MCP sign-in** use a key instead: Cauldron → **Profile →
   Connected AI Tools → Create key** (starts with `oc_`, shown once, pinned to
   one studio), sent as `Authorization: Bearer oc_…`.

Full guide: https://docs.cauldron.studio/guides/connected-ai-tools/

## The tools

**Reads** (free, no spend): `get_context`, `search`, `list_brands`, `get_brand`,
`list_models`, `get_model_details`, `get_recent_generations`, `get_template`,
`show_generation`. `get_template` takes `templateIdOrSlug` and returns a saved
or built-in template's name, blurb, steps, visibility, brand and slug; find
one with `search` scope `templates`. (`get_brew`, with `brewIdOrSlug`, is a
deprecated alias kept for one release: brews became templates with the same
ids and slugs. Don't use it in new work.)

**Finding assets.** `search` takes a `scope`: `library` (any asset the person
can see), `references` (uploaded references), `collections` (projects too,
flagged `isProject: true`), `templates` or `loras`. For `library` and
`references` it matches by meaning as well as by words, so pass a plain
description of what's in the image: `"ceramic candles in earthy colors"` finds
"Four ceramic candle vessels in terracotta…" though its name never says
"earthy". It covers what the person can see in the Library: their own assets
plus teammates' non-private ones on brands they belong to (owners and admins
see the whole studio). `filters.brandId` narrows to one brand.
`filters.sort: "recent"` orders newest first, and with no `query` lists the
newest additions (uploads and imports included): use it for "what did I add
most recently". `filters.source` narrows to `generated`, `uploaded` or
`imported` (one or a list): "the latest generations" = `sort: "recent"` +
`source: "generated"`, no query. `filters` also takes the Library's own
filters, the same fields a saved view holds: `tags` (names or ids) with
`tag_match` `any`|`all`, `status[]` (`draft`, `in_review`, `approved`,
`rejected`, `retired`), `rating[]` (1–5), `made_by` (a person's name or
email), `collections` (names or ids), `uploaded_from`/`uploaded_to`
(YYYY-MM-DD, `by_capture_date: true` reads them as capture dates), `lens`
(`recent` = last 7 days, `unfiled` = in no collection), `media_type`,
`format[]`, `orientation`. Names resolve inside the brand; an unknown or
ambiguous name fails with `candidates`, so pick one and retry rather than
guessing. With filters and no query it lists the newest matches: "approved
hero shots since September" = `{ status: ["approved"], tags: ["hero"],
uploaded_from: "2026-09-01" }`. Each hit carries `brand`, `createdAt`, a
`snippet`, an AI-written `caption` of what is visible (when one exists; it
may be wrong), `match` (`lexical`, `semantic` or `both`) and, when found by
meaning, `similarity` (cosine, higher is closer). `mode` is `hybrid`
(default), `keyword` (exact words only) or `semantic`. The last word also
matches as a prefix, so a half-typed "cand" finds "candle".

Weak matches by meaning are left out, so an empty result with the note
"Nothing matched closely enough" is a real answer: "do I have a red sports
car shot?" → no. Try broader words or drop `filters.brandId` once, then
say so. The exception is `semanticUnavailable: true`: matching by meaning
was down, so hybrid hits are keyword-only and a `semantic` search ran on
nothing. Then say search is degraded and retry shortly (or use `keyword`);
never report "nothing found".

"More like this one": `search` with `similar_to: { asset_id }` ranks by how
that asset looks (`query` is then optional; give both and they rank
together). `notIndexed: true` means that asset hasn't been indexed yet; try
again later rather than reading it as "no matches". `show_generation` with an
`asset_id` shows any asset `search` returned.

**Transforms** — act on an `asset_id` you already own and save the result back
to the Library as a new asset: `upscale_image`, `remove_background`,
`extend_image` (each a small provider cost), and `resize_image` (local, free).
Re-running one returns the existing output instead of spending again.

**Upload** (free, no spend) — `upload_asset` puts a file INTO a brand's Library
so the tools above have something to act on: pass `url` (an https URL the
server fetches; public hosts only) or `data_base64` + `file_name` (a local
file, up to 10 MB), plus `brand_id` and optional `tags`. It returns the new
`asset_id`. The same URL into the same brand twice returns the existing asset
(`reused: true`). This is how "resize our logo to 1200×1200" works end to end:
`upload_asset` → `resize_image`.

**Edit an asset** (free, no spend) — `update_asset` changes an asset that is
already in the Library: its `tags`, `title` or `description`. Pass `asset_id`
(from `search`, `get_recent_generations` or an upload) and only what changes:

- `tags: { add, remove }` edits the current set; `tags: { set }` replaces it
  (`set: []` removes every tag). `set` can't be combined with `add`/`remove`.
- Names match existing tags ignoring case and punctuation ("Spring Sale" and
  "spring-sale" are one tag), the brand's own tag first. A new name becomes a
  studio tag. Removing a tag the asset doesn't carry is ignored.
- `title: null` clears the title; a blank or `null` `description` clears it.
- It returns the asset's `brand`, `title`, `description` and `tags` after the
  change, so you don't need a read to confirm. Same input twice changes
  nothing.

"Tag the spring shots `outdoor` and drop `needs-retouch`": `search` with
`query: "spring"`, show the hits, then one call per asset the person confirms:

```json
{ "asset_id": "…", "tags": { "add": ["outdoor"], "remove": ["needs-retouch"] } }
```

Who can: the people who can edit the asset in the Library (creators and brand
managers on its brand, studio owners and admins). Asset-at-a-time; there is no
bulk form, so for many assets loop and report what changed.

**Tags** (free, no spend). `list_tags` lists the studio's tags the person can
see, as on the Library's Tags page: each with `id`, `name`, `scope`
(`"studio"` or the brand's name), `brandId`, `category`, `usageCount` (how many
assets carry it) and `needsReview` (an AI suggestion nobody has reviewed yet).
Owners and admins see every tag; anyone else sees the tags on their brands plus
studio-wide ones. Narrow it with `brand_id`, `query` (matches name, slug or
category) or `include_review_queue: false`. Use it to answer "what tags do we
have", to spot near-duplicates, and to get a `tag_id`.

`delete_tag` takes `{ "tag_id": "…" }` and deletes that tag for good. It is
irreversible and detaches the tag from every asset that carries it, so before
calling it show the person the tag's name, scope and `usageCount` and get a
clear yes. Call it once per tag, never in a loop the person hasn't approved.
It returns `{ ok: true, id, name, detachedFrom }`, where `detachedFrom` is the
number of assets it was removed from. To take a tag off just a few assets, use
`update_asset` with `tags.remove` instead.

Who can delete: brand managers for their brand's tags, studio owners and admins
for every tag (a studio-wide tag is theirs only). Guests never.

**Collections** (free, no spend). Collections belong to a brand, as in the
Library's collection tree. Before tagging, titling or filing a brand's assets,
read `brand_instructions` from `get_brand` and follow it.

- `list_collections` lists the ones the person can see: each with `id`,
  `name`, `description`, `brand_id` and `asset_count`. Narrow with `brand_id`.
  Use it to get a `collection_id` for the tools below.
- `create_collection` takes `brand_id`, `name` (up to 80 characters) and an
  optional `description` (up to 500). A collection with the same name on that
  brand is returned instead of a duplicate (`created: false`).
- `update_collection` takes `collection_id` and `name` and/or `description`
  (`null` clears it). Its brand, place in the tree and URL slug can't be
  changed here. Returns the collection after the change.
- `add_to_collection` / `remove_from_collection` take `collection_id` and
  `asset_ids` (from `search` or `get_recent_generations`, up to 100 per call).
  Both are safe to repeat: add returns `added`, `already_present`, `not_found`
  (not in this studio, or another brand's asset where the collection only takes
  its own) and `forbidden`; remove returns `removed`, `not_present`,
  `not_found` and `forbidden`. Filing a private draft shares it with the brand.
  Removing never deletes the asset.
- `delete_collection` takes `collection_id` and deletes the collection and any
  sub-collections for good; the assets stay in the Library. Confirm with the
  person first. Returns `deleted_ids`.

Who can: a creator on the brand, or a studio owner or admin. Guests and other
members get `forbidden`.

**Saved views** (free, no spend; listed only when the server has saved views
turned on). A view is a named Library filter set ("Approved hero shots") that
only the person who saved it sees.

- `list_views` lists them: id, name, brand, author, `createdVia` (`library`,
  `assistant` or `mcp`), the filters in plain words and a Library link.
  `brand_id` (id, name or slug) narrows to one brand; omit it for every brand.
- `apply_view` runs one (`view_id`, optional `limit`): returns the view,
  `total` (the count the Library shows), the newest matches shaped like
  `search` hits, and `libraryUrl`.
- `save_view` takes `name` and `filters` (the same fields `search` takes:
  `tags`, `status`, `source`, `rating`, `made_by`, `collections`,
  `uploaded_from`/`uploaded_to`, `lens`, `media_type`, `format`,
  `orientation`, `query`); `brand_id` picks a brand, and omitting it saves on
  All brands. A tag, collection or person that matches nothing or several
  fails with `candidates`: ask which one. Returns the view, its match count
  and a Library link. `from_page: true` only works inside the app's Assistant
  (it reads the open Library page); over MCP pass `filters`.
- `update_view` takes `view_id` plus a new `name`, new `filters` (replacing
  all the old ones) or `as_collection: true`/`false` to show it (or stop
  showing it) as a smart collection on the Collections page.
- `delete_view` takes `view_id`. Only the view goes; the files stay. Confirm
  first.

**Imports** (free, no spend). Every import runs on Cauldron's servers in the
background and returns an import id at once. Files land as drafts. Use the
`migrate` skill for the Dropbox flow; any import starts only after the person
confirms what it will do.

- **Dropbox**: `dropbox_status`, `dropbox_browse`, `dropbox_scan`,
  `dropbox_get_scan`, `dropbox_preview`, `dropbox_import`.
- **pics.io** (listed only when the server has pics.io import on):
  `picsio_status`, `picsio_scan`, `picsio_get_scan`, `picsio_preview`,
  `picsio_import`. See "pics.io" below.
- **Files on disk** (a migration with no source API, or any large batch):
  `presign_uploads` → PUT the bytes → `import_assets`. See "Files on disk" below.
- **Any import**, whichever tool started it: `get_import_status` (counts,
  failed files with reasons, where they landed; poll every 20 to 30 seconds
  until `finished` is true), `resume_import` (retries only failed and waiting
  files, or resumes a `paused` import after the person reconnects),
  `cancel_import` (files already imported stay; ask the person first). They
  take `import_id`.

**Share links** (free, no spend) — what Cauldron's Settings → Shared links
shows. A share link publishes a collection, project or one asset to anyone
holding the URL, no Cauldron account needed; drafts never show. Only create
one when the person asks to share, never as a convenience, and hand the URL to
them rather than posting it anywhere. You need a creator or manager role on
the brand for all of these.

- `list_shares` takes `brand_id` (from `list_brands`) and lists the brand's
  links newest first: `shareId`, what it shows (`kind` `collection`, `project`
  or `asset`, name, `targetId`), URL, approved-only, downloads, password
  on/off, note, who made it, open count and expiry (expired links stay
  listed). `mine: true` narrows to links you made; `collection_id` or
  `asset_id` narrows to that one target's link.
- `get_share` reads one link by `share_id`.
- `create_share` takes exactly one of `collection_id` (a project id works too)
  or `asset_id`, plus options: `expires_at` (ISO 8601, at most 365 days out;
  `null` never expires; a new link without it ends after 30 days, sooner if an
  asset's license ends first), `password` (4–200 characters, people must type
  it, never returned; `null` for none), `allow_download` (lets people download
  approved work), `visibility` (`approved` shows approved work only; `status`
  shows everything but drafts, each with its status) and `note` (shown under
  the title). On a brand that allows mature content you must decide: pass a
  `password`, or `password: null` to publish without one. If the target already
  has a live link you get that link back (`created: false`) with only the
  settings you passed changed. Returns `shareId` and `url`.
- `update_share` takes `share_id` and at least one of the options above; the
  URL stays the same and omitted settings are kept.
- `revoke_share` takes `share_id` and turns the link off for good: everyone
  holding the URL loses access and it can't be turned back on (sharing the
  same thing again makes a new URL). Confirm with the person first. Returns
  what the link showed and `wasLive`.
- `list_shared_collection` with a `collection_id` (from `search` scope
  `collections`) lists every asset on a collection's share link with a
  `previewUrl` (a small JPEG that works in a Google Sheets `=IMAGE()`) and a
  `downloadUrl` (the original; null until the asset is approved). The links
  don't expire; they stop when the share link is turned off or reset. The
  collection must already be shared and you must be able to manage it.

**Settings** (free, no spend) — `get_settings` returns your role, the brands
you can edit (name, description, website, location, self-approval, Slack
channel, video, community flags, prompt prefix/suffix, banned terms), whether
Slack is connected, and the studio settings (owners and admins only). Pass
`brand_id` for one brand. `update_brand_settings` takes `brand_id` plus only the
fields to change; slug, the mature-content switch (`allow_nsfw`), colors and
the rest of the guide are refused.
`update_studio_settings` takes `name` and/or `slack_channel_id`, always for the
key's own studio.

- Who can: studio owners and admins edit everything; a member edits only
  brands they manage, and `update_studio_settings` is owner/admin only. Guests
  never can.
- Slack connect and disconnect stay in the browser. These tools never touch
  the Slack token. If `get_settings` says Slack isn't connected, relay its
  `how_to_connect` instead of retrying.
- A channel is an ID (`C…` or `G…`), not a `#name`, and the bot must be in it
  first (`/invite @Cauldron`). A changed channel gets a test post; if that
  fails nothing is saved. `slack_channel_id: null` on a brand falls back to the
  studio default; on the studio it turns updates off.

**Spend**: `enhance_prompt` (small per-call model cost), `generate_image`
(provider cost per image).

## pics.io

A pics.io connection belongs to one person and one brand. You can't connect it:
the person opens the `connectUrl` the tools return (the brand's
**Integrations** page in Cauldron) and pastes a personal access token they make
in pics.io under **avatar → My account → Security**. No pics.io admin needed.
Never ask for the token in chat. Guests can't import.

1. `picsio_status` with `brand_id` (from `list_brands`). `connected:
   false` or `needsReconnect: true`: relay `connectUrl`, wait, check again.
2. `picsio_scan` with `brand_id`, and `roots` (pics.io collection ids) to read
   only part of the library; omit for all of it. Returns `scanId`.
3. `picsio_get_scan` with `scan_id`, every few seconds while `status` is
   `running`. It shows one level of the collection tree at a time with rolled-up
   `files`, `bytes` and `cantImport`; pass `collection_id` to open one,
   `offset`/`limit` (max 200) to page. Show the collections; let the person pick.
4. `picsio_preview` with `scan_id`, `include` (collection ids; `"root"` is the
   whole library), optional `exclude`, `keep_duplicates`, `include_lightboards`
   (turns the person's private lightboards into collections the whole brand
   sees: ask first) and `flatten_depth` (levels kept, 1 to 8). Copies nothing.
   Show the totals, where files land, what comes with them, duplicates and
   can't-import reasons. `canImport: false`: relay `blocked` and change the
   selection.
5. After a yes, `picsio_import` with the same selection and exactly one of
   `collection_id` (from the preview's `earlierImports`) or
   `new_collection_name` (`defaultNewCollectionName` is free). `keywords`
   (default true) brings pics.io keywords as tags; `extra_tags` adds up to 20 of
   your own. Returns `importId`.
6. `get_import_status` until `finished`. Collections, keywords, description,
   rating, details and comments come along; older versions don't. Nothing in
   pics.io changes.

One import copies up to 5,000 files from a selection of up to 20,000; for more,
import in several runs. Two pics.io imports can run at once per person.

If the person's pics.io plan has no API access, use files on disk instead.

## Files on disk

For a library that's already on this machine (an export from a service with no
API, a drive of shoots), or anything too big for `upload_asset`.

1. `presign_uploads` with `brand_id` (from `list_brands`) and `files`: up to
   1,000 of `{ file_name, size, mime_type? }`, `size` exact in bytes. Returns
   `uploads` (each with a `key`) and `rejected` (unsupported type or over the
   size limit, with a reason; skip those, the rest still go).
2. PUT each file's bytes to its `put_url` with `Content-Type` set to its
   `content_type` exactly. Files over 64 MB come back with `multipart` instead:
   PUT bytes `[start, end)` to each part's `url` (no Content-Type), retry a
   failed part on its own, keep each response's `ETag`, then
   `complete_multipart_upload` with `key`, `upload_id` and every `part_number` +
   `etag`. Giving up on one: `abort_multipart_upload` with `key` and
   `upload_id`. URLs expire after 1 hour (6 for video and multipart parts).
3. `import_assets` with the same `brand_id`, a `run_label` (the collection
   everything is filed under, e.g. "From pics.io") and `files`: up to 1,000 of
   - `key`, `file_name`, and `source_ref`: a stable id for the file at its
     source (e.g. `"picsio:<asset id>"`). A `source_ref` already imported into
     the brand is linked, never copied again, so re-running is safe. Each may
     appear once per call.
   - optional `collections`: paths under `run_label`, e.g.
     `[["Campaigns","Autumn"],["Selects"]]`; the file is filed in each.
   - optional `tags` (up to 50), `lightboards`, and `extras`:
     `{ origin: "picsio", description?, rating? (1–5), capturedAt?, details?:
     [{label, value}], comments?: [{ref, author, at, text}] }`. Extras only fill
     empty fields; a person's own edits win.

   Optional `flatten_below_depth` (levels kept, 1 to 8) and
   `include_lightboards` (default false). Returns `import_id`, `files`,
   `already_imported` and `collection`.

   `source` is `"agent"` (the default) or `"picsio"`. Pass `"picsio"` only
   when the files came from a pics.io export, so `source_ref`
   `"picsio:<asset id>"` matches in-app pics.io imports into the same brand and
   nothing is copied twice. Otherwise leave it out. Duplicates are only found
   within one source, so use the same value on every call of a run.
4. For more than 1,000 files, call again with the same `run_label` and
   `source`: the batches join the same collection. A label only adds to a
   collection an earlier run from the same source made. At most two of your imports run at once; on
   `imports_running`, poll `get_import_status` and send the batch again.

Nothing is in the Library until `import_assets` runs. A PUT on its own does
nothing.

## Rules that actually matter

### 1. `generate_image` spends real money
One call bills 1–4 images at provider rates against the studio's account. The
permission prompt you see IS the spend confirmation — don't add ceremony on top
of it, but don't fan out counts speculatively either. Default `count` to 1
unless variations were asked for.

### 2. Every write names a brand: call `list_brands` first
Everything you generate, upload or import lands in one brand's Library, and
that brand's kit (prompt prefix/suffix, banned terms, palette) applies. So
before `generate_image`, `upload_asset`, `presign_uploads`, `import_assets`,
`dropbox_preview`/`dropbox_import` or `picsio_status`/`picsio_scan`, call
`list_brands` and pass the right one as `brand_id`. `get_brand` shows the kit
if you need it before writing the prompt.

- **Exactly one brand you can add to:** you may leave `brand_id` out and that
  brand is used. Passing it anyway is never wrong.
- **More than one:** leaving it out is refused with `brand_required`, and the
  refusal's `brands` lists them (id and name). Ask the person which brand the
  work is for; don't guess from the list.
- **`"personal"`** is not a brand id and is refused with `brand_required`.
  There is no fallback brand.
- **No brand in the studio yet:** every write is refused with `no_brand`
  (status 409). Tell the person to open the app (`appUrl`) and create their
  first brand, then try again. You can't create one for them.

### 3. Choose the model, don't accept the default
`list_models` returns a browse view (id, provider, cost, `bestFor[]`).
`get_model_details` gives strengths, tradeoffs, and `whenToUse` — call it when
two models have overlapping `bestFor` tags. Cache both within a turn.

Only **sync image models** work here. Video and async models are refused with a
message listing what's usable; that's a v1 boundary, not a bug. There is no
polling flow — an MCP call must return the finished asset inline.

### 4. Reference images beat describing the undescribable
`generate_image` takes `image_refs` — up to 4 entries, each a Library `asset_id`
or an `https` URL. Reach for it when the thing you need is easier to point at
than to write: keep this character, match this product, work in this look.

- Get ids from `search`, `get_recent_generations`, or a previous
  `generate_image` result. Ids are re-signed server-side, so a stale URL from an
  earlier turn is never a problem — pass the id, not the URL you were handed.
- The model has to support it. `get_model_details` reports `supportsImageInput`;
  a model without it **refuses** rather than generating something that ignored
  your reference. Check before you call, and switch models rather than dropping
  the reference.
- A brand kit's own anchor references apply only when you pass none. Yours win.
- References must belong to the key's studio. An id from elsewhere comes back
  `not_found`.

### 5. Write the prompt like a prompt
Subject + setting, style/medium, lighting, mood, composition, palette. A
one-noun prompt gets one-noun output. For a rough idea the user handed you,
`enhance_prompt` first — it returns richer text and does not generate.

### 6. Returned URLs expire
`generate_image` returns short-lived presigned URLs. If one 403s later, re-fetch
with `show_generation` using the asset or generation id rather than regenerating.
The asset itself is permanent in the Library — same `generations` + `assets` rows
as an in-app generation.

### 7. Don't call `get_context` reflexively
It's for orienting when you genuinely lack workspace/brand context. Greetings,
straightforward generation requests, and lookups don't need it.

## Naming convention

Refer to models by **capability**, never by backend infrastructure — the studio
deliberately hides GPU/serving details from users, and these outputs often get
relayed to clients. "Fast photoreal model," not the pod or provider stack behind
it.

## Errors you'll actually hit

| Error | Meaning |
|---|---|
| `billing_inactive` | Studio billing is off. Every spend tool refuses; reads and `resize_image` still work. |
| `brand_required` (400) | No `brand_id` and you can add to more than one brand (pick from `brands` in the refusal, asking the person), or you passed `"personal"`. |
| `no_brand` (409) | The studio has no brand yet. The person opens the app (`appUrl`) and creates their first brand. |
| `brand_not_found` (404) | That `brand_id` isn't a brand in this studio. Call `list_brands`. |
| `forbidden` (403, on a write) | You don't have Creator access on that brand, or on any brand when `brand_id` was left out. A brand manager can grant it. |
| `unauthorized` | Key revoked, or you were removed from the studio. Mint a new one. |
| daily limit | Same per-user generation cap as the web app. Resets daily. |
| async/video refusal | Model isn't sync-image. The message lists valid ones. |
| `does not accept reference images` | You passed `image_refs` to a model without `supportsImageInput`. Pick another; don't drop the reference. |
| `image_ref … not_found` | The asset isn't in this studio (or the id is wrong). |
| `upload_asset` refusal | Plain words: wrong file type (name it with a real extension), over the size limit, no Creator access on the brand, or the URL couldn't be fetched (https on a public host only; private and local addresses are blocked). Host larger files and pass `url`. |
| `update_asset` refusal | "Asset not found": wrong id, or the asset is in another studio; find it with `search`. "can't edit": no creator or manager role on the asset's brand. A tag name with no letter or digit is refused before anything is written. |
| `delete_tag` refusal | "Not found": wrong id, or the tag is in another studio; get the id from `list_tags`. "Forbidden": no manager role on the tag's brand (or not a studio admin, for a studio-wide tag). Nothing was deleted. |
| collection refusal | "Collection not found": wrong id, or another studio's; call `list_collections`. "forbidden": no creator role on that brand (or not a studio owner or admin); guests never. "Collections aren't available on this server": the Library is off on this server. |
| share refusal | "Pass exactly one of collection_id or asset_id" on `create_share`; `expires_at` in the past or over 365 days out; a password outside 4–200 characters; no creator or manager role on the brand; "not found" for a `share_id` from another studio or a link already turned off. Nothing was changed. |
| view refusal | Views are off on this server (the tools aren't listed either); `view_id` not yours or not found (call `list_views`); a tag, collection or person name that matched nothing or several (`candidates` lists them: ask, then retry); `from_page` over MCP ("There's no Library page in view"): pass `filters`. |
| `get_template` not found | Wrong id or slug, or a template you can't read. Find it with `search` scope `templates`. |
| import refusals | Plain sentences: no Creator access on the brand, a guest account, the hourly cap on starting imports (it says how long to wait), out of storage, a selection too big. Relay them; don't work around them. |
| `forbidden` (settings) | You can't edit that brand or the studio: you're a guest, or a member who doesn't manage it. |
| `slack: …` | The test post to the new channel failed, usually because the bot isn't invited or the ID is wrong. Nothing was saved. |
