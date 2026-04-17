# Key Binding

This package provides functionality for input key bindings and virtual button key bindings.

---

## Usage

### 1. Key Binding Setup

**PC Setup**

Pressing the **"P"** key will display the UI for setting key bindings.
From this UI, you can change the key bindings by clicking the respective key change button and entering the desired key.
The key binding data changed on the client is stored in the database via the server, and the modified key bindings will be applied in subsequent gameplay.

**Mobile Setup**

On mobile, the buttons on the screen and the slots are linked, and you can adjust the position and size of the buttons.
The position-adjusted buttons are also stored in the database, and the buttons will appear in their changed position in the next gameplay.

### 2. Virtual Button Window

Once the key binding data is retrieved from the database and sent from the server to the client, the `PlayerSkillManager` on the client will learn four example skills.
By pressing the virtual buttons displayed on the screen or pressing the corresponding keyboard keys labeled on the virtual buttons, the skills will be cast.
As the skill is cast, a cooldown will be displayed, and the skill cannot be used again until the cooldown expires.
Note that the `SlotDownEvent` registered in `PlayerSkillManager` is connected to `InputLogic`, not `InputService`.

---

## Features

### 1. Key Binding Changes and Saving

The player's key binding data is saved and loaded from `PlayerSetting`.
Player input does not use `InputService` directly but passes through the intermediate layer, `InputLogic`. `InputLogic` calls the event associated with the slot bound to the inputted keyboard key.

### 2. Mobile Joystick Size Adjustment Support

When the key binding setup UI for this package is opened on mobile, a window for setting the position of the virtual buttons can be displayed.
Virtual buttons can be moved, and this data is also saved on the server.

### 3. Skill Acquisition and Usage

Once the player's key binding data is received, the virtual window will display five items: four skills and the jump action. More skills can be used via the keyboard.
The Slot events called by `InputLogic` are received by `PlayerSkillManager`, which then casts the skills.

---

## Installation

**1. Virtual Button Window UI Path Configuration**

You need to set the spawn paths for the Virtual Button Window UI.
The `ParentUIPath` can be modified in `KeyBindingSampleLogic`.
By default, it is set to `"/ui/DefaultGroup"`.

**2. Common Popup UI Path Configuration**

You need to set the spawn paths for the Common Popup UI.
The `ParentUIPath` can be modified in `UIPopupLogic`.
By default, it is set to `"/ui/PopupGroup"`.

**3. Player Components**

- `PlayerSetting` — Component responsible for managing the key binding data.
- `PlayerControllerComponent` — In the sample, the existing `PlayerControllerComponent` is removed to use the `PlayerControllerExtend` component. After the game starts, multiple components are added to the player. This process is performed in the `UserEnterEvent` of `KeyBindingSampleLogic`. The process of adding components in the User Enter Event can be skipped by pre-adding them as default components for the player.
- `PlayerSkillManager` — After reading the key binding data from the database, the server sends this data to the client, and the client then sets up the Virtual Button Window and learns the skills.
- `PlayerDBManager` — The server-side `PlayerSetting` that stores the key binding data will save it to the database via `PlayerDBManager`. The data is saved when `PlayerDBManager:OnBeginPlay` is triggered or when the player exits the game.

**4. Key Binding**

The key bindings for the default keys must be added to `InputLogic.DefaultKeyMap`, and this is done in `KeyBindingSampleLogic:OnBeginPlay`.

On mobile, the `UIJoystick` is used to set the key binding for movement input, and the `UIVirtualButtonWindow` is used to set the key binding for skill buttons.
To modify the key bindings for skill buttons, you need to handle the slots added in the VirtualButtonWindow model and `UIVirtualButtonWindow` component.

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding! 🎉
