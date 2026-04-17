# PlayerData Package

This PlayerData package is an integrated module that handles essential player management functions within the game.

---

## Features

### 1. Loading Screen Support

Manages the loading screen displayed when players join the game, ensuring a smooth entry experience.

### 2. Player Kick and Ban System

Allows administrators to kick or ban players using multiple ProfileCodes, enabling effective moderation of inappropriate behavior.
Ban information is stored and managed in `GlobalDataStorage`.

### 3. Player Data Save and Load Component

The `PlayerData` component securely saves and retrieves players' basic information as needed.
Player information is stored and managed in `UserDataStorage`.

### 4. Admin Tool for Data Modification

Provides a tool for administrators to modify players' basic information directly, allowing flexible management and operation.

---

## Recommendation

It is recommended to set `PlayerEntityAuthorityCheck` to `true` in `WorldConfig`.
If this setting is false, the server function is exposed to all clients, which may lead to security vulnerabilities.

---

## Installation

This package is available for use after import, but some initial setup is required.

**1. Player Components**

The following components need to be added to the `DefaultPlayer`:
- `PlayerDBManager`, `PlayerData`

**2. Admin Permissions Check for Tool Usage**

You need to check if the user has admin permissions using the `AdminLogic:IsAdmin` function.

**3. UI Path Configuration**

You need to set the spawn paths for the message UI and the tool UI.
The UI Path can be modified in `PlayerDataSampleUILogic`, `PlayerBanSampleUILogic`, `PopupSampleUILogic`.

**4. DataStorage Name Configuration**

The names and tags you are using might overlap with existing ones.
You can modify the `storageName` in `GMPlayerDataToolLogic`, `GMPlayerBanToolLogic`.

**5. UIGroup Initialization**

Create a new UIGroup and rename it to `'SystemGroup'`.
Add the `UILoading` Model to this UIGroup.
This UIGroup is used in `LoadingSampleUILogic` and `PopupSampleUILogic`, and its name can be changed.

---

## Player Data Management System

### Overview of PlayerDBManager and PlayerData Integration

`PlayerDBManager` is responsible for loading and saving data used by components like `PlayerData` from/to `DataStorage`.
Creators can add new components similar to `PlayerData` with similar functionality.

`PlayerDBManager` calls the following four functions in `PlayerData`:

1. `LoadFromDB`
2. `OnLoadedDataFromDB`
3. `PostOnLoadedDataFromDB`
4. `SaveToDB`

Newly added components should implement these functions as well so that `PlayerDBManager` can call them automatically for integration.

### PlayerData Internal Handling

`PlayerData` uses the `PlayerBasicInfo` struct to serialize and deserialize data.
It uses `IsSaveDB` (DirtyFlag) to check if data has changed, and only saves to `DataStorage` when modifications occur.
Data saving is performed periodically by `PlayerDBManager`.

---

## Timing of Loading Screen Removal

### Loading Screen Removal Process

1. Server data loads first, followed by client data loading.
2. Once client data loading is complete, the client requests player information from the server (via `NotifyWorldDataLoadCompleted`).
3. Player information is then loaded and synchronized between client and server.
4. When player data initialization finishes successfully on both sides, the loading screen is removed.

### Error Handling

- If an issue occurs with the player's data during this process, that specific player will be kicked.
- If there is a problem loading the world data, all players will be kicked.

---

## PlayerData Tool Usage

Press **F3** to open the tool UI, which allows you to manage player data.

- You can use this tool to modify `PlayerData`. The data that makes up `PlayerData` is only an example, and you are free to add various other data as needed.
- You can set the time offset using the **"User Time Offset"** option in the top-left corner.
- User data should be safely modified only when the user is **offline**.

---

## PlayerBan Tool Usage

Press **F4** to open the tool UI, which allows you to manage player ban information and kick players.

You can set the time offset using the **"User Time Offset"** option in the top-left corner.

### About Ban

When a player enters, their ban information is checked first, and the ban is enforced if applicable.
Banned users will see a popup message and will be automatically kicked after a specified duration.
If the player is not banned, their information is loaded from DataStorage and initialized.

- **Scope**: All spaces (world instances and rooms)
- Ban can be set with a start and end time.
- In the **Edit Panel**, you can modify ban information for individual users.
- Use **BulkEdit** to update the ban information for multiple users simultaneously.

| Field | Description |
|-------|-------------|
| `Message` | Popup message displayed to the user (typically the ban reason). |
| `Extra` | Administrator notes about the user. Not visible to the user. |

### About Kick

- **Scope**: All spaces (world instances and rooms)
- Kick information is **not** stored in DataStorage.
- When a kick is requested, the target user will see the popup message written by the administrator. After a specified duration, the user will be automatically kicked.
- If **Direct Kick** is used, the user will be immediately kicked from the server without displaying a popup message.

---

## Loading UI Test Instructions

Press **F5/F6** to test fade-in and fade-out transitions on the loading screen.
Script location: `LoadingSampleUILogic`.

---

## Cautions

If too many users are banned or kicked at once, it can cause server overload.
Additionally, it may excessively consume DataStorage credits.

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding! 🎉
