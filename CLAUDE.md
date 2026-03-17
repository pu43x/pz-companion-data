# CLAUDE.md — pz-companion-data

## What This Repo Is

The static data repository for the PZ Companion App. Contains recipe definitions, loot location hints, guide content, and item metadata — all as plain JSON files.

This repo contains no runnable code. It is consumed by:
- `pz-companion-api` — loaded at startup as a fallback when no server-synced recipes exist
- `pz-companion-app` — bundled for offline guide access via the service worker

---

## Project Context

Part of the `pz-companion` suite:

| Repo | Role |
|------|------|
| `pz-companion-mod` | Lua mod on game server |
| `pz-companion-api` | Node.js middleware — consumes this repo as fallback |
| `pz-companion-app` | React PWA — bundles guides from this repo |
| `pz-companion-data` | **This repo.** Static reference data |

### Important: This is fallback data, not the live source

In Phase 4, the Lua mod dumps per-server recipe data from the live game. That becomes the primary source. This repo is used when:
- A server has not yet completed its first recipe sync
- The server is offline and the app needs to show something
- A recipe exists in the game but wasn't captured by the mod

Loot location hints and guide content are always served from this repo — the game server never provides those.

---

## Folder Structure

```
pz-companion-data/
├── README.md
├── recipes/
│   ├── index.json               ← Master list: { "files": ["cooking.json", ...] }
│   ├── cooking.json
│   ├── construction.json
│   ├── weapons.json
│   ├── medical.json
│   ├── electrical.json
│   └── carpentry.json
├── loot/
│   └── locations.json           ← item_id → array of location type strings
├── guides/
│   ├── cooking.json
│   ├── medical.json
│   ├── tools.json
│   ├── foraging.json
│   └── tips.json
└── items/
    └── items.json               ← item_id → { name, category, weight }
```

---

## Schemas

### recipes/*.json

Each file is an array of recipe objects:

```json
[
  {
    "id": "base.woodenwall",
    "name": "Wooden Wall Frame",
    "category": "construction",
    "skill_required": {
      "skill": "carpentry",
      "level": 2
    },
    "ingredients": [
      { "item_id": "Base.Plank", "name": "Plank", "count": 4 },
      { "item_id": "Base.Nails", "name": "Nails", "count": 8 }
    ],
    "tools_required": [
      { "item_id": "Base.Hammer", "name": "Hammer" }
    ],
    "crafting_chain": [
      { "produces": "Base.Plank", "recipe_id": "base.plank" }
    ],
    "notes": "Core building block for base construction."
  }
]
```

#### Field reference

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `id` | string | ✅ | Lowercase, dot-separated. e.g. `base.woodenwall` |
| `name` | string | ✅ | Display name |
| `category` | string | ✅ | One of: `cooking`, `construction`, `weapons`, `medical`, `electrical`, `carpentry`, `survival` |
| `skill_required` | object or null | ✅ | `{ skill, level }` or `null` if no skill needed |
| `ingredients` | array | ✅ | Items consumed in crafting |
| `tools_required` | array | ✅ | Tools needed but not consumed. Empty array if none. |
| `crafting_chain` | array | ✅ | Ingredients that are themselves craftable. Empty array if none. |
| `notes` | string or null | — | Optional tip shown in recipe detail |

#### Valid skill names

```
carpentry, cooking, firstAid, mechanics, aiming, blunt, blade,
sneaking, foraging, farming, fishing, trapping, electricity, metalworking
```

#### Valid categories

```
cooking, construction, weapons, medical, electrical, carpentry, survival
```

---

### loot/locations.json

A flat object mapping PZ item IDs to arrays of location type strings:

```json
{
  "Base.Nails":        ["hardware stores", "garages", "sheds", "warehouses"],
  "Base.Plank":        ["warehouses", "construction sites", "lumber yards", "sheds"],
  "Base.DuctTape":     ["hardware stores", "garages", "storage units"],
  "Base.Hammer":       ["hardware stores", "garages", "toolboxes", "kitchens"],
  "Base.Saw":          ["hardware stores", "garages", "toolboxes"],
  "Base.Bandage":      ["medical cabinets", "bathrooms", "pharmacies", "hospitals"],
  "Base.Antibiotics":  ["pharmacies", "hospitals", "medical cabinets"]
}
```

- Keys must be valid PZ item IDs (`Module.ItemName` format)
- Location strings are human-readable, shown directly in the app UI
- Keep location strings lowercase and concise (max ~30 chars)
- 2–6 locations per item is ideal

---

### guides/*.json

Each file is an array of guide sections:

```json
[
  {
    "heading": "Basic Principles",
    "body": "Cooked food provides better nutrition and reduces illness risk. Always use a heat source — campfire, oven, or BBQ. Raw meat can cause food poisoning."
  },
  {
    "heading": "Cooking Skill",
    "body": "Higher cooking skill reduces burnt food chance and unlocks new recipes. Level 2+ allows stews and soups which provide multi-stat boosts."
  }
]
```

Guide files map to these display titles in the app:

| File | Display title | Icon |
|------|--------------|------|
| `cooking.json` | Cooking & Nutrition | 🍳 |
| `medical.json` | Medical & First Aid | 🩹 |
| `tools.json` | Tools Reference | 🔧 |
| `foraging.json` | Foraging Guide | 🌿 |
| `tips.json` | Survival Tips | 💡 |

---

### items/items.json

```json
{
  "Base.Hammer":    { "name": "Hammer",    "category": "Tool",     "weight": 0.9 },
  "Base.Nails":     { "name": "Nails",     "category": "Material", "weight": 0.1 },
  "Base.Bandage":   { "name": "Bandage",   "category": "Medical",  "weight": 0.1 }
}
```

Used by the API to enrich item IDs with display names and weights when the game server isn't the source.

---

### recipes/index.json

```json
{
  "files": [
    "cooking.json",
    "construction.json",
    "weapons.json",
    "medical.json",
    "electrical.json",
    "carpentry.json"
  ]
}
```

The API reads this first to know which recipe files to load. Add new category files here when created.

---

## Data Rules

- **All item IDs must use PZ format** — `Module.ItemName` (e.g. `Base.Hammer`, `Base.Nails`)
- **Recipe IDs must be unique across all files** — lowercase, dot-separated
- **No duplicate recipes** — if a recipe appears in the Lua mod output, this file is ignored for that server
- **crafting_chain only lists direct dependencies** — do not recurse deeply. The API resolves chains at query time.
- **Guide body text must be plain text** — no markdown, no HTML. The app renders it as-is.
- **Validate JSON before committing** — malformed JSON silently breaks the API fallback

---

## Contributing New Recipes

1. Find the recipe in-game or on the PZ wiki (https://pzwiki.net)
2. Add it to the appropriate category file
3. Add any new item IDs to `items/items.json`
4. Add loot hints for any ingredients to `loot/locations.json`
5. Validate your JSON (use a linter or `jq . filename.json`)
6. Open a PR with the game version the recipe was verified against in the description

---

## PZ Item ID Reference

Common modules:
- `Base.*` — core game items
- `farming.*` — farming items
- `Hydrocraft.*` — Hydrocraft mod items (if applicable to your server)

The full item list can be found at: https://pzwiki.net/wiki/Items
