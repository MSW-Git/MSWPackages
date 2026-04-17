# GlobalConfig

This package provides features for managing GlobalConfig.
GlobalConfig refers to values that are used consistently across all areas and can be dynamically changed using the management tool.

---

## Features

### 1. GlobalConfig Management Tool

You can load data stored in `GlobalDataStorage`, save modified data back to `GlobalStorage`, or reset it using the information from the DataSet.

> Note: Reset data will not be reflected in `GlobalDataStorage` until you click the save button, so make sure to press the save button to apply the changes.

### 2. Adding New GlobalConfig

You can add new GlobalConfig data to the `GlobalConfigDataSet`.

---

## Installation

This package is available for use after import, but some initial setup is required.

**1. Admin Permissions Check for Tool Usage**

You need to check if the user has admin permissions using the `Util:IsAdmin` function.

**2. UI Path Configurations**

You need to set the spawn paths for the test button UI and the tool UI.
The `ParentUIPath` and `ParentUIToolPath` can be modified in `GlobalConfigSetToolLogic`.
By default, they are set to `"/ui/DefaultGroup"`.

**3. DataStorage Name and Tag Configuration**

The names and tags you are using might overlap with existing ones.
You can modify the `StorageName` and `TagName` in `GlobalConfigLogic`.

---

## Usage

You can open the tool UI using the **F9** key, which allows you to manage GlobalConfig data.

- Add the information for the new GlobalConfig to the `GlobalConfigDataSet`.
- If there is no stored value in DataStorage, the default value stored in the DataSet will be used.
- The `Id(Integer)`–`Name(String)` EnumTable of GlobalConfigs is automatically added inside `GlobalConfigEnumLogic`. Since it is not added as a property, use the `ToInt`/`ToString` methods.

Refer to the example usage in the DataSet and `UIButtonTestWorldBoss` Script.

---

## Cautions

- Duplicate IDs are not allowed. If there are duplicate IDs, only the last entry in the DataSet will be visible.
- Previously saved data may remain if a previously used ID is deleted and then reused.

---

## Data Structure

| Field | Type | Description |
|-------|------|-------------|
| `Id` | 6-digit integer | Unique identifier |
| `Name` | string | Config name |
| `DefaultValue` | string | Default value |
| `ClientSync` | boolean | Whether to sync to client |

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding! 🎉
