---
name: cauldron-docs
description: Look up how Cauldron works in its user documentation at docs.cauldron.studio. Use when asked how to do something in Cauldron (brands, the Library, review and approval, sharing with clients, projects, importing from Dropbox, billing, API keys, limits), what a Cauldron feature does, or why the app behaved a certain way, and before explaining a Cauldron feature from memory.
---

# Cauldron docs

The user guides live at https://docs.cauldron.studio. Read them instead of
answering from memory: the product changes often and the docs are kept current.
Nothing is bundled here, so every answer reflects today's docs.

## How to read them

1. **Find the page.** Fetch https://docs.cauldron.studio/llms.txt for the
   overview, or https://docs.cauldron.studio/sitemap-0.xml for every page URL.
   Guides sit under `/guides/<topic>/`, settings under `/settings/<topic>/`.
2. **Fetch one page as markdown.** Add `.md` to the page path:
   `https://docs.cauldron.studio/guides/library.md`. One page is about 1,500
   tokens. Fetch two or three pages when a question spans features.
3. **Answer from the page**, in the product's own words, and link the page.

Never fetch `llms-full.txt` for a single question: it is the whole site, around
80,000 tokens. Use it only when asked to summarise the docs as a whole.

## Where things are

| Question is about | Page |
|---|---|
| What Cauldron is, who it's for | `/introduction.md`, `/concepts.md` |
| Getting started, sign-up | `/getting-started.md`, `/sign-up.md` |
| Brands and brand kits | `/guides/brands.md` |
| The Library, uploading, finding assets | `/guides/library.md`, `/guides/uploading.md`, `/guides/finding-assets.md` |
| Importing from Dropbox | `/guides/dropbox-import.md` |
| Projects, collections, sharing with clients | `/guides/projects.md`, `/guides/sharing.md` |
| Review and approval | `/guides/review.md` |
| Creating images and video, tools, models | `/guides/creating-images.md`, `/guides/creating-video.md`, `/guides/tools.md`, `/guides/models.md` |
| Connecting AI tools, API keys | `/guides/connected-ai-tools.md`, `/guides/api-keys.md` |
| Billing, usage and limits | `/settings/billing.md`, `/guides/usage-and-limits.md` |
| What changed recently | `/changelog.md` (large; fetch only when asked) |

If a page is missing, check the sitemap: pages move. If the docs don't cover
the question, say so rather than guessing; the `cauldron` skill's tools can
often answer by looking at the studio itself.
