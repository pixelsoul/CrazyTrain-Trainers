# CrazyTrain Trainers

Community-maintained trainer definitions for [CrazyTrain](https://github.com/pixelsoul/CrazyTrain).

## Contributing a Trainer

1. Fork this repo
2. Add a new file to the `trainers/` folder named `your-game-title.json`
3. Follow the schema below
4. Open a pull request

When your PR is merged into `main`, the `dist/trainers.json` is rebuilt automatically and served via CDN within minutes.

## Trainer Schema

There are two ways to specify a memory location for a mod — **static address** or **AOB signature**. AOB is preferred because it survives game updates; static addresses break whenever the game is patched.

### Static address

```json
{
  "gameId": "unique-kebab-case-id",
  "title": "Game Title",
  "processName": "game.exe",
  "mods": [
    {
      "name": "Mod Name",
      "type": "boolean",
      "defaultHotkey": "F1",
      "enableValue": [1],
      "disableValue": [0],
      "memoryOffsets": [
        {
          "module": "game.exe",
          "address": "0x1A2B3C",
          "offsets": [16, 40, 128]
        }
      ]
    }
  ]
}
```

### AOB signature (preferred)

```json
{
  "gameId": "unique-kebab-case-id",
  "title": "Game Title",
  "processName": "game.exe",
  "mods": [
    {
      "name": "Mod Name",
      "type": "boolean",
      "defaultHotkey": "F1",
      "enableValue": [1],
      "disableValue": [0],
      "memoryOffsets": [
        {
          "module": "game.exe",
          "address": "0x00000000",
          "offsets": [],
          "aobSignature": "48 89 5C 24 ?? 57 48 83 EC 20 48 8B D9",
          "aobOffset": 0
        }
      ]
    }
  ]
}
```

### Field Reference

| Field                              | Type                       | Description                                                                        |
| ---------------------------------- | -------------------------- | ---------------------------------------------------------------------------------- |
| `gameId`                           | string                     | Unique identifier (kebab-case, matches the app's game list)                        |
| `title`                            | string                     | Display name shown in the UI                                                       |
| `processName`                      | string                     | Exact `.exe` filename the game runs as                                             |
| `mods[].name`                      | string                     | Display name for the mod                                                           |
| `mods[].type`                      | `"boolean"` \| `"integer"` | Whether this is a toggle or a value                                                |
| `mods[].defaultHotkey`             | string                     | Default key binding (e.g. `"F1"`, `"Numpad0"`)                                     |
| `mods[].enableValue`               | number[]                   | Bytes written when enabled (default `[1]`)                                         |
| `mods[].disableValue`              | number[]                   | Bytes written when disabled (default `[0]`)                                        |
| `mods[].memoryOffsets`             | array                      | One or more memory location descriptors                                            |
| `memoryOffsets[].module`           | string                     | Module (`.exe` / `.dll`) the address is relative to                                |
| `memoryOffsets[].address`          | string (hex)               | Static offset from the module base (e.g. `"0x1A2B3C"`). Ignored when `aobSignature` is set |
| `memoryOffsets[].offsets`          | number[]                   | Pointer-chain offsets dereferenced after the base address is resolved              |
| `memoryOffsets[].aobSignature`     | string *(currently optional, will be required)* | Space-separated hex bytes scanned at runtime to locate the target; `??` matches any byte (e.g. `"48 8B ?? ?? 00"`) |
| `memoryOffsets[].aobOffset`        | number *(currently optional, will be required)* | Byte offset added to the AOB match address before writing (default `0`). Use when the signature points to the start of an instruction and the patch target is a few bytes in |

## How Memory Addressing Works

When `aobSignature` is present, CrazyTrain scans the process memory at runtime for the first occurrence of that byte pattern (restricted to the named `module` if one is specified), then adds `aobOffset` to arrive at the exact write target. The `address` field is ignored in this path.

When `aobSignature` is absent, `address` is treated as a hex offset from the named module's base load address (or as an absolute address if `module` is omitted), and `offsets` is a pointer chain dereferenced on top of that.

Use tools like Cheat Engine or x64dbg to find signatures and offsets for a specific game version. AOB signatures should be long enough to be unique in the module (~12–16 bytes) but short enough that a minor patch won't change every byte.
