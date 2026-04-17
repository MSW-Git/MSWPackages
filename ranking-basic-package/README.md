# Ranking (Basic)

This package provides a basic ranking feature, allowing creators to display ranking to users.

---

## Features

### 1. Ranking Features

You can set up and use a single ranking. It uses `SortableDataStorage`.

### 2. Ranking UI

A ranking UI is provided to display the current ranking data to players.
Previous rankings are not accessible or viewable through the ranking UI.

### 3. Ranking Data Management Tool

Allows viewing and modifying ranking data.

---

## Installation

This package is available for use after import, but some initial setup is required.

**1. PlayerEntityAuthorityCheck Setting**

This package is designed under the assumption that `PlayerEntityAuthorityCheck` is set to `true` in `WorldConfig`.

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

In the `InitAndWait` function of `RankingDataStorageLogic`, you can modify `RankingConfigData` to change the configuration.

**6. UIHUD**

You can use the UIHUD model to conduct ranking registration tests.

---

## Usage

- Press **F2** to open the HUD UI, where players can register ranking.
  If the **Force** checkbox is selected, the score will be updated regardless of whether it is lower than or equal to the existing score.
- Press **F3** to open the UI, where players can view ranking.
  You can use the **ForceUpdate** button to regenerate the ranking list and clear the cached ranking data.

---

## Tool Usage

Press **F4** to open the tool UI, which allows you to manage the ranking data and user data.

### View

Retrieve ranking data from SortableStorage using the following parameters:

- `cycleIndex` — The target cycle index
- `itemCount` — The number of ranking entries to retrieve

### Edit ViewData

View and edit ranking data in SortableStorage. Data can be set after a query operation using the following parameters:

- `cycleIndex` — The target cycle index
- `ProfileCode` — The profile code to update

You can modify the following data fields:

- `Score` — Ranking score
- `Tag` — A string that can be defined by the creator. By default, the nickname is stored.

---

## Ranking Data Structure

The `RankingDataStorageLogic:SetScoreAndWait` function is used to register rankings.

On the server, ranking data is managed and utilized in two different ways:

**1. RankingDataStorageLogic - RankingDataList**

This list is updated over longer cycles and is used to calculate and display ranking positions to users.
`GetMyRankingDataAndWait` retrieves data directly from `SortableDataStorage` without returning cached values.

**2. RankingDataCacheLogic - RankingDataTable**

This table is updated more frequently and is used to fetch user-specific ranking data.
Since updating the ranking data list can take a long time, this table provides more up-to-date information for individual users in shorter intervals.
It also allows checking ranking data for cycles that are not currently active.
`GetMyRankingDataAndWait` attempts to fetch cached values before accessing `SortableDataStorage`.

---

## Ranking Config DataSet

| Field | Description |
|-------|-------------|
| `Key` | The keyName in DataStorage. Must be unique — should not conflict with keys used by other rankings or other DataStorage names. |
| `CycleEnum` | The ranking cycle (Day, Week, Month, Year, ...). If not provided, the ranking cycle will not exist. |
| `ViewCount` | Maximum number of users displayed in the ranking UI. Configure with consideration for DataStorage credit usage. |
| `RefreshIntervalSeconds` | Interval (in seconds) at which the ranking is refreshed. |
| `RefreshCacheIntervalSeconds` | Interval (in seconds) at which the cached data is removed. |
| `RankModeEnum` | Method used to calculate rankings (`Index`, `Rank`, `DenseRank`). |
| `ReleaseBaseTime` | Specifies the release date of the ranking. The `CycleIndex` is calculated based on this date — do not change arbitrarily. The cycle reference time can be set using `BaseDayOfWeek` and `BaseHour` in `DateTimeLogic`. |

---

## Cautions

### DataStorage Credit

Excessive use of DataStorage Credit may occur.
Adjust the following values in the `RankingConfigDataSet` to maintain appropriate credit usage:

1. `ViewCount`
2. `RefreshIntervalMinutes`
3. `RefreshCacheIntervalSeconds`

### Ranking Update Interval

Rankings are periodically updated in each world instance.
The "My Ranking Info" section at the bottom of the ranking is updated separately by `CacheLogic` on a different cycle.
Therefore, the two pieces of information displayed to the user may not match, and neither represents real-time rankings.

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding! 🎉
