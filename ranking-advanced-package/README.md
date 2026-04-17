# Ranking (Advanced)

This package provides a ranking feature, allowing creators to display ranking to users.

---

## Features

### 1. Ranking Features

Multiple rankings can be configured through DataSet.
Data required for rankings can be stored and accessed via `SortableDataStorage` and `UserDataStorage`.

### 2. Ranking UI

A ranking UI is provided to display the current ranking data to players.
Previous rankings are not accessible or viewable through the ranking UI.

### 3. Ranking Data Management Tool

Allows viewing and modifying ranking data and user data.

### 4. Score-Based Previous Ranking Rewards

Rewards can be distributed based on the scores from the previous ranking.
Rankings without a defined cycle (e.g., Day, Week) cannot distribute rewards.

---

## Recommendation

It is recommended to set `PlayerEntityAuthorityCheck` to `true` in `WorldConfig`.
If this setting is false, the server function is exposed to all clients, which may lead to security vulnerabilities.

---

## Installation

This package is available for use after import, but some initial setup is required.

**1. Player Components**

The following components need to be added to the `DefaultPlayer`:
- `PlayerDBManager`, `PlayerRanking`

**2. Admin Permissions Check for Tool Usage**

You need to check if the user has admin permissions using the `AdminLogic:IsAdmin` function.

**3. UI Path Configuration**

You need to set the spawn paths for the message UI and the tool UI.
The `UIPath`, `UIToolPath`, `UIHUDPath` can be modified in `RankingSampleUILogic`.
By default, they are set to `"/ui/DefaultGroup"`.

**4. DataStorage Name Configuration**

The names you are using might overlap with existing ones.
You can modify the `StorageName` in `RankingDataStorageLogic`.

**5. Ranking Config Settings**

Multiple rankings can be configured and managed through `RankingConfigDataSet`.

**6. UIHUD**

You can use the UIHUD model to conduct ranking registration tests.

---

## Usage

- Press **F2** to open the HUD UI, where players can register ranking.
  If the **Force** checkbox is selected, the score will be updated regardless of whether it is lower than or equal to the existing score.
- Press **F3** to open the UI, where players can view ranking.
  - **Force Update Button** — Allows administrators to manually update the SharedMemory, ensuring that ranking data is synchronized immediately.
  - **View Past Record Button** — Enables players to view their previous rankings. Rewards can be claimed based on the scores from past rankings.

---

## Tool Usage

Press **F4** to open the tool UI, which allows you to manage ranking data and user data.

You can set the time offset for your region using the **"User Time Offset"** option in the top-left corner of the tool.
You can retrieve and input data based on this time setting. It does not affect the already stored date data.

> User data should be safely modified only when the user is offline.

### View

Retrieve ranking data from SortableStorage using:

- `id` — Unique identifier for the ranking
- `cycleIndex` — Target cycle index
- `itemCount` — Number of ranking entries to retrieve

### Edit ViewData

View and edit ranking data in SortableStorage. Parameters:

- `id` — Unique identifier for the ranking
- `cycleIndex` — Target cycle index
- `ProfileCode` — Profile code to update

Modifiable fields: `Score`, `Tag`

### Edit UserData

View and edit `UserRankingData` in `UserDataStorage`. Query by `profileCode`.

| Field | Description |
|-------|-------------|
| `Id` | Ranking ID |
| `CycleIndex` | Ranking cycle index |
| `Score` | Ranking score, used for reward distribution |
| `UpdateTime` | Date when the ranking was last updated |
| `RewardTime` | Date when the reward was claimed (0 = not yet claimed) |
| `Extra` | Custom string defined by the creator |

---

## Ranking Data Structure

`PlayerRanking:SetScoreAndWait` is used to register rankings.

**1. SortableDataStorage & RankingData Struct**

Holds information required for displaying rankings.
To store more detailed information, Tags are used.

**2. UserDataStorage & UserRankingData Struct**

Contains more detailed ranking data.
Used as the basis for distributing score-based rewards.

---

## Shared Ranking Data Generation and Sharing Structure

**1. Data Generation**

A single world instance accesses `SortableDataStorage` to generate ranking data, then stores it in `WorldInstanceSharedMemory`.
The system does not generate rankings for previous `CycleIndex` values.

**2. Data Sharing**

Other world instances refer to data stored in `WorldInstanceSharedMemory` without directly accessing `SortableDataStorage`.
Each world instance has a different update interval, so ranking data is not updated in real-time.

**3. Data Accuracy**

Due to varying update intervals, data in `WorldInstanceSharedMemory` may not always be current.
Therefore, this package is not suitable for accurate rank-based rewards — score-based rewards are supported instead.

---

## Ranking Config DataSet

| Field | Description |
|-------|-------------|
| `Id` | A unique integer ID. |
| `Key` | Unique DataStorage keyName. Must not conflict with other rankings or DataStorage names. |
| `Name` | Simple name for identification purposes. |
| `CycleEnum` | Ranking cycle (Day, Week, Month, Year, ...). If not provided, no cycle exists. |
| `ViewCount` | Maximum users shown in ranking UI. Maximum is 1000 due to DataStorage Credit limits. |
| `RefreshIntervalMinutes` | Interval (in minutes) at which SharedMemory is refreshed. |
| `HasReward` | Whether score-based rewards are distributed. |
| `RankModeEnum` | Method used to calculate rankings (`Index`, `Rank`, `DenseRank`). |
| `MaxUserDataCount` | Maximum number of user data entries to retain. Older data is deleted first when the limit is exceeded. |
| `ReleaseBaseTime` | Release date of the ranking. `CycleIndex` is calculated based on this — do not change arbitrarily. |
| `Disable` | This data will not be loaded. |

---

## Cautions

### DataStorage Credit

Excessive use of DataStorage Credit may occur. Adjust these values:

1. Number of active rankings
2. `ViewCount`
3. `RefreshIntervalMinutes`

### RankingData Tag

Ranking data is stored in a single shared variable with a maximum size limit of **80,000 bytes**.
To avoid exceeding this limit, the length of Tags is restricted.
Since `|` is used as a delimiter in SharedMemory, it must **not** be included in Tags.

### Instance Room

User ranking queries are not available in the instance room.
However, players can still register rankings and view their own records.

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding! 🎉
