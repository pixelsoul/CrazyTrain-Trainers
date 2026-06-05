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
      "comment": "Optional tooltip shown in the UI — describe side-effects or caveats here.",
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

## Mod Sections

The `mods` field can be either a flat array (as above) or a named-section object. Use sections to group related mods under labelled headers in the UI — useful for trainers with many mods.

```json
{
  "gameId": "unique-kebab-case-id",
  "title": "Game Title",
  "processName": "game.exe",
  "mods": {
    "Player": [
      {
        "name": "Infinite Health",
        "comment": "Sets HP to max — resets on damage if the game recalculates HP each frame.",
        "type": "boolean",
        "defaultHotkey": "F1",
        "enableValue": [255],
        "disableValue": [100],
        "memoryOffsets": [
          {
            "module": "game.exe",
            "address": "0x00000000",
            "offsets": [],
            "aobSignature": "48 89 5C 24 ?? 57 48 83 EC 20",
            "aobOffset": 0
          }
        ]
      }
    ],
    "Weapons": [
      {
        "name": "Infinite Ammo",
        "type": "boolean",
        "defaultHotkey": "F2",
        "enableValue": [1],
        "disableValue": [0],
        "memoryOffsets": [
          {
            "module": "game.exe",
            "address": "0x00000000",
            "offsets": [],
            "aobSignature": "F3 0F 5C CE 48 8D 8D 28",
            "aobOffset": 0
          }
        ]
      }
    ]
  }
}
```

Section keys are displayed as labelled dividers in the UI. When `mods` is a flat array, no section headers are shown and behaviour is identical to previous versions.

## Field Reference

### Top-level

| Field         | Type             | Description                                                    |
| ------------- | ---------------- | -------------------------------------------------------------- |
| `gameId`      | string           | Unique identifier (kebab-case, matches the app's game list)    |
| `steamAppId`  | number           | Steam App ID — used to match against the installed game library |
| `title`       | string           | Display name shown in the UI                                   |
| `processName` | string           | Exact `.exe` filename the game runs as                         |
| `mods`        | `Mod[]` or `Record<string, Mod[]>` | Flat array of mods, or a named-section object grouping mods under labelled headers |

### Mod fields

| Field                   | Type                                    | Description                                                                                           |
| ----------------------- | --------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `name`                  | string                                  | Display name for the mod                                                                              |
| `comment`               | string *(optional)*                     | Extra context shown as a tooltip on the info icon in the UI — describe side-effects or caveats        |
| `type`                  | `"boolean"` \| `"integer"` \| `"inject"` | `boolean` = toggle bytes; `integer` = write a numeric value; `inject` = JMP hook with trampoline      |
| `dataType`              | string *(optional)*                     | Data type hint for the value being patched (e.g. `"double"`, `"float"`, `"int"`). Informational only. |
| `defaultHotkey`         | string                                  | Default key binding (e.g. `"F1"`, `"Numpad0"`)                                                        |
| `enableValue`           | number[]                                | Bytes written when enabled (default `[1]`)                                                            |
| `disableValue`          | number[]                                | Bytes written when disabled (default `[0]`)                                                           |
| `patchSize`             | number *(inject only)*                  | Number of bytes to overwrite at the injection point with the JMP hook. Minimum 5; minimum 14 if a near allocation is unavailable. |
| `trampolineCode`        | string *(inject only)*                  | Space-separated hex bytes of the trampoline body (e.g. `"0F 11 4B 10 41 55"`). Do **not** include a JMP-back — the backend appends it automatically. |
| `memoryOffsets`         | array                                   | One or more memory location descriptors (see below)                                                   |

### memoryOffsets fields

| Field                              | Type                       | Description                                                                        |
| ---------------------------------- | -------------------------- | ---------------------------------------------------------------------------------- |
| `module`                           | string                     | Module (`.exe` / `.dll`) the address is relative to                                |
| `address`                          | string (hex)               | Static offset from the module base (e.g. `"0x1A2B3C"`). Ignored when `aobSignature` is set |
| `offsets`                          | number[]                   | Pointer-chain offsets dereferenced after the base address is resolved              |
| `aobSignature`                     | string *(optional)*        | Space-separated hex bytes scanned at runtime to locate the target; `??` matches any byte (e.g. `"48 8B ?? ?? 00"`) |
| `aobOffset`                        | number *(optional)*        | Byte offset added to the AOB match address before writing (default `0`). Use when the signature points to the start of an instruction and the patch target is a few bytes in |

## How Memory Addressing Works

When `aobSignature` is present, CrazyTrain scans the process memory at runtime for the first occurrence of that byte pattern (restricted to the named `module` if one is specified), then adds `aobOffset` to arrive at the exact write target. The `address` field is ignored in this path.

When `aobSignature` is absent, `address` is treated as a hex offset from the named module's base load address (or as an absolute address if `module` is omitted), and `offsets` is a pointer chain dereferenced on top of that.

Use tools like x64dbg or similar to find signatures and offsets for a specific game version. AOB signatures should be long enough to be unique in the module (~12–16 bytes) but short enough that a minor patch won't change every byte.
