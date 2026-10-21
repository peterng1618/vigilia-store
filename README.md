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
every push to `main` that touches `catalogue/**`, by running
[`scripts/generate-catalogue-index.mjs`](https://github.com/peterng1618/vigilia/blob/main/scripts/generate-catalogue-index.mjs)
from the app repo at a pinned commit. It carries one entry per item with that
item's **derived facet record** — author, colour mode, sensors, style tags and
shape — because the client filters and sorts on those and they are never authored
into `theme.json`.

**An item's facets are derived, never written.** `item.json` carries what a person
decided; the facet record is computed from `theme.json` and only ever appears in
`index.json`.

## How an item arrives

A pull request that adds one `catalogue/<id>/` directory. An item is JSON, so the
review instrument is the diff itself. The app opens the pull request on the
author's behalf — that lands with Vigilia's phase E.

## Delivery

`index.json` is fetched from `raw.githubusercontent.com`, which serves it with a
five-minute cache and an `ETag` a browser revalidates against for free.
Thumbnails come from jsDelivr, which caches a branch for twelve hours.

## The catalogue is empty

No item has been submitted yet, so `index.json` carries zero entries. That is the
honest state rather than a placeholder: the client draws its own empty catalogue
rather than a fabricated one.
