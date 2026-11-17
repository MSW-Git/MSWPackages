# ColorPicker

This package provides an HSV color picker UI that reports the selected color through callbacks.

---

## Features

1. A saturation/value pad, a hue slider, numeric H/S/V and R/G/B/A fields, and a hex field all edit the same color and stay synchronized.
2. Two previews show the current color beside the color the picker was opened with.
3. Every edit invokes `onChanged`, so callers can preview the color live.
4. Confirming invokes `onConfirmed`; canceling invokes `onCanceled`, where the caller restores the preview.
5. `ColorUtilLogic` provides the color conversion methods the picker uses (HSV / RGB / `Color` / hex), usable on their own.

| Script / Model | Role |
|---|---|
| `Core/ColorPickerLogic` | Public entry point. Spawns the picker on the first `Show` and reuses it afterwards. |
| `Core/ColorPicker` | Internal. Component on the picker UI: finds its parts through the model's groups, draws the pad and slider, handles input, and invokes the callbacks. |
| `Core/ColorUtilLogic` | Logic singleton with the color conversion methods (HSV / RGB / `Color` / hex / RGBA int). Holds no state. |
| `Core/ColorTypeStructs/ColorRGB`, `Core/ColorTypeStructs/ColorHSV` | Structs the conversion methods take and return. |
| `Core/Model_ColorPicker` | The picker UI model spawned by `ColorPickerLogic`. |

The `ColorPicker` component is internal to the package. Do not call it directly; use `ColorPickerLogic`. Its methods are not part of the API and are not documented here.

`ColorPickerLogic` is `ClientOnly`; call it from client code. `ColorUtilLogic` has no ExecSpace and works on either side.

---

## Installation

After importing the package, verify these settings.

1. UI parent
The picker is spawned under `ParentGroupEntity` in `ColorPickerLogic`.
It is empty by default, so the picker goes under `/ui/DefaultGroup`. Set it to the UI group the picker should appear in.

2. Picker model
`PickerModelId` in `ColorPickerLogic` is empty by default, so the model named `Model_ColorPicker` (`Core/Model_ColorPicker`) is looked up by name and spawned.
If you copy the model to customize it, set this to the copy's model id.

3. Model entities
The `ColorPicker` component finds its parts in `ResolveChildren` by walking the model's groups by name (`ColorPicker` → `ColorPanels` / `ColorPreviews` / `InputFields` / `Buttons` → `SVPad`, `HueInput`, `OKButton`, ...).
If you edit the model, update `ResolveChildren` to match it.

---

## Usage

```lua
local original = self.SpriteGUIRenderer.Color
_ColorPickerLogic:Show(
	original,
	function(color) self.SpriteGUIRenderer.Color = color end,
	nil,
	function() self.SpriteGUIRenderer.Color = original end
)
```

This previews every edit on the sprite and puts it back to the original color when canceled.

| Argument | Description |
|---|---|
| `initialColor` | Initial color. `nil` opens at the default white; the previous session's color is not carried over. |
| `onChanged(Color color)` | Called on every edit. |
| `onConfirmed(Color color)` | Called with the chosen color when the confirm button is pressed. Use it for work that should happen only on confirm, such as saving. |
| `onCanceled()` | Called when the session is canceled — the cancel button or `Hide`. The picker does not undo your preview; restore the original color here. |

### ColorPickerLogic

| Method | Description |
|---|---|
| `boolean Show(Color initialColor, any onChanged, any onConfirmed, any onCanceled)` | Opens the picker. Returns `false` when it is already open (the current session continues and these callbacks are dropped) or could not be created. |
| `void Hide()` | Closes the picker as a cancel. Does nothing when the picker is not open. |

| Property | Description |
|---|---|
| `PickerModelId` | Model spawned as the picker. Empty uses the model named `Model_ColorPicker`. |
| `ParentGroupEntity` | UI entity the picker is spawned under. Empty uses `/ui/DefaultGroup`. |
| `PickerEntity` | The spawned picker entity, cached and reused by later `Show` calls. If it has no `ColorPicker` component, `Show` returns `false`. |

### Input fields

| Field | Range | Invalid input |
|---|---|---|
| H | 0–360 | Keeps the current value |
| S / V | 0–100 | Keeps the current value |
| R / G / B / A | 0–255 | Keeps the current value |
| Hex | `RRGGBB` | Becomes `000000` |

The number fields take whole numbers only.
Numbers outside the range are clamped.

The hex field accepts an optional leading `#` and ignores surrounding spaces.
It always displays uppercase `RRGGBB` without alpha.
The hex field sets RGB only; alpha is set in the A field.
The field's `CharacterLimit` is 7, so `#RRGGBB` fits; without `#`, a 7th digit is ignored.
A short hex value is read as a number with its leading digits as 0, so `FF` becomes `0000FF`.

Only the edited field is applied.
Leaving a field with its value unchanged neither changes the color nor invokes `onChanged`.

The pad and slider respond to the left mouse button and to touch; the middle and right buttons are ignored.
When two or more touches land on them, only the first one changes the color.

### ColorRGB / ColorHSV

Value types the conversions exchange. Create one with `ColorRGB()` / `ColorHSV()` and set its fields.

| Struct | Fields | Defaults | Range |
|---|---|---|---|
| `ColorRGB` | `R` / `G` / `B` / `A` (`integer`) | 0 / 0 / 0 / 255 | 0–255. `A` 255 is opaque. |
| `ColorHSV` | `H` / `S` / `V` (`number`) | 0 / 0 / 1 | `H` 0–360, `S` / `V` 0–1. |

### ColorUtilLogic

| Method | Description |
|---|---|
| `ColorRGB HSVToRGB(ColorHSV hsv)` | `ColorHSV` (`H` wraps at 360) → `ColorRGB`, rounded. `A` is left opaque. |
| `ColorHSV RGBToHSV(ColorRGB rgb)` | `ColorRGB` → `ColorHSV`. An achromatic color returns `H = 0`, and black also returns `S = 0`. |
| `ColorRGB ColorToRGB(Color color)` | `Color` (0–1) → `ColorRGB`, clamped and rounded. |
| `Color RGBToColor(ColorRGB rgb)` | `ColorRGB` → `Color`. |
| `ColorRGB HexToRGB(string hex)` | `RRGGBB` (optional `#`, surrounding spaces, either case) → `ColorRGB` with `A` left opaque. Like Paint, a short value is read with its missing leading digits as 0 (`FF` → `0000FF`), digits past 6 are dropped, and a value that is not hex becomes `000000`. |
| `string RGBToHex(ColorRGB rgb)` | `ColorRGB` → uppercase `RRGGBB`. |
| `integer RGBToPixelInt(ColorRGB rgb)` | `ColorRGB` → `0xRRGGBBAA` for `PixelGUIRendererComponent.SetPixelsByRGBAInt`. |

```lua
local hsv = ColorHSV()
hsv.H = 210
hsv.S = 0.75
hsv.V = 0.8
local rgb = _ColorUtilLogic:HSVToRGB(hsv)        -- R 51, G 128, B 204, A 255
local hex = _ColorUtilLogic:RGBToHex(rgb)        -- "3380CC"
```

---

## Rules

**One shared picker** — Calling `Show` while the picker is open does nothing; the current session continues, and the later caller's callbacks are never called.

**Callback order** — The picker closes before `onConfirmed` and `onCanceled` run, so calling `Show` from inside them opens a new session, and calling `Hide` does nothing.

**Gray and black keep the hue** — Entering gray or white in the fields keeps the current hue; entering black keeps both the hue and the saturation, so raising the value brings the color back. Each `Show` resets hue and saturation to 0 before applying `initialColor`.

**Resolution** — The pad and slider draw at the `Width` / `Height` set in the model: 128×128 for the pad and 32×360 for the slider. The pad is redrawn on every hue change at a cost proportional to `Width * Height`, so a pad larger than 128×128 may drop frames while the hue slider is dragged.

---

## Sample

### `Sample/ColorPickerUsageSample.mlua`

A component that previews picker edits on a `SpriteGUIRendererComponent`. It keeps the last confirmed color, passes it to `Show` so the picker reopens at that color, and restores the sprite to it in `onCanceled`. `Show` and `Hide` are inspector buttons for testing in play mode.

| Property | Description |
|---|---|
| `SpriteGUIRenderer` | The renderer to tint. |
| `SelectedColor` | The last confirmed color. Starts as the sprite's color. |

### `Sample/Model_ColorPickerUsageSample`

A UI sprite with `ColorPickerUsageSample` already attached and bound to its own `SpriteGUIRendererComponent`. Place it, enter play mode, and press **Show** on the component in the inspector.

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding!
