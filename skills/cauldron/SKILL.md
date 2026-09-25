---
name: cauldron
description: Generate images and work with a Cauldron studio from the terminal via the Cauldron MCP. Use when asked to generate/create an image, make a visual, search the Cauldron Library, look up a brand kit, list or compare image models, show a recent generation, generate from a reference image, upscale/extend/resize an image or remove its background, polish a prompt, or when Cauldron/the studio is mentioned. Also use before any generate_image call to get model choice, brand scoping, and spend rules right.
---

# Cauldron

The Cauldron MCP exposes a Cauldron studio as 15 tools. Everything is
scoped to the (user, studio) pair the API key was minted for — no argument
reaches another studio's data.

## Setup

If the tools below aren't available, the MCP server isn't connected. This skill
can be installed without it, so walk the user through connecting instead of
guessing at tool calls.

1. **Get a key.** Cauldron → **Profile → Connected AI Tools → Create key**. It
   starts with `oc_` and is shown once. One key = one studio; working across
   studios means a key per studio.
2. **Connect.** Pick whichever fits the user's client:
   - Claude Code plugin: `/plugin marketplace add cauldronstudio/skills`, then
     `/plugin install cauldron@cauldron`. It prompts for the key and stores it
     in the OS keychain.
   - Claude Code without the plugin:
     `claude mcp add --transport http cauldron https://app.cauldron.studio/api/mcp --header "Authorization: Bearer oc_…"`
   - Any other MCP client: a streamable HTTP server at
     `https://app.cauldron.studio/api/mcp` with the header
     `Authorization: Bearer oc_…`.
3. **Restart or reconnect** the client so it picks up the tool list.

Full guide: https://docs.cauldron.studio/guides/connected-ai-tools/

## The tools

**Reads** (free, no spend): `get_context`, `search`, `list_brands`, `get_brand`,
`list_models`, `get_model_details`, `get_recent_generations`, `get_brew`,
`show_generation`.

**Transforms** — act on an `asset_id` you already own and save the result back
to the Library as a new asset: `upscale_image`, `remove_background`,
`extend_image` (each a small provider cost), and `resize_image` (local, free).
Re-running one returns the existing output instead of spending again.

**Spend**: `enhance_prompt` (small per-call model cost), `generate_image`
(provider cost per image).

## Rules that actually matter

### 1. `generate_image` spends real money
One call bills 1–4 images at provider rates against the studio's account. The
permission prompt you see IS the spend confirmation — don't add ceremony on top
of it, but don't fan out counts speculatively either. Default `count` to 1
unless variations were asked for.

### 2. Pass `brand_id`, or it lands in Personal
Omitting `brand_id` sends the asset to the caller's **Personal** brand and
applies no kit. For any client work that's wrong twice over: the asset is in the
wrong Library, and the brand's prompt prefix/suffix, banned terms, and palette
never applied. Call `list_brands` → pass the id. `get_brand` if you need to see
the kit before writing the prompt.

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
| `unauthorized` | Key revoked, or you were removed from the studio. Mint a new one. |
| daily limit | Same per-user generation cap as the web app. Resets daily. |
| async/video refusal | Model isn't sync-image. The message lists valid ones. |
| `does not accept reference images` | You passed `image_refs` to a model without `supportsImageInput`. Pick another; don't drop the reference. |
| `image_ref … not_found` | The asset isn't in this studio (or the id is wrong). |
