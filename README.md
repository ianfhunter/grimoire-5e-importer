# grimoire-5e-importer

DiceCloud v1 → Grimoire (`dnd-5e-2024`) character import.

One-shot converter: resolve DiceCloud v1 effect/formula stacks, emit a Grimoire
`grimoire://character/v1` import file aimed at the **`dnd-5e-2024`** sheet.

This is an **external tool**, not a Grimoire add-on (scrapers only fill book/system
metadata; core avoids game-specific import code). Suitable to share as a CLI that
talks to Grimoire’s character import API / UI.

## Requirements

- Python 3.10+
- For **public** sheet fetch without an API key: Playwright

```bash
pip install playwright
playwright install chromium
```

- For **private** sheets: set `DICECLOUD_API_KEY` (DiceCloud v1 account → API key).
  Avrae-compatible: `GET /character/{id}/json?key=…`

## Usage

```bash
# From a saved dump
python3 convert.py examples/creethe-quorath.dicecloud.json \
  -o examples/creethe-quorath.grimoire.json

# Fetch a public v1 sheet (Playwright)
python3 convert.py --fetch 'https://v1.dicecloud.com/character/QN748GQ9L24gXbiCW/Creethe-Quorath' \
  -o examples/creethe-quorath.grimoire.json \
  --dump-raw examples/creethe-quorath.dicecloud.json

# Private sheet
export DICECLOUD_API_KEY='…'
python3 convert.py --fetch QN748GQ9L24gXbiCW -o out.grimoire.json
```

Import in Grimoire: **Characters → Import**, or `POST /api/characters/import`
with `{"payload": <file contents>, "import_entries": true}` and an API key / session.

Optional: `--embed-schema examples/dnd-5e-2024.schema.json` so the file installs
the sheet on an instance that lacks it.

## What maps

| DiceCloud v1 | Grimoire field |
|---|---|
| Resolved ability effects | `strength`…`charisma` (final totals) |
| Classes / level | `level`, `klass` (embedded stub entry), subclass from Oath/… features |
| Race | `species` (embedded stub) + `species_traits` freeform |
| Background (1st backstory line) | `background` (embedded stub) |
| Skills / saves | `skill_profs`, `*_save_prof` |
| HP / speed / AC | `hp_max`, `speed`, `_overrides.armor_class` |
| Attacks | `attacks` list (bonus/damage evaluated) |
| Spells | `spells` `_inline` + prepared |
| Features / feats | `features` / `feats` `_inline` (also from `Free Feat:` effects) |
| Mundane items / coins | `equipment` `_inline`, `gp`/`sp`/… |
| Magic-ish items (`+N`, potions, …) | `magic_items` |
| Note tabs | appended into `notes` |
| Portrait URL | `portrait_url` on the file (manual upload — Grimoire stores files) |

## Limits

- One-shot — no live sync back to DiceCloud
- 2014-ish v1 sheets → **2024** Grimoire schema (numbers transfer; rules may differ)
- No SRD pack linking unless you hand-map `_ref` ids later
- Complex homebrew toggles may need manual cleanup after import
- Formula evaluator is intentionally small (arithmetic + `floor`/`ceil`/`min`/`max`)

## Licence

MIT for this tool. Do not copy Avrae’s GPL sources into Grimoire core; this
reimplements a minimal effect resolver for export only.
