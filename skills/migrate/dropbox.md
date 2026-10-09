# Dropbox

Each person connects their own Dropbox to Cauldron, once, in the app. Cauldron
only reads from it; nothing in Dropbox is changed, moved or deleted. Guests can't
import from Dropbox.

## Tools

| Tool | What it does |
|---|---|
| `dropbox_status` | Connected or not, which Dropbox account, and any import running or last finished. |
| `dropbox_scan` | Starts reading one space: `personal` or `team`. Returns a `scanId`. |
| `dropbox_get_scan` | Polls a scan and lists one level of folders with file counts. Pass `path` to open a folder. |
| `dropbox_preview` | What an import of the chosen folders would do. Copies nothing. |
| `dropbox_import` | Starts the import. Returns an `importId`. |
| `get_import_status` | Counts, failed files with reasons, and where the files landed. |
| `resume_import` | Retries failed and waiting files of an import. |
| `cancel_import` | Stops an import. Files already imported stay. |

None of these spend credits.

## Steps

1. `dropbox_status`.
   - `connected: false`: give the person `connectUrl` and ask them to connect
     there (Library → Import → From Dropbox). You cannot do this step. Call
     `dropbox_status` again when they say it's done.
   - `needsReconnect: true`: same, the connection ended and needs renewing.
   - `activeImport` is set: an import is already running. Offer to check on it
     with `get_import_status` before starting another.
2. `dropbox_scan` with `space`. Use `personal` unless the account has a team
   space (`account.hasTeamSpace`) and the person says the files are there. A
   scan reads the whole space; folders are chosen later.
3. `dropbox_get_scan` until `status` is `complete`. Show the folders. A folder
   with `state: "too_big"` can't be taken whole: open it and choose subfolders.
4. `dropbox_preview` with `scan_id`, `brand_id`, and `include` set to the folder
   paths exactly as `dropbox_get_scan` returned them (`"."` means loose files in
   the root). Optional: `exclude`, `flatten_depth`, `keep_duplicates`.
   - `canImport: false`: relay `blocked` and stop.
   - `earlierImports` is not empty: some of these files came in before. Ask
     whether to add to that collection (recommended) or make a new one. Either
     way nothing is copied twice.
   - Deep folder trees: `flatten_depth` joins everything below that level into
     one collection per branch, named like "RAW / Day 1". Offer it when
     `maxDepth` is above 3.
5. After a yes, `dropbox_import` with the same selection and exactly one of
   `collection_id` (from `earlierImports`) or `new_collection_name`
   (`defaultNewCollectionName` is a good default). Dropbox tags come along
   unless `dropbox_tags` is false; `extra_tags` adds your own.
6. `get_import_status` until `finished` is true, then report `counts` and
   `landed`. If there are `failedFiles`, offer `resume_import`.

## What gets left out

- **Duplicates**: identical files in the selection. One copy is kept, unless
  `keep_duplicates` is true.
- **Can't import**: file types or sizes Cauldron doesn't take. The preview names
  each reason with example files.
- **Already imported**: linked into the collection, not copied again.

## Limits

One import copies up to 5,000 files. For a larger library, import it in several
runs, folder by folder; the preview says when a selection is too big. There is
also an hourly cap on starting imports; the refusal says how long to wait.

User guide: https://docs.cauldron.studio/guides/dropbox-import/
