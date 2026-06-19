# Pokémon TCG Pricing & Collection Analytics

A local, **DuckDB-based pricing and collection-analytics pipeline** for the Pokémon
Trading Card Game. It ingests card and price data from pokemontcg.io, PokeAPI, and
TCGplayer exports, models it into a star-schema warehouse, and derives inventory
valuation, set completion, artist/variant premiums, and buy/sell signals — all
offline, no service required.

> **Full design write-up: [ARCHITECTURE.md](ARCHITECTURE.md)** — data flow, schema,
> the analytics views, and a candid section on the pipeline's data-quality limits
> and planned mitigations. That's the most interesting read in the repo.

## Pipeline

```
external sources ─► PowerShell ingestion ─► raw/ cache ─► DuckDB ─► views ─► HTML/PDF
(pokemontcg.io,        (seed_*.ps1,         (idempotent)  (star    (SQL      (Chrome
 PokeAPI, TCGplayer     import_*.ps1)                      schema)  window     headless)
 CSV exports)                                                       fns)
```

## What's in here

- **`scripts/schema.sql`** — the warehouse: dimensions (`dim_card`, `dim_set`,
  `dim_pokemon`, `dim_variant`, `dim_condition`, `dim_source`), append-only facts
  (`fact_price`, `fact_inventory_snapshot`), a crosswalk (`card_alias`), and a
  curation layer (`card_override`, `outlier_flag`).
- **`scripts/views.sql`** — materialized analytics: inventory value, set completion,
  rarity & Pokémon price indices, artist/variant premiums, buy/sell signals.
- **`scripts/*.ps1`** — idempotent ingestion (`seed_sets`, `seed_cards`,
  `seed_pokemon`, `import_tcgplayer_prices`) and report rendering
  (`build_dashboard`, `build_premium_change_report`).

## Design principles

Raw-first caching · idempotent ingestion · append-only facts · quarantine-over-guess
crosswalking · a clear trust hierarchy (`card_override` > `outlier_flag` > raw fact).
The materialized views always read the cleaned + curated layer. See
[ARCHITECTURE.md](ARCHITECTURE.md) for the full rationale.

## Honest data-quality analysis

ARCHITECTURE.md documents where the model breaks and why — thin top-of-market
pricing, promo-set cohort distortion (a measured **400× intra-cohort price spread**),
and meta-relevance confounds — each with the empirical evidence and the mitigation
applied or planned.

## What's excluded

`.gitignore` omits the DuckDB database (`pipeline.duckdb` — large and holds personal
inventory), the `raw/` data caches (third-party API + TCGplayer data), and generated
`reports/` (personal collection valuation).

## Stack

DuckDB · SQL · PowerShell · Chrome headless (HTML → PDF).

## Related

Companion repo: **[pokemon-tcg-checklists](https://github.com/dgoodenough/pokemon-tcg-checklists)**
— a printable collector-checklist & binder-grid generator over the same card domain.

## License

[MIT](LICENSE) — covers the original code, not the third-party data it ingests.
Pokémon and card data © The Pokémon Company / Nintendo / Game Freak.
