# Quest & Achievement

This package provides common quest and achievement features in the game.

---

## Recommendation

It is recommended to set `PlayerEntityAuthorityCheck` to `true` in `WorldConfig`.
If this setting is false, the server function is exposed to all clients, which may lead to security vulnerabilities.

---

## Installation

This package is available for use after import, but some initial setup is required.

**1. Player Components**

The following components need to be added to the `DefaultPlayer`:
- `PlayerDBManager`, `PlayerAccount`, `PlayerAchievement`, `PlayerQuest`

**2. Admin Permissions Check for Tool Usage**

You need to check if the user has admin permissions using the `Util:IsAdmin` function.

**3. UI Path Configuration**

You need to set the spawn paths for the message UI and the tool UI.
- The `ParentUIPath` and `ParentUIToolPath` can be modified in `AchievementSampleLogic`.
- The `ParentUIPath` and `ParentUIToolPath` can be modified in `QuestSampleLogic`.
- By default, they are set to `"/ui/DefaultGroup"`.

**4. DataStorage Name Configuration**

The name you are using might overlap with existing ones.
- Modify the `StorageName` in `GMAchievementToolLogic`.
- Modify the `StorageName` in `GMQuestToolLogic`.

**5. UIHUD**

You can use the UIHUD model to conduct achievement and quest tests.

---

## Action Event and Condition

Achievements and quests process progress values by receiving `ActionEvent` through `ActionCondition`.

1. Define the desired Action in `ActionEnum` in advance, then create an `ActionEvent` and send it to `UserEntity`.
2. Predefine the desired `ActionCondition` in `ActionConditionEnum`. Inherit from `ActionConditionData`, add a new `ActionConditionData`, and implement it by overriding the necessary functions.
3. The `GetActionEnumList` function must be implemented.
4. Implement the `Check` function and `GetNextUserValue` function to verify whether the `actionEvent` is valid and to return the updated UserValue.
5. Some conditions may require values to be updated when a quest/achievement is reset or when the user logs in. In such cases, implement the `IsUpdateNeeded` and `GetUpdatedValue` functions.

---

# Achievement

This package provides common achievement features in the game.

## Features

### 1. Achievement Features

The achievement progress increases based on the user's actions, and rewards can be granted upon completion.
Repeatable achievements (daily, weekly, etc.) can be added.

- Achievements can have multiple steps. The value increases up to the last step's value, regardless of the current step.
- A parent achievement can be assigned, and it will be completed once all child achievements are finished.
- Even if achievement data is modified incorrectly, user data is designed to be preserved as much as possible.

### 2. Achievement User Data Management Tool

Allows viewing and modifying user achievement data.

## Usage

Press **F1** to open the UI, where you can view achievement progress and modify user data through the debug interface.
You can configure achievement data in `AchievementDataSet` and `AchievementStepDataSet`.

## Tool Usage

Press **F2** to open the tool UI, which allows you to manage user achievement data.

- The start and end dates are stored and used in **UTC time** in DataStorage.
- You can set the time offset using the **"User Time Offset"** option in the top-left corner.
- User data should be safely modified only when the user is **offline**.

## Data Structure

### AchievementData

| Field | Description |
|-------|-------------|
| `Id` | Integer |
| `ParentId` | Parent achievement ID. A parent data cannot have another parent. |
| `Name` | Name |
| `CategoryEnum` | CategoryEnum |
| `CycleEnum` | Setting CycleEnum resets achievements based on the reference time. Configured using `BaseHour` and `BaseDayOfWeek` in `DateTimeLogic`. |
| `CondEnum`, `CondArg`, `CondExtra` | ActionCondition settings for achievement progress conditions. |
| `Priority` | Listed in ascending order. If equal, sorted by ID ascending. If not specified, maximum integer value is applied. |
| `Disable` | This data will not be loaded. |

### AchievementStepData

| Field | Description |
|-------|-------------|
| `Id` | Achievement ID |
| `Step` | Achievement step. A single achievement can have multiple step data. Must be entered in order. |
| `Value` | Achievement condition integer value. Cannot be the same as or smaller than the previous step's value. |
| `Description` | Step description |
| `RewardItems` | Reward items. Uses `,\|` as separator. Format: `"itemId1,count1\|itemId2,count2..."` |
| `Disable` | This data will not be loaded. |

---

# Quest

This package provides common quest features in the game.

## Features

### 1. Quest Features

The quest progress increases based on the user's actions, and rewards can be granted upon completion.
Repeatable quests (daily, weekly, etc.) can be added.

A Quest has the following four states:

| State | Description |
|-------|-------------|
| `Init` | Initial state. User data is not stored in the database. |
| `Progress` | Quest is in progress. |
| `Completed` | Quest has been completed. |
| `Stored` | Also an initial state, but user data is stored in the database. |

- Quests can have multiple conditions.
- Even if quest data is modified incorrectly, user data is designed to be preserved as much as possible.

### 2. Quest User Data Management Tool

Allows viewing and modifying user quest data.

## Usage

Press **F3** to open the UI, where you can view quest progress and modify user data through the debug interface.
You can configure quest data in `QuestDataSet` and `QuestConditionDataSet`.

## Tool Usage

Press **F4** to open the tool UI, which allows you to manage user quest data.

- The start and end dates are stored and used in **UTC time** in DataStorage.
- You can set the time offset using the **"User Time Offset"** option in the top-left corner.
- User data should be safely modified only when the user is **offline**.

## Data Structure

### QuestData

| Field | Description |
|-------|-------------|
| `Id` | Integer |
| `Name` | Name |
| `Desc` | Description |
| `ProgressingDesc` | The description when the state is `Progress` |
| `CategoryEnum` | CategoryEnum |
| `IsRepeatable` | Whether this quest is repeatable. |
| `CycleEnum` | Setting CycleEnum resets quests based on the reference time. Configured using `BaseHour` and `BaseDayOfWeek` in `DateTimeLogic`. |
| `LinkedPrevId` | ID of the prerequisite quest required to start. This quest will not appear unless the prerequisite is completed. |
| `RequiredId` | ID of the prerequisite quest required to start. |
| `AutoAccept` | If the quest is available, it will be accepted automatically. |
| `CannotAbandon` | Prevents the quest from being abandoned. |
| `RewardItems` | Reward items. Uses `,\|` as separator. Format: `"itemId1,count1\|itemId2,count2..."` |
| `Priority` | Sorted by: State (Progress > CanAccept > Others > Completed), Priority (ascending), Id (ascending). |
| `Disable` | This data will not be loaded. |

### QuestConditionData

| Field | Description |
|-------|-------------|
| `Id` | Quest ID |
| `Description` | Condition description |
| `CondEnum`, `CondArg`, `CondExtra` | ActionCondition settings for quest progress conditions. |
| `Value` | Quest condition integer value. |
| `Disable` | This data will not be loaded. |

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding! 🎉
