---
name: migrate
description: Move a library of files from another service into a Cauldron Library through the Cauldron MCP. Use when asked to migrate, import, move, bring over or copy folders or files from Dropbox into Cauldron, to check on or retry an import, or when someone asks how to get an existing library into Cauldron. Currently covers Dropbox.
---

# Migrate a library into Cauldron

A migration copies files from a source into one brand's Library. Cauldron does
the copying on its own servers; you start it, watch it, and report. Nothing
downloads to this machine.

The tools come from the Cauldron MCP. If they aren't available, the server isn't
connected: follow Setup in the `cauldron` skill first.

## The flow, for every source

1. **Pick the brand.** `list_brands`, then ask which brand the files belong to
   unless the person already said. Never guess: an import into the wrong brand
   is tedious to undo.
2. **Check the source is connected.** If it isn't, you cannot connect it
   yourself. Give the person the link the tool returns, wait for them to say
   it's done, then check again.
3. **Scan**, and poll until the scan completes.
4. **Let the person choose folders.** Show the top-level folders with their file
   counts. Drill into a folder only when asked. Don't pick for them.
5. **Preview**, and show it before anything is copied: how many files will be
   imported, how many are already in Cauldron, duplicates left out, files that
   can't be imported and why, and which collections they land in.
6. **Confirm.** Start the import only after an explicit yes to that preview.
7. **Import**, then poll the status until it finishes. Poll every 20 to 30
   seconds, not in a tight loop. Large imports take a long time; say so and offer
   to check back.
8. **Report** in the product's words: Imported, Already imported, Duplicates
   left out, Can't import, Failed. If files failed, list them with their reasons
   and offer to retry.

## Rules

- **Files arrive as drafts.** Tell the person they need approving in the Library
  before anyone works from them.
- **Running the same import again is safe.** Only new files are copied; files
  imported before are linked into the collection, never copied twice. Say this
  when someone worries about duplicates.
- **Relay the tool's sentences.** Refusals and failures come back as plain
  sentences. Pass them on; don't show a bare error code or invent a cause.
- **Don't work around a refusal.** Limits, a guest account, or a brand the person
  can't add files to are decisions the studio made.

## Sources

- **Dropbox**: read `dropbox.md` in this folder.
