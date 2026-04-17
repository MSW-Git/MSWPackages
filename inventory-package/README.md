# Inventory

This package provides inventory functionality, equipment mounting/dismounting, and item usage features.

---

## Installation

**Inventory UI Path Configuration**

The package is configured to spawn as a child of `DefaultGroup`.
You can change the spawn location through the `ParentUIPath` of `InventorySampleLogic`.
Since inventory is mainly used as a popup, it is recommended to move it to a group similar to `PopupGroup`.

**Player Component Addition**

You must add the following three components as Player components:
- `PlayerDBManager`
- `PlayerJob`
- `PlayerInventory`

**Admin Permissions Check**

You need to check if the user has admin permission using the `AdminLogic:IsAdmin` and `AdminLogic:IsAdminLocalPlayer` functions.

**Recommendation**

It is recommended to set `PlayerEntityAuthorityCheck` to `true` in `WorldConfig`.
If this setting is false, the server function is exposed to all clients, which may lead to security vulnerabilities.

---

## Introduction

### Equipment

**Grade**

The package includes classification for equipment by grade.
Grades include `None`, `Rare`, `Epic`, `Unique`, and `Legendary`.
Each grade has a different background color applied to the equipment.

**Mounting and Dismounting**

- When equipment is mounted, the icon of the mounted equipment is displayed on the right side of the UI.
- The corresponding item in the inventory grid on the left is dimmed.
- Character costume is also set according to the equipment.

### Items

**Usage**

When clicking on a non-equipment item slot, a usage popup appears.
The package includes a dummy stamina item; when used, a log is output.

**Deletion**

Items can be deleted through the trash can icon, and are deleted individually.
Equipped items or locked items cannot be deleted.

**Lock**

Items can be locked or unlocked through the lock icon on the right side of the item popup.
When locked, an icon is displayed at the bottom of the item slot.

**Duration**

Items with expired usage periods cannot be used.
Items with usage periods display a small clock in the bottom right corner of the item slot.

### Loading Order at Game Start

There is separate logic for loading data on server and client.
When client loading is completed, information about the completion is sent to the server, and player information update from the DB begins.

### Player Components

- `PlayerJob` — Manages job and job experience.
- `PlayerInventory` — Manages all owned items and equipment currently equipped by job.

---

## Usage

**Example**

- Press **I** to open and close inventory.
- Press **Alpha1** to create dummy stamina.
- Press **Alpha2** to create dummy stamina with 1-minute validity period.
- Click inventory item slots to open equipment or consumption popup.

**Job**

The package only contains data for one job.
Since it's structured to save multiple jobs, you just need to add job data.
Add data for the corresponding job to `JobDataSet`, `(JobCode)BasicStatDataSet`, and `PlayerJobCodeEnum`.

**Stat**

Addition of stats for characters or equipment is needed.

**Item**

To add items, you must enter data in the following two DataSets:
- `ItemDataTable`
- `ItemGearDataSet` — The ID entered here must also have data for the same ID in `ItemDataTable`.

Items have different usage methods depending on their category.
If you add a category of items with usage functionality to `ItemCategoryEnum`, you must also write the `UsableFunction` in `ItemCategoryEnum`.

The sorting function for items can be added to the `InventorySortEnum` logic along with Enum properties.

---

## Data Structures

### ItemData

Used to contain various information about items. Used for caching data read from `ItemDataTable` — read-only.

| Field | Description |
|-------|-------------|
| `Id` | Unique value that does not duplicate with any other item ID. |
| `Name` | The name of the item, displayed in UI, etc. |
| `RUID` | The image RUID of the item. |
| `UsableFunction` | The name of the function used to use this item. Must exist in `ItemCategoryEnum.UsableFunctionsByName`. |
| `UsableFunctionParams` | The list of parameters for the `UsableFunction`. |
| `CanDiscard` | Whether the item can be discarded. |
| `NonStackable` | Whether multiple items can be contained in one slot. |
| `ExpirationDate` | The expiration date of the item. The item cannot be used after this time. |
| `ExpirationSeconds` | The duration of the item. Added at the time of item creation. In `ItemDataTable`, entered in days as `ExpirationPeriodDay`. |

### ItemStruct

Contains information about items owned by users.

| Field | Description |
|-------|-------------|
| `GUID` | Unique value of the created item. Even items with the same ItemId have different GUIDs. |
| `ItemId` | The ItemId of the item's ItemData. |
| `Count` | The number of items. |
| `EndDateSeconds` | The expiration date of the item converted to seconds. |
| `CreateDateTimeSec` | The creation time of the item converted to seconds. |
| `Lock` | Whether the item is locked. If locked, it cannot be discarded. |
| `ItemGearStruct` | A structure used when the item is equipment. |

### ItemGearData

Contains various information about equipment items. Used for caching data read from `ItemGearDataSet` — read-only.

| Field | Description |
|-------|-------------|
| `Id` | Unique value that does not duplicate with any other equipment item ID. |
| `AvatarRUID` | The RUID value applied to the avatar when wearing the equipment. |
| `GearCategory` | The category of the equipment. |
| `ReqLevel` | The minimum level required to wear the equipment. |
| `ReqJob` | The job ID that can wear the equipment. |
| `DefaultGrade` | The default grade of the equipment. |

### ItemGearStruct

Contains information about equipment items owned by users.

| Field | Description |
|-------|-------------|
| `IsAttached` | Whether the user has equipped it. |
| `Grade` | The equipment grade. |

---

## Item Addition and Deletion Logic

Item addition and deletion work through `ItemFactoryLogic`.

Various failure factors can occur during item addition (insufficient inventory space, stack count overflow, etc.).
Therefore, when adding items, **a simulation is run first**, and if successful, the actual addition process is performed.

### ItemCreateParamStruct

| Field | Description |
|-------|-------------|
| `ItemId` | The ID of the item to be created. |
| `ItemCount` | The number of items to be created. |
| `EndDateSeconds` | Enter `0` for permanent items. For time-limited items, enter the end date in UTCTime seconds. e.g., `DateTime.UtcNow.Elapsed // 1000 + 60 * 10` |
| `IsIgnoreInvenCapacity` | Always create without checking inventory slot count. Used for paid item purchases, item boxes, etc. |
| `Grade` | Equipment grade when the item being created is equipment. If not set, the default grade is used. |

### ItemCreateResultStruct

| Field | Description |
|-------|-------------|
| `PrevItemStructList` | Table for identical item confirmation. (Simulate only) |
| `AccSlotCountTable` | Table for checking slot count limit excess. (Simulate only) |
| `NewItemGUIDs` | GUID table for newly created items. |
| `AccItemCountTable` | Number of ItemCount added by ItemGUID. |
| `FailInfo` | Failure-related message. Can be checked with `FailInfo.Message`. |

### ItemDeleteParamStruct

| Field | Description |
|-------|-------------|
| `ItemGUID` | The item GUID to delete. Cannot be used together with `ItemId`. |
| `ItemId` | The item ID to delete. Cannot be used together with `ItemGUID`. |
| `ItemCount` | The number of items to be deleted. |

### ItemDeleteResultStruct

| Field | Description |
|-------|-------------|
| `DeletedItems` | GUID table for items being deleted. |
| `DeletedItemCountTable` | Number of items being deleted by ItemGUID. |
| `FailInfo` | Failure-related message. Can be checked with `FailInfo.Message`. |

---

## Tool Usage

Press **O** to open the management tool.

> When adding, modifying, or deleting user items through the management tool, the corresponding player must be **offline**.

**Look And Add View**

In this view, you can check the item possession status and distribute items.
The Created Date and Expire Date are based on UTC.
If you enter a Time Offset in the upper left, the Date will be displayed based on the combined timezone.
If you set a period for the distributed item, that period is based on the combined timezone.

**Load And Edit View**

In this view, you can load the user's DB string and manually change values to add, modify, or delete items.
Be careful — even if the string is not written properly, it can be saved as-is since proper validation is not performed on the string.

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding! 🎉
