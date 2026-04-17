# GameEvent

This package provides a game event system.
You can dynamically configure event schedules and edit related user data via an admin tool using `DataStorage`.

---

## Features

### 1. GameEvent System

Define and use game events through `DataSet`.
The server automatically updates event status based on the configured schedule and broadcasts the status to clients.

### 2. GameEvent UserData

Stores and retrieves user-specific event data.
User data is managed in `UserDataStorage`.

### 3. Admin Tool for Data Modification

Administrators can directly modify user event data through a tool, enabling flexible event management.

### 4. Example UI

A sample UI is included for users to view the list of ongoing or upcoming game events.

---

## Recommendation

It is recommended to set `PlayerEntityAuthorityCheck` to `true` in `WorldConfig`.
If set to `false`, server functions may be exposed to all clients, leading to potential security issues.

---

## Installation

After importing the package, perform the following initial setup:

**1. Add the following components to `DefaultPlayer`:**

- `PlayerDBManager`, `PlayerGameEvent`

**2. Admin Permission Check**

Use `AdminLogic` to verify if a user has admin privileges before granting tool access.

**3. UI Path Configuration**

Set spawn paths for message UI and tool UI in `GameEventSampleUILogic`.

**4. DataStorage Name Configuration**

Ensure naming/tag conflicts don't occur with existing data.
Modify the name in `GMGameEventToolLogic` if needed.

---

## Core Logic / Components / Functions

- `GameEventScheduleData`
- `GameEventDefData`
- `GameEventStatusEnum`
- `GameEventScheduleChangedEvent`
- `UserGameEventData`
- `GameEventLogic` — `Get___List`, `Get___ListByDefEnum`
- `PlayerGameEvent` — `GetOrCreateUserData`, `GetUserData`, `SetValue`, `SetDirty`, `IsExpiredUserData`

---

## GameEvent Data Structure

Game events are structured using two types of data:

**Schedule Data**

Defines the event duration and the ID of the event definition to use.
This data is server-only. The client cannot access past or future events and receives only the necessary active events.

**Definition Data**

Defines the type of event and parameters to be used.
Used both on the server and client.
You can extend `GameEventDefData` by event type (`DefType`).

---

## GameEvent Status

Events progress through the following statuses based on time:

| Status | Description |
|--------|-------------|
| `Awaiting` | Waiting (not visible to client) |
| `Preview` | Shown in advance before activation |
| `Active` | Live and ongoing |
| `Grace` | Grace period (e.g., reward exchange) |
| `Close` | Ended and removed from client |

Statuses always progress in order and never reverse.
In the following cases, an event will be closed and re-added:

1. Event data is modified dynamically
2. Server time is changed to the past (e.g., via cheat)

Each status change triggers events and function calls:

- `GameEventChangedEvent` is triggered from `GameEventLogic`
- Related `GameEventDefData` functions are called: `OnAwaiting`, `OnPreview`, `OnActive`, `OnGrace`, `OnClose`
- `OnAwaiting` is not called on the client, and clients cannot see awaiting events.
- Even if multiple statuses are skipped, only one event/function call is triggered per transition.

---

## GameEvent UserData

Some events may require storing user-specific progress data.
Use `PlayerGameEvent` to manage user data stored in `UserDataStorage`.

- UserData is a Lua table indexed by `ScheduleId` and `DefId`.
- The structure must be JSON-serializable (`_HTTPService:JSONEncode`).
- When modifying UserData, you must set both the data's `dirty` flag and the `PlayerGameEvent` dirty flag to save correctly.

### Storage Limit Management

As more events are added, UserData can accumulate and may exceed `DataStorage` limits.
If `ScheduleId` is no longer defined, the data is considered unnecessary and is deleted.
You can customize this logic in the `PlayerGameEvent:IsExpiredUserData` function.

---

## Sample UI Usage

Press **F3** to open the sample UI.
It shows a banner list of active and upcoming events available to the client.

---

## GameEvent Tool Usage

Press **F4** to open the admin tool UI.
You can manage schedule data and user data through this interface.

- Schedule data is stored in `GlobalDataStorage` and broadcasted to all instances.
- If a `ScheduleId` exists both in `DataSet` and `DataStorage`, the one in `DataStorage` takes precedence. If a schedule is deleted from `DataStorage` but still exists in `DataSet`, it will be reused.

> **Ideal for hotfixes or urgent event deactivation without server downtime.**
> **Not recommended to dynamically add new events that require UserData.**
> Due to broadcast failure, UserData may not find its corresponding schedule and may be incorrectly deleted.

- Use the **User Time Offset** option in the top-left corner to view and input time-based data for different time zones. This does **not** affect stored data timestamps.

> Make sure to edit UserData only while the user is offline.

---

## DataSet Columns

### GameEventScheduleData

| Column | Description |
|--------|-------------|
| `Id` | Schedule ID |
| `DefId` | Definition ID |
| `PreviewTime` | Preview start time |
| `StartTime` | Event start time |
| `EndTime` | Event end time |
| `CloseTime` | Full termination time |
| `Disable` | Whether the event is disabled |

### GameEventDefDataSet

| Column | Description |
|--------|-------------|
| `DefId` | Definition ID |
| `Type` | Enum string from `GameEventDefTypeEnum` |
| `Arg1~Arg3` | Arguments based on type |
| `ShowBanner` | Whether to show in client sample UI |
| `Disable` | Whether the definition is disabled |

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding! 🎉
