# MaplestoryToast

This package provides functionality similar to the toast feature in the game MapleStory.

---

## How to Use

You can use the three methods of the Logic called `MaplestoryToast`.

### `Show`

Displays the toast window. The previous toast content will disappear.

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `informSentence` | The text to display on the toast. If you insert a string identical to the `ArgsPattern` property inside the text, one element from `informArgs` will be inserted at that position. |
| `informArgs` | Used when you want to dynamically change parts of the text. The number of patterns in `informSentence` and the number of elements in `informArgs` must be the same. |
| `informColor` | The color of the text. |
| `duration` | The duration for which the toast will be displayed, excluding FadeIn and FadeOut time. |
| `decorationRUIDs` | The Sprite RUID table for the toast: `{Left image, Center image, Right image, Portrait}` |
| `useFading` | When `true`, FadeIn and FadeOut effects will be applied. If `false`, the toast will appear immediately and disappear instantly after the duration. |

### `ChangeArgs`

Replaces the arguments in the previous `informSentence` with new ones.

**Parameters:**

| Parameter | Description |
|-----------|-------------|
| `informArgs` | Used when you want to dynamically change parts of the text. The number of patterns in `informSentence` and the number of elements in `informArgs` must be the same. |

### `Hide`

Immediately hides the toast window.

---

## Example

The toast is triggered by a KeyEvent in the Logic called `PressToNotify`.

| Key | Description |
|-----|-------------|
| **Alpha1** | Displays the toast with a FadeIn effect. After the exposure time ends, the toast will fade out and disappear. |
| **Alpha2** | Displays the toast immediately. After the exposure time ends, it will disappear instantly. |
| **Alpha3** | Changes only the arguments of the previously set text and displays it. The options will be applied based on the values shown right before. |
