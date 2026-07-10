# Pokémon TCG Pricing & Collection Analytics

I built this because Pokémon card pricing is a fascinating mess. It's hundreds of
thousands of tiny, highly correlated markets, and it's genuinely hard to tell whether
a price trend belongs to one card, an artist, a set, or a Pokémon. Low-population
cards break in strange ways: two listings each set to climb a few percent over the
lowest, so the price spirals; worse-condition copies sometimes list higher than clean
ones; plenty of cards have no usable signal at all. Making sense of that sparse,
correlated data is the whole point.

It's a local, **DuckDB-based pricing and collection-analytics pipeline**. It ingests
card and price data from pokemontcg.io, PokeAPI, and TCGplayer exports, models it into
a star-schema warehouse, and derives inventory value, set completion, artist/variant
premiums, and buy/sell signals, all offline with no service required.

> **Full design write-up: [ARCHITECTURE.md](ARCHITECTURE.md)** — data flow, schema,
> the analytics views, and a candid section on the pipeline's data-quality limits and
> planned mitigations. It's the most interesting read in the repo.

## What it does

- **Warehouses the data properly.** A star schema (`scripts/schema.sql`) with
  dimensions (`dim_card`, `dim_set`, `dim_pokemon`, `dim_variant`, `dim_condition`,
  `dim_source`), append-only price and inventory facts, a card crosswalk, and a
  curation layer. Ingestion (`scripts/*.ps1`) is idempotent and raw-first cached, so
  nothing is ever silently overwritten.
- **Slices the correlated markets.** Materialized views (`scripts/views.sql`) for
  inventory value, set completion, rarity and Pokémon price indices, artist/variant
  premiums, and buy/sell signals. This is the fun part: hundreds of thousands of
  related markets are catnip for trying different ways to cut the data.
- **Is honest about dirty, sparse data.** The design leans on quarantine-over-guess
  crosswalking and a clear trust hierarchy (`card_override` > `outlier_flag` > raw
  fact), and the views always read the cleaned, curated layer. ARCHITECTURE.md
  documents where the model breaks and why, with evidence: thin top-of-market pricing,
  promo-set cohort distortion (a measured **400× intra-cohort price spread**), and
  meta-relevance confounds, each with the mitigation applied or planned.

## Where it's at

This is early and very unfinished, and I'll say so plainly: it's the repo with the
most real potential and the most work left to become a product. The honest payoff so
far is better questions, not answers. I don't have much experience with sparse data
like this, and getting my hands dirty with it is a lot of the point.

Next up is **historical trend tracking** (the facts are append-only precisely so this
is possible), on top of the mitigations already listed in ARCHITECTURE.md.

## Pipeline

```
external sources ─► PowerShell ingestion ─► raw/ cache ─► DuckDB ─► views ─► HTML/PDF
(pokemontcg.io,        (seed_*.ps1,         (idempotent)  (star    (SQL      (Chrome
 PokeAPI, TCGplayer     import_*.ps1)                      schema)  window     headless)
 CSV exports)                                                       fns)
```

## What's excluded

`.gitignore` omits the DuckDB database (`pipeline.duckdb`, large and holds personal
inventory), the `raw/` data caches (third-party API + TCGplayer data), and generated
`reports/` (personal collection valuation).

## Stack

DuckDB · SQL · PowerShell · Chrome headless (HTML → PDF).

## Related

Companion repo:
**[pokemon-tcg-checklists](https://github.com/dgoodenough/pokemon-tcg-checklists)**, a
printable collector-checklist and binder-grid generator over the same card domain.

## License

[MIT](LICENSE), covers the original code, not the third-party data it ingests.
Pokémon and card data © The Pokémon Company / Nintendo / Game Freak.
