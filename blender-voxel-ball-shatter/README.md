# Voxel Ball Shatter

Blender Python script for converting selected mesh objects into voxel cube collections. It is useful for turning a voxel-like ball or mesh into separate cube pieces that can be edited, assigned materials, or exported as individual objects.

## What It Does

- Runs on selected Blender Mesh objects.
- Uses a fixed `VOXEL_SIZE` and snaps cube centers to a stable voxel grid.
- Creates a new `<object>_shattered` collection for each source object.
- Reconstructs voxel occupancy from axis-aligned source faces.
- Can fill the volume with scanlines, then keep only the outer layer.
- Can merge unit voxels into larger non-overlapping cuboid blocks.
- Can randomize merged block sizes while trying to stay under a target output block count.
- Can use the evaluated mesh after modifiers.
- Preserves material slots, UV layers, and mesh Color Attributes / vertex colors used by material nodes.
- Creates the shattered collection under the source object's collection.
- Guards against accidental huge output with `MAX_OUTPUT_CUBES_PER_OBJECT`.

## Run

1. Open Blender.
2. Select one or more Mesh objects.
3. Open `voxel-ball-shatter.py` in Blender's Text Editor.
4. Adjust the config block if needed.
5. Click `Run Script`.

You can also run it from Blender Python:

```python
exec(open(r"/Users/ddonlien/Documents/GitHub/ball-maze-tools/blender-voxel-ball-shatter/voxel-ball-shatter.py", encoding="utf-8").read())
```

This script cannot run in normal system Python because it imports `bpy` and Blender math/geometry APIs.

## Key Config

- `VOXEL_SIZE`: size of one voxel cube. The current default is `0.1`.
- `GRID_SNAP_STEP`: boundary snap precision. For `0.1` voxels, `0.05` is usually correct; use `0.0125` if the source asset has that offset.
- `FILL_VOLUME_BY_SCANLINE`: fills the voxel volume before extracting the outer layer.
- `SCANLINE_AXIS`: scan direction for filling, where `0 = X`, `1 = Y`, `2 = Z`.
- `OUTER_DIRECTIONS_MODE`: `SIX` checks all six neighbors; `FIVE_NO_BOTTOM` ignores bottom exposure.
- `USE_EVALUATED_MESH`: uses modifier-evaluated geometry when `True`.
- `REPLACE_EXISTING_COLLECTION`: replaces an existing `<object>_shattered` collection when rerunning.
- `LINK_TO_ALL_SOURCE_COLLECTIONS`: links the output collection under every source collection when `True`.
- `MAX_OUTPUT_CUBES_PER_OBJECT`: safety limit for generated cube count.
- `MERGE_LEVEL`: merge strength. `1` keeps unit voxels, `2` allows two-voxel bars, `3` allows `2x2x2` blocks, higher values allow larger blocks.
- `RANDOMIZE_MERGE`: enables random merged block sizes.
- `RANDOM_OUTPUT_BLOCK_LIMIT`: target maximum number of output block objects when random merge is enabled.
- `RANDOM_SEARCH_MAX_MERGE_LEVEL`: upper search bound for automatic random merge sizing.
- `RANDOM_SEED`: makes randomized merging repeatable.
- `RANDOM_VOLUME_BIAS`: higher values favor larger random blocks.
- `RANDOM_ATTEMPTS`: retry count when trying to stay below `RANDOM_OUTPUT_BLOCK_LIMIT`.
- `RANDOM_MAX_MERGE_SIDE`: hard cap for random block side length. Lower values are much safer in Blender's main thread.
- `RANDOM_MAX_CANDIDATE_SHAPES`: limits how many merge shapes are tested per random run.
- `RANDOM_MAX_BLOCKS_PER_SEED`: limits accepted candidate blocks per seed voxel.
- `RANDOM_MAX_ORIGINS_PER_SHAPE`: limits random origin checks for each shape.
- `RANDOM_EFFECTIVE_ATTEMPTS_CAP`: safety cap on actual random merge passes, even if `RANDOM_ATTEMPTS` is higher.
- `RANDOM_MAX_SEEDS_TO_SEARCH`: safety cap before remaining voxels fall back to `1x1x1`.
- `PREFER_CUBE_LIKE_BLOCKS`: prioritizes cubes and near-cubes over long bars.
- `AXIS_NORMAL_THRESHOLD`: ignores faces that are not close to axis-aligned.

## Notes

The script assumes voxel-style geometry with mostly axis-aligned faces. If a material reads Color Attributes such as `Color`, `Metallic`, `Roughness`, `Emission`, or similar, the generated block meshes create matching CORNER-domain Color Attributes and sample values from the source faces.

Merged blocks are single Blender objects, but their exterior faces remain subdivided on the source voxel grid so material slots, UVs, and Color Attributes can still be sampled per small face.
