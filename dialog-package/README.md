# Dialog

This feature provides a dialogue UI that displays text one character at a time.

---

## Usage

You can open the sample dialog using the **Alpha1** key.

---

## Features

### 1. Start Dialogue

By entering the desired `DialogId` into the `DialogLogic:StartDialog` method, the dialogue corresponding to the entered `DialogId` will start.

### 2. Move to the Next Line of Dialogue

Once the dialogue has been fully displayed, pressing the **Space** key will move to the next line.
If you press the Space key while the dialogue is still being displayed, the current line will finish, and you will be able to move to the next line.

### 3. Choose from Dialogue Options

Once the dialogue has been fully displayed, if there are any selectable options in that line, they will appear in the UI.
By selecting an option, the corresponding dialogue will be displayed.

---

## Installation

**1. UI Path Configuration**

You need to set the spawn paths for the dialogue UI.
The `ParentUIPath` can be modified in `DialogLogic`.
By default, it is set to `"/ui/DefaultGroup"`.

**2. Dialogue Data Configuration**

Currently, there is example dialogue data, but in order for the creator to create the desired dialogue, the data must be modified.

The `DialogDataSet` is a DataSet used for the dialogue, and the Dialog Data is a list-style table made up only of Dialog Script Data.

> e.g., `Dialog Data = {DialogScriptData1, DialogScriptData2, DialogScriptData3, ...}`

When storing multiple pieces of data in a single cell, use a delimiter to separate them.
The delimiter used is `DialogDataLogic.DataSetSplitSeparator`.

**3. @Nick Keyword**

If you want to display the current player's name in the dialogue or options, simply enter the `DialogDataLogic.NicknameMarker` keyword into the DataSet.

**4. Highlighting Specific Words**

If there is a sentence you want to emphasize when the dialogue is displayed, wrap the sentence with the same keyword as `DialogDataLogic.HighlightMarker` in the dialogue data.

- It's recommended to use a single character special symbol that is rarely used for `DialogDataLogic.HighlightMarker`.
- The highlighted color is controlled by `DialogDataLogic.HighlightHexcode`.

---

## Dialog Data Structure

| Field | Description |
|-------|-------------|
| `DialogId` | All Dialog Script IDs in a single dialog share the same `DialogId`. |
| `ScriptId` | The ID of the dialogue line. |
| `CharacterName` | The name of the character speaking. |
| `CharacterScript` | The dialogue of the character speaking. |
| `PortraitRUID` | The RUID of the portrait of the character speaking. |
| `SelectionScripts` | The text displayed in the dialogue options. Separate multiple options using a delimiter within a single cell. |
| `SelectionScriptIds` | The `ScriptId` of the next line of dialogue when a particular option is selected. Separate the ScriptIDs for each option with a delimiter within a single cell. |

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding! 🎉
