# vigilia-store

The catalogue behind [Vigilia](https://github.com/peterng1618/vigilia)'s theme
store. GitHub is the whole backend: there is no server to run and no recurring
cost.

## What is here

```
catalogue/<id>/item.json      what the item is — authored, and the file a PR changes
catalogue/<id>/theme.json     the theme document, exactly as the editor saves it
catalogue/<id>/thumbnail.*    the picture a cell draws
index.json                    generated; do not edit
```

**`index.json` is generated and never hand-edited.** A workflow regenerates it on
every push to `main` that touches `catalogue/**`, by checking out the app repo at
a pinned commit and running its generator:

```
node app/src/web/packages/host/bin/vigilia.js \
  --build-catalogue-index catalogue --out index.json \
  --generated-at "$(git log -1 --format=%cI)"
```

The generator lives in the app repo (`packages/host/src/catalogue/build-index.ts`)
rather than here, because the facet record it writes is computed by
`@vigilia/renderer-core` — the one layer both this catalogue and the app can
import. `index.json` therefore carries one entry per item with that item's
**derived facet record**: author, colour mode, sensors, style tags and shape.

**An item's facets are derived, never written.** `item.json` carries what a person
decided; the facet record is computed from `theme.json` and only ever appears in
`index.json`.

## `item.json`

Exactly these nine keys. A missing or mistyped one refuses the **whole build** and
names the field, so a typo is never published as a silently missing item.

| Key | Type | Meaning |
|---|---|---|
| `kind` | string | `dashboard-theme` today; the only kind v1 exchanges |
| `name` | non-empty string | what a cell prints |
| `version` | non-empty string | the item's own version, e.g. `1.4.0` |
| `createdAt` | string | ISO 8601 instant, when the item first appeared |
| `updatedAt` | string | ISO 8601 instant of the last change |
| `downloads` | integer ≥ 0 | shown in the cell's facts band |
| `favorites` | integer ≥ 0 | shown when a person sorts by favourites |
| `packageUrl` | non-empty string | where the theme package is downloaded from |
| `thumbnail` | non-empty string | a path under `catalogue/<id>/`, joined to the CDN base |

**No `id` and no `author`.** The `id` is the directory name; the author is a facet
derived from `theme.json`'s `metadata.author`. Writing either into `item.json` does
nothing, because the shape above is the contract the generator refuses against.

## How an item arrives

A pull request that adds one `catalogue/<id>/` directory. An item is JSON, so the
review instrument is the diff itself. The app opens the pull request on the
author's behalf — that lands with Vigilia's phase E.

## Delivery

`index.json` is fetched from `raw.githubusercontent.com`, which serves it with a
five-minute cache and an `ETag` a browser revalidates against for free.
Thumbnails come from jsDelivr, which caches a branch for twelve hours. The
workflow pins the app repo at a commit rather than a branch so a change to the
generator cannot alter this catalogue's output without a commit here.

## The catalogue is empty

No item has been submitted yet, so once the Action has run `index.json` carries
zero entries. That is the honest state rather than a placeholder: the client
draws its own empty catalogue rather than a fabricated one.
