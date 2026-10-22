# Hand Animations packs

You can give any item its own grip, animations and texture with an ordinary resource pack. Put one JSON file per item
(or group of items) in:

```
assets/<your_namespace>/handanimations/items/<anything>.json
```

Packs reload with the other resource packs (F3+T). Hold an item and run **`/handanimations item`** to see its ids,
kind and which pack file matches it. A file with a mistake is skipped and the log says why.

A complete working example is in `examples/HandAnimationsExamplePack` (also zipped as `HandAnimationsExamplePack.zip`).

## A file

```json
{
  "match": { "skyblock_id": "HYPERION" },
  "priority": 0,
  "kind": "blade",
  "model": "example:crystal_blade",
  "hold": { "pitch": -10, "wrist_roll": 15 },
  "item": { "translation": [0, 1, 0], "rotation": [0, 0, 0], "scale": 1.1 },
  "swing":   { "keys": [ ... ] },
  "use":     { "length": 10, "keys": [ ... ] },
  "equip":   { "length": 8,  "keys": [ ... ] },
  "inspect": { "length": 60, "keys": [ ... ] }
}
```

Everything except `match` is optional. Leave something out and the item keeps its normal behaviour.

### match

You need at least one of these. Each one can be a single value or a list, and an item has to pass every check given.

| key | matches |
| --- | --- |
| `items` | Minecraft item ids, e.g. `"minecraft:stick"` |
| `skyblock_id` | Hypixel SkyBlock ids, e.g. `"HYPERION"` or `"ASPECT_OF_THE_END"` |
| `name_contains` | text in the item's name, e.g. `"Aspect of the"` |
| `kind` | the item's built-in kind: `generic`, `blade`, `tool`, `drill`, `bow`, `rod`, `wand`, `block`, `head` |

When several files match, the one with the higher `priority` wins. On a tie, the more specific file wins:
`skyblock_id`, then `name_contains`, then `items`, then `kind`.

### kind

Gives the item another kind's built-in swings and right-click animations, e.g. `"wand"` for a stick that should
thrust instead of jab.

### model (textures)

The item model drawn in the hand, as `namespace:path`. It points at `assets/<namespace>/items/<path>.json`, the same
kind of item model file Minecraft uses. Add the model and its texture to the same pack:

```
assets/example/items/crystal_blade.json          {"model": {"type": "minecraft:model", "model": "example:item/crystal_blade"}}
assets/example/models/item/crystal_blade.json    {"parent": "minecraft:item/handheld", "textures": {"layer0": "example:item/crystal_blade"}}
assets/example/textures/item/crystal_blade.png
```

It only changes what is drawn in first person. Inventory icons stay as they are.

### hold

Added to the arm's normal pose while the item is held. All values are for the right hand; the left hand mirrors them.

| channel | meaning | unit |
| --- | --- | --- |
| `x`, `y`, `z` | move the fist right, up, back | blocks (about 0.1 is small, 0.3 is big) |
| `pitch` | raise (negative) or lower (positive) the arm | degrees |
| `yaw` | swing the arm inward (positive) or outward | degrees |
| `roll` | roll the whole arm | degrees |
| `wrist_roll` | twist the hand | degrees |
| `wrist_pitch` | tip the hand forward or back | degrees |

### item

Moves the item inside the hand, the same way a model's display settings do. `translation` is in pixels,
`rotation` is in degrees (X, then Y, then Z), and `scale` multiplies the size.

### Animations

`swing` replaces the left-click swing, `use` the right-click animation, `equip` the draw animation when you switch to
the item, and `inspect` the inspect key (Y).

```json
"use": {
  "length": 10,
  "keys": [
    { "t": 0 },
    { "t": 0.35, "y": 0.2, "pitch": -50 },
    { "t": 0.6, "pitch": 25, "spin": 90 },
    { "t": 1 }
  ]
}
```

- `length` is in ticks (20 per second). Swings ignore it, because a swing always lasts as long as the game's swing.
- Each key has a time `t` from 0 to 1 and any of the `hold` channels, plus `spin`, which turns the item in the hand in
  degrees. A channel left out of a key is 0, the resting pose.
- Keys must be in order of `t`. The motion curves smoothly through them.
- Start and end with an empty key (`{"t": 0}` and `{"t": 1}`) so the animation begins and ends at rest.
