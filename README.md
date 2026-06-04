# CrazyTrain Trainers

Community-maintained trainer definitions for [CrazyTrain](https://github.com/pixelsoul/CrazyTrain).

## Contributing a Trainer

1. Fork this repo
2. Add a new file to the `trainers/` folder named `your-game-title.json`
3. Follow the schema below
4. Open a pull request

When your PR is merged into `main`, the `dist/trainers.json` is rebuilt automatically and served via CDN within minutes.

## Trainer Schema

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
          "address": "0xDEADBEEF",
          "offsets": [16, 40, 128]
        }
      ]
    }
  ]
}
```

### Field Reference

| Field                    | Type                       | Description                                                 |
| ------------------------ | -------------------------- | ----------------------------------------------------------- |
| `gameId`                 | string                     | Unique identifier (kebab-case, matches the app's game list) |
| `title`                  | string                     | Display name shown in the UI                                |
| `processName`            | string                     | Exact `.exe` filename the game runs as                      |
| `mods[].name`          | string                     | Display name for the mod                                    |
| `mods[].type`          | `"boolean"` \| `"integer"` | Whether this is a toggle or a value                         |
| `mods[].defaultHotkey` | string                     | Default key binding (e.g. `"F1"`, `"Numpad0"`)              |
| `mods[].enableValue`   | number[]                   | Bytes written when enabled (default `[1]`)                  |
| `mods[].disableValue`  | number[]                   | Bytes written when disabled (default `[0]`)                 |
| `mods[].memoryOffsets` | array                      | Base address + pointer chain offsets                        |

## How Memory Offsets Work

- `address`: The base pointer address in hex (e.g. `"0x1A2B3C4D"`)
- `offsets`: Array of pointer offsets to dereference in order to reach the final value

Use tools like Cheat Engine to find these values for a specific games version.
