# DropTableResolver

This package provides functionality for determining which items to return based on drop tables and probability.

---

## Features

### 1. Drop Item Resolver

The core functionality allows inputting data into a drop table and returning items probabilistically using a DataSet.
There are two drop modes supported:

**Single Drop**

- For each group, a pick attempt is made based on the group's `groupRate`.
- If the attempt succeeds, exactly one item is selected from that group.
- The quantity of the selected item is randomly determined between its min and max count values.
- Each group is evaluated independently, so multiple groups can be selected.
- Integer weights can be assigned to items within a group to influence selection probability.

**Multi Drop**

- Each item has its own drop rate and is evaluated independently.
- The system attempts to pick each item individually based on its assigned probability.
- The quantity of selected items is randomly determined between their min and max count values.
- Drop probabilities can be defined per `<GroupId, ItemId>` using percentage values.

### 2. Drop Simulation Functionality

A simulation function is included to verify and test drop rates.
Simulation results are output to the log for analysis and debugging.

### 3. Sample Dropped Item Entity

Sample code is included for dropping item entities at the LocalPlayer's position.
This example demonstrates client-side spawning and behavior.
To use server-side spawning, the script must be modified accordingly.

---

## Installation

This package is available for use after import, but some initial setup is required.

**1. Admin Permissions Check for Tool Usage**

You need to check if the user has admin permissions using the `AdminLogic` functions.

**2. UI Path Configuration**

You need to set the spawn paths for the sample UI.
The `ModelHUDName` can be modified in `DropResolverSampleUILogic`.
By default, they are set to `"/ui/DefaultGroup"`.

**3. DroppedItem Sample Pool Path**

You need to set the pool parent paths for the dropped items.
The `Path` can be modified in `DroppedItemSampleLogic`.
By default, they are set to `"/common"`.

---

## Usage

You can open the test HUD UI using the **F2** key, which allows you to test the functions.

All drop-related functions are located in the `DropLogic` class.
Please refer to the function definitions and accompanying comments for more information.

### Function Parameter Descriptions

| Parameter | Default | Description |
|-----------|---------|-------------|
| `RollCount` | — | The number of drop attempts to perform. |
| `RateMul` | `1.0` | Drop rate multiplier. If the resulting rate exceeds 1.0, it is clamped. In DropSingle, this affects the probability of picking a group, but does not influence the weights of items within the group. |
| `CountMul` | `1` | Multiplies the quantity of each picked item. The number of items in the `DropResultStruct` will also be scaled accordingly. |

---

## Cautions

**1. DataSet Load Order**

DataSets have dependencies between each other, so they must be loaded in a specific order.
Refer to `DatasetLoadLogic` for the correct loading sequence.

**2. Probability and Epsilon**

Probabilities are calculated using floating-point numbers, and to prevent precision errors, a small epsilon value is added to the calculated probabilities.
The epsilon is set to `0.000001`, which effectively increases each probability by about 0.0001%.
For details, see the `DropLogic.Epsilon` property.

**3. Max Roll Count**

If an excessive number of rolls is attempted (outside of simulation), the drop process will fail to avoid performance or logic issues.
Refer to the `DropLogic.MaxRollCount` property for more information.

**4. DroppedItem Sample Code**

Sample code is provided to visually spawn DroppedItem entities at the LocalPlayer's position based on roll results.
This is intended for client-side demonstration only.
To use this in an actual creator world, the script must be modified accordingly for your game's logic and server setup.

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding! 🎉
