# Example-BehaviourTree

This package provides Behaviour Tree node samples.

---

## Features

These BT nodes fall into 2 categories and ship with supporting utility scripts.

### ActionNode

**Movement**

| Node | Description |
|---|---|
| `MoveTo` | Moves the entity toward the position stored in `TargetPositionKey` at the speed in `MoveSpeedKey` until within `ArrivalThreshold`. |
| `MoveToDirection` | Moves the entity in the direction of the position stored in `LookAtPositionKey` until within `ArrivalThreshold`. |
| `MoveToTarget` | Moves the entity toward the entity stored in `TargetEntityKey` until within `ArrivalThreshold`. |
| `Chase` | Moves the entity toward the entity stored in `TargetEntityKey` at the speed in `MoveSpeedKey` until within `ArrivalThreshold`. |
| `Jump` | Triggers a jump on the entity's `MovementComponent`. |
| `DownJump` | Triggers a down jump (drop-through) on the entity's `MovementComponent`. |

> **Plane** — all four movement nodes project positions to `Vector2` (XY plane) for distance and direction. The Z component is ignored, so they are intended for sideview / top-down 2D scenarios.
>
> **Required components on the controlled entity**:
> - `MoveTo`, `Chase` — `TransformComponent`.
> - `MoveToDirection`, `MoveToTarget`, `Jump`, `DownJump` — `TransformComponent` + `MovementComponent` + one Body component from the three available types: `RigidbodyComponent`, `KinematicbodyComponent`, or `SideviewbodyComponent`. `MovementComponent` requires a Body to drive movement; without it the entity will not actually move even though the node keeps returning `Running`.
> - Missing any required component causes the node to return `Failure` silently.

**Physics**

| Node | Description |
|---|---|
| `AddForce` | Adds the force stored in `ForceKey` to the entity's rigidbody. |
| `SetForce` | Sets the entity's rigidbody force to the force stored in `ForceKey`. |

**Lifecycle**

| Node | Description |
|---|---|
| `SpawnEntity` | Spawns the `SourceEntity` model at `SpawnPosition`; if `ResultEntityKey` is set, stores the spawned entity to that key. |
| `DestroyEntity` | Destroys the entity stored in `TargetEntityKey` after `Delay`. |
| `SetEntityEnable` | Enables or disables the entity stored in `TargetEntityKey` based on the `Enabled` property. |

**FX / Sound**

> Note: the integer `ParticleType` on the three particle nodes maps 1:1 to the API reference enums.
> - `PlayBasicParticle` → `BasicParticleType`
> - `PlayAreaParticle` → `AreaParticleType`
> - `PlaySpriteParticle` → `SpriteParticleType`
>
> Each node casts via `<Type>.CastFrom(self.ParticleType)` and returns `Failure` if `None` is produced. Refer to the API reference of the corresponding enum for the available values.

| Node | Description |
|---|---|
| `PlayBasicParticle` | Plays a basic particle effect with the configured position, rotation, and scale. |
| `PlayAreaParticle` | Plays an area particle effect within `AreaSize`. |
| `PlaySpriteParticle` | Plays a sprite-based particle effect using the configured `SpriteRUID`. |
| `PlaySound` | Plays a sound by `SoundId` at the configured volume. |

**Timing**

| Node | Description |
|---|---|
| `Wait` | Waits for `WaitTime` seconds before returning `Success`. |
| `WaitRandom` | Waits for a random duration between `WaitTimeMin` and `WaitTimeMax`. |
| `WaitBlackboardTime` | Waits for the duration stored in `WaitTimeKey`. |

**Blackboard writers**

> Note: the `Component` / `ComponentRef` variants require an explicit concrete component type on both the node and the matching Blackboard variable (e.g. `MovementComponent`).

| Node | Description |
|---|---|
| `SetBlackboardValue_*` | Writes the configured `Value` to the Blackboard variable named in `Key`. Provided for `Bool`, `Int`, `Number`, `String`, `Vector2`, `Vector3`, `Vector4`, `Color`, `Entity`, `EntityRef`, `Component`, `ComponentRef`. |

### DecoratorNode

**Flow control**

| Node | Description |
|---|---|
| `Inverter` | Inverts the child's `Success` / `Failure` and passes `Running` through unchanged. |
| `Succeeder` | Forces the child's final result to `Success` (passes `Running` through). |
| `Loop` | Re-runs the child `LoopCount` times, or indefinitely while `IsInfinite` is `true`. |
| `TimeLimit` | Fails if the child does not complete within `Duration`. |
| `Cooldown` | Blocks the child from running again until `Duration` has elapsed since the last completion. |

**State checks**

| Node | Description |
|---|---|
| `CanMove` | Allows the child to run only if the entity can move forward by `LookAheadDistance` without being blocked. |
| `IsOnGround` | Allows the child to run only if the entity's rigidbody reports it is on the ground. |
| `IsJumping` | Allows the child to run only if the entity is mid-jump (rising velocity or airborne). |

**Blackboard checks**

> Note: the `Component` / `ComponentRef` variants require an explicit concrete component type on both the node and the matching Blackboard variable.

| Node | Description |
|---|---|
| `BlackboardCondition_*` | Allows the child to run only if the Blackboard value in `Key` matches `CompareValue` under the chosen `Operator`. Provided for every supported value type. |
| `CompareBlackboardValues_*` | Allows the child to run only if the Blackboard values in `LeftKey` and `RightKey` satisfy the chosen `Operator`. Provided for every supported value type. |

### Util

| Script | Description |
|---|---|
| `Comparator` | Registers per-type comparison functions (`IsSet`, `Equal`, `Greater`, `Less`, …) used by the Blackboard check nodes. |
| `OperatorPolicy` | Declares which operators (equality / ordered) are valid for each Blackboard value type. |
| `ValueType` | Enumeration of Blackboard value types (`Bool`, `Int`, `Number`, `String`, `Color`, `Vector2`–`Vector4`, `Entity`, `EntityRef`, `Component`, `ComponentRef`). |

**Operator enum**

Integer values accepted by the `Operator` property on `BlackboardCondition_*` / `CompareBlackboardValues_*` nodes (defined in `Comparator.mlua`).

| Value | Name | Meaning |
|:-:|---|---|
| 0 | `IsSet` | value is not `nil` (second operand ignored) |
| 1 | `IsNotSet` | value is `nil` (second operand ignored) |
| 2 | `Equal` | `a == b` |
| 3 | `NotEqual` | `a ~= b` |
| 4 | `Greater` | `a > b` |
| 5 | `GreaterOrEqual` | `a >= b` |
| 6 | `Less` | `a < b` |
| 7 | `LessOrEqual` | `a <= b` |

**Allowed operators per type**

Policy from `OperatorPolicy.mlua`. Any disallowed combination causes `Compare` to return `false` immediately.

| Value Type | Allowed Operators |
|---|---|
| `Int` | `IsSet`, `IsNotSet`, `Equal`, `NotEqual`, `Greater`, `GreaterOrEqual`, `Less`, `LessOrEqual` |
| `Number` | `IsSet`, `IsNotSet`, `Equal`, `NotEqual`, `Greater`, `GreaterOrEqual`, `Less`, `LessOrEqual` |
| `Bool` | `IsSet`, `IsNotSet`, `Equal`, `NotEqual` |
| `String` | `IsSet`, `IsNotSet`, `Equal`, `NotEqual` |
| `Color` | `IsSet`, `IsNotSet`, `Equal`, `NotEqual` |
| `Vector2` | `IsSet`, `IsNotSet`, `Equal`, `NotEqual` |
| `Vector3` | `IsSet`, `IsNotSet`, `Equal`, `NotEqual` |
| `Vector4` | `IsSet`, `IsNotSet`, `Equal`, `NotEqual` |
| `Entity` | `IsSet`, `IsNotSet`, `Equal`, `NotEqual` |
| `EntityRef` | `IsSet`, `IsNotSet`, `Equal`, `NotEqual` |
| `Component` | `IsSet`, `IsNotSet`, `Equal`, `NotEqual` |
| `ComponentRef` | `IsSet`, `IsNotSet`, `Equal`, `NotEqual` |

---

## Usage

You can use any node from this package by adding it to a `BehaviourTree` entry.

- `ActionNode` scripts extend `ActionNode` and return `Success` / `Failure` / `Running` from `OnBehave(delta)`.
- `DecoratorNode` scripts extend `DecoratorNode` and decide whether their single child runs.
- `Util` scripts are called from the BT nodes.

---

## Sample

The `Sample/` folder contains two demo trees that combine nodes from this package, plus a component example that populates the tree's Blackboard from outside.

### `Sample/Patrol.behaviourtree`

A simple patrol tree that walks through three waypoints in order.

- **Blackboard variables**
  - `bb_MoveSpeed` (`Number`, default `5.0`) — movement speed.
  - `bb_Waypoint1` (`Vector3`, `(0, 0, 0)`) — first waypoint.
  - `bb_Waypoint2` (`Vector3`, `(5, 0, 0)`) — second waypoint.
  - `bb_Waypoint3` (`Vector3`, `(5, 5, 0)`) — third waypoint.
- **Node graph**
  - `SequenceNode` *(root)*
    - `MoveTo` (`bb_Waypoint1`, `bb_MoveSpeed`, `ArrivalThreshold = 1.5`)
    - `Wait` (`WaitTime = 1.0`)
    - `MoveTo` (`bb_Waypoint2`, `bb_MoveSpeed`, `ArrivalThreshold = 1.5`)
    - `Wait` (`WaitTime = 1.0`)
    - `MoveTo` (`bb_Waypoint3`, `bb_MoveSpeed`, `ArrivalThreshold = 1.5`)
    - `Wait` (`WaitTime = 1.0`)

### `Sample/ChaseOrPatrol.behaviourtree`

Chases the player when one is available; otherwise patrols back and forth between two waypoints.

- **Blackboard variables**
  - `bb_Player` (`Entity`) — chase target. Injected at runtime (see `AIChaseOrPatrol` below).
  - `bb_MoveSpeed` (`Number`, `5.0`) — declared for movement speed; not consumed by the current node graph.
  - `bb_Waypoint1` (`Vector3`, `(0, 0, 0)`) — patrol waypoint A.
  - `bb_Waypoint2` (`Vector3`, `(5, 0, 0)`) — patrol waypoint B.
- **Node graph**
  - `SelectorNode` *(root)*
    - `BlackboardCondition_Entity` (`Key = bb_Player`, `Operator = IsSet`)
      - `MoveToTarget` (`TargetEntityKey = bb_Player`, `ArrivalThreshold = 1.5`)
    - `BlackboardCondition_Entity` (`Key = bb_Player`, `Operator = IsNotSet`)
      - `SequenceNode`
        - `MoveToDirection` (`LookAtPositionKey = bb_Waypoint1`, `ArrivalThreshold = 3.0`)
        - `MoveToDirection` (`LookAtPositionKey = bb_Waypoint2`, `ArrivalThreshold = 3.0`)

### `Sample/AIComponents/AIChaseOrPatrol.mlua`

A component example that fills the `bb_Player` slot on the `ChaseOrPatrol` tree at runtime.

### `Sample/Model_ChaseOrPatrol.model`

A ready-to-spawn model that attaches `AIChaseOrPatrol` (which extends `AIComponent`) with its `BehaviourTreeId` already pointing at `ChaseOrPatrol.behaviourtree`, plus the components the movement nodes need: `TransformComponent`, `MovementComponent`, and `RigidbodyComponent` (Body). Drop it into a map to see the sample tree run without further setup.

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding!
