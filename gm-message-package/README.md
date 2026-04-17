# GMMessage

This package provides a notification message feature, allowing creators to display messages to users.
It was developed with the goal of supporting smooth game operation by delivering important messages to players.

---

## Features

### 1. Notification Message Data Management

You can set and manage the start time, end time, display interval, and display duration of the messages.
The message data is stored in the GlobalDataStorage.

### 2. Notification Message Display

The message is displayed at the top of the user's game screen.

---

## Installation

This package is available for use after import, but some initial setup is required.

**1. Admin Permissions Check for Tool Usage**

You need to check if the user has admin permissions using the `Util:IsAdmin` function.

**2. UI Path Configuration**

You need to set the spawn paths for the message UI and the tool UI.
The `ParentUIPath` and `ParentUIToolPath` can be modified in `GMMessageLogic`.
By default, they are set to `"/ui/DefaultGroup"`.

**3. DataStorage Name and Tag Configuration**

The names and tags you are using might overlap with existing ones.
You can modify the `StorageName` and `TagName` in `GMMessageLogic`.

---

## Usage

You can open the tool UI using the **F11** key, which allows you to manage the notification data.

- The start and end dates are stored and used in **UTC time** in the DataStorage.
- You can set the time offset for your region using the **"User Time Offset"** option in the top-left corner of the tool.
  This setting will only affect the date displayed for viewing and notification configuration.
  It does not affect the already stored date data or the notification display logic.
- If you use the ID of an existing notification, it will overwrite the previous message data, and the previous message will be removed.
- After setting or deleting a notification, click the **refresh button** to view the updated data.

---

## Cautions

> Notice data is loaded only once when the world instance is created.

If there is too much data, it will consume more storage credits,
so it's recommended to delete notices that are no longer in use.

---

## Notification Data Structure

| Field | Type | Description |
|-------|------|-------------|
| `Id` | 6-digit integer | Unique identifier for the notification |
| `Message` | string | The message content to display |
| `Start/End Date` | datetime (UTC) | Start must be earlier than end date |
| `Display Interval` | integer (seconds, ≥ 2) | Interval at which the message is shown |
| `Duration` | integer (seconds, ≥ 2) | How long the message stays on screen; must be shorter than Display Interval |

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding! 🎉
