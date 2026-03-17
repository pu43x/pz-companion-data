# pz-companion-data

Static reference data for the PZ Companion App. All files are plain JSON — no build step required.

## Phase 1 Contents

| Path | Description |
|------|-------------|
| `recipes/index.json` | Master list of recipe category files |
| `recipes/cooking.json` | 10 cooking recipes |
| `recipes/construction.json` | 10 construction recipes |
| `recipes/weapons.json` | 5 weapon/tool recipes |
| `recipes/medical.json` | 5 medical recipes |
| `loot/locations.json` | Loot hints for all ingredient and tool item IDs |
| `items/items.json` | Item metadata (name, category, weight) for all referenced items |
| `guides/cooking.json` | 4-section cooking and nutrition guide |
| `guides/medical.json` | 4-section medical and first aid guide |
| `guides/tools.json` | 4-section tools reference guide |
| `guides/foraging.json` | 4-section foraging guide |
| `guides/tips.json` | 5 general survival tips |

## Consumed By

- **pz-companion-api** — loaded at startup as fallback recipe data when a server has not completed Phase 4 recipe sync
- **pz-companion-app** — guide content bundled for offline access via the service worker

## Adding New Recipes

See CLAUDE.md for schema reference, data rules, and the contribution checklist.
