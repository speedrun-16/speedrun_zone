# Speedrun: Zones
![Author](https://img.shields.io/badge/Author-PWNED-8cf?style=for-the-badge "Author") ![Version](https://img.shields.io/badge/Version-1.0-blue?style=for-the-badge "Version")

One of the core plugins that powers our speedrun16 server.

Provides a shared zone registry, editor, touch detection and API that allows other plugins to register custom zone types with their own callbacks and visibility settings.

![preview](https://github.com/user-attachments/assets/35021c28-b854-4ef7-a924-03508e986a67)

## ☰ Features
- Polygon and legacy AABB zone shapes, with per-map JSON persistence
- AABB is read-only for legacy compatibility; new zones are always polygons
- In-game editor with anchors, wall expand/shrink, and undo
- Type registry with colors, descriptions, and visibility flags
- Player touch detection with per-type callbacks and global forwards
- Editor mode visualization and aim-based face highlighting

## ☰ Dependencies
- [neumenu](https://github.com/speedrun-16/neumenu)
- [visual](https://github.com/speedrun-16/visual)

## ☰ Data Storage
Zones are stored per map in the AMX Mod X data directory at `.speedrun/zones/<map>.json`.

The directory is created on first run. Zones are saved on plugin unload and loaded on map start.

### JSON Format
Polygon zones:
```json
{
  "type": "zone_start",
  "id": "ZONE#1",
  "polygon": true,
  "height": "128.0",
  "vertices": [
    {"x": "0.0", "y": "0.0", "z": "0.0"},
    {"x": "64.0", "y": "0.0", "z": "0.0"},
    {"x": "64.0", "y": "64.0", "z": "0.0"}
  ]
}
```

Legacy AABB zones \
(supported for easy migration, will be deprecated in a future release):
```json
{
  "type": "zone_start",
  "id": "ZONE#2",
  "origin": {"x": "0.0", "y": "0.0", "z": "32.0"},
  "min": {"x": "-32.0", "y": "-32.0", "z": "-32.0"},
  "max": {"x": "32.0", "y": "32.0", "z": "32.0"}
}
```

## ☰ In-Game Editor
Requires `ADMIN_CFG` access. \
Open the editor with:
- `say /zone`
- `zone`
- `box`

Editor menu actions:
- Create or remove zones
- Cycle zone function (type)
- Go to selected zone
- Toggle noclip
- Expand or shrink aimed wall by 1 unit

Anchor editing:
- Aim at an anchor and hold `+attack` (`MOUSE1`) to grab and drag it.
- While dragging a polygon vertex, hold `+use` (`E`) or `+reload` (`R`) to duplicate the vertex (add a new corner).
- Height anchors adjust the top of polygon zones.

Undo:
- While crouched in the editor, press `radio1` (`Z`) to undo the last change.

## ☰ Zone Type API
Zone types are registered by other plugins. Each type defines a class name, a description, a render color, optional visibility flags and callback names.

### Register a Type
```pawn
new Zone:g_start_zone;

public plugin_init()
{
    g_start_zone = sr_zone_register_type(
        .class_name = "zone_start",
        .description = "Start zone",
        .color = {0, 255, 0},
        .visibility = ZONE_VISIBLE_FULL,
        .on_enter = "on_start_enter",
        .on_leave = "on_start_leave"
    );
}

public on_start_enter(zone_entity, player_id) {}
public on_start_leave(zone_entity, player_id) {}
```

### Visibility Flags
- `ZONE_VISIBLE_NONE`
- `ZONE_VISIBLE_BOTTOM`
- `ZONE_VISIBLE_TOP`
- `ZONE_VISIBLE_WALLS`
- `ZONE_VISIBLE_FULL` (bottom + top + walls)

Use `sr_zone_set_visibility(zone, visibility)` or `sr_zone_get_visibility(zone)` to control per-type visibility.

## ☰ Global Forwards
These forwards fire for any zone type:
- `sr_zone_start_touch(zone_entity, player_id, zone_class)`
- `sr_zone_stop_touch(zone_entity, player_id, zone_class)`
- `sr_zone_touch(zone_entity, player_id, zone_class)`
- `sr_zone_created(zone_entity, zone_class)`
- `sr_zone_deleted(zone_entity, zone_class)`

## ☰ Limits
- Maximum polygon vertices per zone: 32
- Maximum zones tracked per player: 16

## Author
[PWNED](https://github.com/5z3f)
