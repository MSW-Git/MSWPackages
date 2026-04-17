# ResourcePackage

This package manages in-game resources for players, including both standard and rechargeable resources.
Game Masters (GMs) can view and modify player resources.
Players can earn, spend, and track their resources.

---

## Main Features

### ResourceAdminLogic (GM Functions)

Provides the following features for GMs to manage resources:

- **View Player Resources**
  - View all resources owned by a specific player.
  - See current values and recharge timers.

- **Edit Resources**
  - Modify resource values for any player.
  - Adjust recharge timers for rechargeable resources.
  - Save changes to player database.

- **Access Control**
  - Admin authentication ensures only authorized users can modify resources.

### PlayerResource (Player Functions)

Provides the following features for players to manage their resources:

- **Resource Management**
  - Add, consume, and track resources.
  - Resources are automatically synchronized between server and client.
  - ※ `PlayerResource` must be added to the player entity to work correctly.

- **Rechargeable Resources**
  - Special resource type that regenerates over time.
  - Server manages recharge timers to prevent cheating.

- **Data Persistence**
  - Resources are automatically saved to the database.
  - Data loads when player connects.

---

## Resource Data Format

### Resource Base Structure

| Field | Type | Description |
|-------|------|-------------|
| `Type` | Integer | Unique resource identifier |
| `Value` | Integer | Current amount of the resource |

### Rechargeable Resource Structure *(extends Resource Base)*

| Field | Type | Description |
|-------|------|-------------|
| `NextRechargeServerTime` | Integer | Server timestamp for next recharge |
| `RechargeInterval` | Integer | Time between recharges |
| `RechargeAmount` | Integer | Amount added per recharge |
| `MaxValue` | Integer | Maximum capacity for this resource |

---

## Resource Data Configuration

Resources are defined in the `ResourceDataSet.csv` file with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `Type` | Integer | Unique identifier for the resource |
| `Name` | String | Display name of the resource |
| `RUID` | String | Resource unique identifier |
| `IsRechargeable` | Integer | `0` = standard resource, `1` = rechargeable resource |
| `RechargeValue` | Integer | Amount to add per recharge cycle |
| `RechargeInterval` | Integer | Time in seconds between recharges |
| `MaxValue` | Integer | Maximum amount of this resource (`0` = unlimited) |
| `DefaultValue` | Integer | Starting value for new players |
| `IsInitialGrant` | Integer | Whether to grant this resource to new players (`0` = no, `1` = yes) |
| `ResetToMaxOnCreate` | Integer | Whether to set to max value for new players (`0` = no, `1` = yes) |

---

## Recommendation

It is recommended to set `PlayerEntityAuthorityCheck` to `true` in `WorldConfig`.
If this setting is false, the server function is exposed to all clients, which may lead to security vulnerabilities.

---

## Sample Project Guide

The sample project demonstrates resource management with a simple UI.

- **Press F3** → Open GM Resource Tool
  - View and modify resources for any player by entering their User ID.
  - Edit values and recharge timers directly.
  - ※ Must be modified while the user is **offline**.

- **Resource HUD**
  - Displays current resources for the player.
  - Shows recharge timers for rechargeable resources.

- Sample usage examples are included in `ResourceSampleLogic`.
