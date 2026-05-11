# JSON Rail Importer

Imports a Maze Builder JSON layout into the current Unreal Engine level.

## What It Does

- Reads `web-maze-builder/maze_layout.json` by default.
- Reads rail references from `web-maze-builder/rail_config.csv`.
- Places generated rail actors into the `GeneratedMazeRails` folder.
- Can clear previously generated actors in that folder.
- Prefers Blueprint/Class references when configured.
- Falls back to Static Mesh references from rail part columns such as `Side`, `BR`, `BL`, `B`, `L`, and `R`.

## Run

Run inside Unreal Editor Python:

```python
exec(open(r"/Users/ddonlien/Documents/GitHub/ball-maze-tools/ue-json-rail-importer/import_json_rails_to_level.py", encoding="utf-8").read())
```

This script cannot run in normal system Python because it imports `unreal`.

## Key Config

Edit the top of `import_json_rails_to_level.py` when needed:

- `DEFAULT_LAYOUT_JSON`: layout JSON to import.
- `DEFAULT_RAIL_CONFIG_CSV`: rail reference CSV.
- `FOLDER_PATH`: folder used for generated actors.
- `CLEAR_EXISTING_IN_FOLDER`: whether to delete old generated actors first.
- `SELECT_SPAWNED_ACTORS`: whether to select imported actors after placement.
- `GRID_TO_WORLD`, `LOCATION_SCALE`, `LOCATION_OFFSET`, `ROTATION_OFFSET`: placement transform settings.
- `BP_MAZE_CLASS_PATH`, `BP_MAZE_BOUNDARY_CLASS_PATH`, `BP_MAZE_BOTTOM_CLASS_PATHS`: optional project-side Blueprint classes.

## Notes

Maze Builder JSON is the data contract. If a rail imports with the wrong asset, first check `rail_config.csv` references and the importer fallback columns before changing generated JSON.
