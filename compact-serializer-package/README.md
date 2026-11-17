# CompactSerializer

This package provides a serializer that turns supported Lua values into compact strings and restores them.

---

## Features

1. Turns supported Lua values into one compact string and restores them.
2. A list of tables with the same fields stores each field name once and the values column by column.
3. Key types and the integer / float distinction are preserved.
4. Compression is a separate pair of methods that works on any string, so you choose when to use it.
5. If any value is unsupported, serialization fails without producing partial output and logs the failing path.

| Script | Role |
|---|---|
| `Core/CompactSerializerLogic` | The only entry point for callers. |
| `Core/CompactSerializerWriter` / `CompactSerializerReader` | Do the work of `Serialize` / `Deserialize`. Internal. |
| `Core/CompactSerializerCodec` | The rules of the serialized string, shared by both. Internal. |
| `Core/CompactSerializerCompressor` | LZW compression behind `Compress` / `Decompress`. Internal. |
| `Core/CompactSerializerColumnLayout` | A struct the internal scripts pass to each other. Internal. |

Methods run in the caller's execution space because they do not declare `@ExecSpace`.

---

## Installation

After importing the package, nothing needs to be configured: `_CompactSerializerLogic` is ready to call.

Sample (optional)
The sample needs nothing placed or attached: play and press `F2` or `F3` (see **Sample** below).

---

## Usage

```lua
local data = {
	Level = 12,
	Position = "1,2,0",   -- an engine value converted to a string (see Rules)
	Items = { { Id = 1001, Count = 1 }, { Id = 2001, Count = 20 } },
}

-- Table to string
local text = _CompactSerializerLogic:Serialize(data)

if text ~= nil then
	-- Keep or send the string wherever a string is accepted.
end

-- String back to table
local restored = _CompactSerializerLogic:Deserialize(text)
```

### CompactSerializerLogic

The methods below are the whole API. Call them as `_CompactSerializerLogic:Serialize(...)`; every other script in the package is internal.

| Method | Description |
|---|---|
| `string Serialize(any value)` | Converts `value` into one string. Returns `nil` on failure. |
| `any Deserialize(string data)` | Restores the value from a string `Serialize` returned. Returns `nil` on failure. |
| `string Compress(string text)` | Compresses any string. Returns `nil` if `text` is not a string. |
| `string Decompress(string data)` | Restores the string `Compress` returned. Returns `nil` on failure. |

Compression is up to the caller: `Compress` always compresses, and a compressed string must go through `Decompress` before `Deserialize`. Compression pays off for large data, such as a long list of items; small data can come out longer, so compare the lengths if it matters. What can be serialized, and what happens when something cannot, is under **Rules** below.

### Using with DataStorage

A DataStorage value is a string, so save data can be stored as the string `Serialize` returns. DataStorage credit is charged by the size of the value, in units of 4,000 bytes, so a shorter string costs less; save data with long lists is usually worth compressing.

```lua
local storage = _DataStorageService:GetUserDataStorage(self.Entity.PlayerComponent.ProfileCode)

-- Save
local raw = _CompactSerializerLogic:Serialize(self.SaveData)

if raw ~= nil then
	local errorCode = storage:SetAndWait("SaveData", _CompactSerializerLogic:Compress(raw))
end

-- Load
local errorCode, raw = storage:GetAndWait("SaveData")

if errorCode == 0 and not _UtilLogic:IsNilorEmptyString(raw) then
	local loaded = _CompactSerializerLogic:Deserialize(_CompactSerializerLogic:Decompress(raw))
end
```

---

## Rules

**Supported** — `nil`, `boolean`, `integer`, `number`, `string`, and plain tables of these (keys `string` / `number` / `boolean`).

**Not supported** — NaN, engine values (`Vector2`, `Color`, `Quaternion`, `DateTime`, `FastVector2`, …), `Entity` / `Component` references, functions, `@Struct` instances and other tables with a metatable, circular references, tables nested deeper than 63 levels.

**Other values** — Convert anything that is not a supported value before it goes into the table, and convert it back after `Deserialize`. Write an engine value as a string of its fields, such as `"1,2,0"` for `Vector3(1, 2, 0)` (`string.format("%.9g", field)` keeps a 32-bit float field exact), and store an id instead of an `Entity` or `Component`.

**On failure** — `Serialize` and `Deserialize` return `nil` and log the cause with a path such as `root.inventory.3.owner`. Because a serialized `nil` also comes back as `nil`, do not use the return value alone to decide whether `Deserialize` succeeded.

**Structs** — Convert `@Struct` data to plain tables before serializing: give the struct a `Serialize()` method that returns a table and a `Deserialize(table)` method that fills it back in. A list of same-shaped tables is written column by column, so it stays small.

**Output order** — The order of dictionary keys and of column-layout columns is not fixed, so the same data can produce different strings. Compare the data, not the strings, to tell whether it changed.

**Size** — Shorter field names make a shorter string. Compression takes longer the longer the string is, and the frame is held until it finishes. For large data, skip `Compress` or split the data into several strings.

---

## Format

The serialized string is a one-item JSON array, `[data]`; the array is needed because `JSONEncode` takes a table. Values that cannot be written in the shapes below (empty tables, infinity, …) are written as a string starting with `~`.

**Basic values**

| Type | Written as | Example |
|---|---|---|
| `nil` | An empty array | `[]` |
| `boolean` | `true` / `false` without quotes; `"~t"` / `"~f"` only as dictionary keys | `true` |
| `integer` / `number` | The number without quotes (`"~205"`-style strings only as dictionary keys). `math.huge` / `-math.huge` become `"~+"` / `"~-"`; NaN is refused | `3`, `-0.639`, `"~+"` |
| `string` | `"..."` in double quotes. One more `~` is added when it starts with `~` | `"hi"`, `"~~tilde"` |

**Tables**

| Shape | Written as | Example |
|---|---|---|
| Array-shaped (keys counting up from 1) | `[value,value,...]`; keys are not written, each value's position is its key | `[1,"two"]` |
| Dictionary-shaped (any other keys) | `{"key":value,...}`; keys are always quoted strings (see below) | `{"name":"a","~5":"b"}` |
| Empty table | `"~e"` | `"~e"` |

Dictionary keys keep their type: strings as written (`"~~..."` when they start with `~`), integers `"~205"`, floats `"~2.5"`, booleans `"~t"` / `"~f"`, ±`math.huge` `"~+"` / `"~-"`. A table of plain tables, such as a list of items, is written column by column instead (see **Column layout** below).

**Column layout** — a table of two or more values that are all plain tables with only string keys. Each value is a row; the rows do not need the same keys. The key list is written once, then one column per key holding every row's value; `"~"` is an empty cell for a key the row lacks. The column layout is used only when it is shorter: tables that share few keys, or have only a couple of rows, are written as a plain array or dictionary, and so is a table at the two deepest nesting levels.

| Shape | Written as | Example |
|---|---|---|
| Array-shaped (`~r`, rows by position) | `["~r", [keys], column, ...]` | `["~r", ["count","id"], ["p",2,"~"], ["d",1001,1]]` |
| Dictionary-shaped (`~d`, rows by key) | `["~d", [keys], rowKeyColumn, column, ...]` | `["~d", ["progress","state"], ["d",5228,-18], ["p",6,0], ["y",["done"],0,0]]` |

Each column uses the shortest of three encodings (ties keep the earlier one):

| Tag | Written as | Used for |
|---|---|---|
| `p` | `["p", value...]` | Anything; always used for columns holding nested tables |
| `d` | `["d", first, difference...]` | Integer columns, such as slots or ids that count up |
| `y` | `["y", [dictionary], index...]` (0-based) | Few distinct values, such as item ids, states or booleans |

Cells are written like any other value: nested tables in their own shape, and empty cells as `"~"`.

**Compression** — `Compress` LZW-compresses any string and writes the result in URL-safe base64 characters (`A-Z a-z 0-9 - _`), so it can be stored or sent anywhere a plain string can. `Decompress` reverses it. A compressed string never starts with `[`, so `Deserialize` refuses it with a warning to call `Decompress` first.

---

## Sample

### `Sample/CompactSerializerSampleLogic.mlua`

A Logic that tests serialization during play; it stores nothing. Play and press `F2` or `F3`.

**`F2` — struct data.** Each run serializes the sample data, restores it from the string, and logs the string (in full when uncompressed, as sizes when compressed) and the restored values. The restored copy replaces the sample data, and every run changes `RunCount`, `LastPosition` and the first item's `Level`, so the next run shows that the values survived the round trip.

The data is held in two `@Struct`s, `Sample/CompactSerializerSampleSaveData` and `Sample/CompactSerializerSampleItem`. Each has `Serialize()` and `Deserialize(table)`, the conversion described under **Structs** above; `LastPosition` (a `Vector3`) is written there as an `"x,y,z"` string, as described under **Other values**.

| Property | Default | Description |
|---|---|---|
| `Compress` | `true` | `true` passes the serialized string through `Compress` and `Decompress`. |
| `ItemCount` | `50` | How many items the sample data holds. With only a few (try `3`), the compressed string comes out longer than the uncompressed one. |

**`F3` — your own table.** Serializes the plain table returned by `CreateTableData()`; edit the table in that method to try any data. The log shows the serialized string, its length with `Serialize` and after `Compress`, the length `JSONEncode` would give for the same table (or that it cannot write it), and whether the restored value equals the original.

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding!
