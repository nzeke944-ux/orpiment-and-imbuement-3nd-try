# Orpiment & Imbuement — Fabric Mod Source (MC 26.2)

Vanilla-friendly gear-upgrade mod. Diamond → Netherite → Orpiment. Custom
weapons, imbuement enchantments, curses, and an Imbuement Specialist
villager. This is a **source-only** package — no compiled `.jar`. It's built
for **Fabric Loader**, targeting **Minecraft 26.2**, Java 21.

## What's in here

```
build.gradle, settings.gradle, gradle.properties   - Loom build config
src/main/java/com/orpiment/imbuement/
  OrpimentImbuement.java          - mod entrypoint, registers everything
  registry/                       - items, armor stats, enchant keys, tags,
                                     villager profession, damage types
  item/                           - Scythe, Leviadon, Shadow Dagger,
                                     Orpiment Wingsuit custom item classes
  event/                          - all runtime gameplay logic (curses,
                                     enchant effects, villager trades)
  mixin/                          - melee damage scaling, Lust crossbow hook
  client/                         - client entrypoint (minimal)
src/main/resources/
  fabric.mod.json, *.mixins.json
  assets/orpimentimbuement/       - textures (cropped from your sheet),
                                     item models, armor equip asset, lang
  data/orpimentimbuement/         - enchantment defs, recipes, tags,
                                     custom damage type
```

## Textures

All 25 icons were auto-cropped straight from your texture sheet
(`textures/` folder has the originals) and trimmed/centered onto
transparent 32×32 canvases. The 4 batch-2 enchant books that weren't on
the sheet (Timber, Spring, Rebound, Updraft) are hue-shifted placeholders
generated from the Boring book — swap those first if you want them
distinct.

**Not on the sheet, so generated as simple placeholders:** the armor
"equip layer" textures (`textures/models/armor/orpiment_layer_1.png` /
`_layer_2.png`) — the sheet only had item-icon renders, not the
worn-body skin. Right now they're a flat black-with-gold-trim recolor.
Replace these for a real in-world look on the player model.

## What's fully wired vs. what's a first-pass stub

**Solid / functional as written:**
- All items, armor stats, tool stats (see doc-comments in `ModItems.java`
  and `ModArmorMaterials.java` for the exact numbers I chose)
- Recipes: Orpiment Ingot, all 9 smithing-table upgrades, Wingsuit, Scythe,
  Shadow Dagger
- Boring (real BFS vein-mine, no size cap) and Timber (real flood-fill tree
  fell)
- Reckless, Vampirism, Phoenix, Reaping (anvil-combine + on-hit chance)
- Swiftstep, Spring, Rebound, Updraft, Stalking, Insite
- Imbuement Specialist villager: new POI on Chiseled Bookshelf, reads the
  shelf's 6 slots live and sells whatever enchanted books are stored there
  for 26 emeralds + 1 book
- Scythe Charged Strike and Shadow Dagger backstab damage multipliers (via
  a `LivingEntity#damage` mixin)

**First-pass / flagged simplifications — polish these before release:**
- **Leviadon recipe** doesn't actually return the empty bucket (vanilla
  shaped recipes can't do a partial remainder like that without a custom
  `RecipeSerializer`). Right now the water bucket is just consumed.
- **Leviadon Over-Enchanting / Hydro-Charge free-cast:** charge accumulation
  works (stored in item NBT, caps at 5), but actually *using* a charge to
  bypass vanilla's "must be in water/rain" check on Riptide/Channeling
  needs a mixin into `TridentItem`'s throw logic that I didn't write —
  `LeviadonItem.consumeCharge()` is ready to be called from there.
- **Curse of FEESH** is a simplified tick-based approximation (air drains on
  land, refills + speed boost underwater). A true full reversal of swim
  physics/controls needs a movement mixin.
- **Curse of Eternity** grants Unbreakable + drains 1 hunger every ~20s
  while equipped, rather than precisely "1 hunger per 20 actual uses" —
  counting real uses (attacks/blocks broken) needs deeper hooks.
- **Enchanted book textures per-enchantment** (so a Swiftstep book actually
  looks different from a Phoenix book in your hand) needs a `minecraft:select`
  item model keyed off which enchantment is stored — I left the individual
  model JSONs in `models/item/` but didn't wire the selector, since the exact
  schema is finicky and version-dependent. Vanilla enchanted books are all
  one item ID, so this is inherently a client-model-override problem.
- Curses are set up as loot-only (`"curse": true`, not sold at the
  enchanting table), but I didn't inject them into any vanilla loot tables —
  add them to a structure's loot table (or your own custom loot injection)
  to make them actually obtainable in survival.

## Version note

I don't have real documentation for "Minecraft 26.2" specifically — I built
this against the most current data-driven-enchantment / component-based
armor API pattern I know (1.21.x-era Fabric). Method names in the registry
classes are the most likely spot to need small adjustments if 26.2's actual
API differs. `gradle.properties` has placeholder version strings
(`yarn_mappings`, `fabric_version`) that need to be swapped for whatever's
actually published for 26.2 before this will compile.
