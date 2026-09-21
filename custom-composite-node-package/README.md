# CustomCompositeNode

This package provides custom Composite Tree node samples.

---

## Features

These nodes all extend `CompositeNode`. They cover ordering, weighting, branching, and threshold cases that the built-in composites (`SequenceNode`, `SelectorNode`, `ParallelNode`) do not.

### CompositeNode

**Order control**

| Node | Description |
|---|---|
| `ShuffleSequence` | Shuffles the child order, then runs children one by one like `SequenceNode`. Stops at the first `Failure`; returns `Success` only when every child succeeded. |
| `ShuffleSelector` | Shuffles the child order, then runs children one by one like `SelectorNode`. Stops at the first `Success`; returns `Failure` only when every child failed. |

> **Stopping** means the remaining children are not executed at all in that pass, not merely that the return value is decided.
>
> Both nodes reshuffle on the next entry after a pass ends. Neither exposes an inspector property — there is nothing to configure.

**Selection**

| Node | Description |
|---|---|
| `WeightedRandomSelector` | Picks one child by weighted roulette over `WeightsText` and returns that child's result unchanged. Picks again on the next entry after the child finishes. |
| `SwitchSelector` | Reads the Blackboard integer named in `SwitchKey` and runs the child at that 1-based index, like `switch`-`case` on an enum. Returns that child's result unchanged. Falls back to `DefaultChildIndex` when the value is out of range. |

> **`SwitchSelector` requires a Blackboard.** The tree must be attached through `AIComponent.BehaviourTreeId`. With a runtime `AddComponent` + `SetRootNode` pair the `BlackBoard` is `nil` and every pass falls back to `DefaultChildIndex`.

**Parallel**

| Node | Description |
|---|---|
| `ParallelSuccessThreshold` | Runs every child in parallel and, once all of them have finished, returns `Success` if at least `SuccessThreshold` children succeeded. |

> **No early exit.** The node waits for every child to finish even when the outcome is already decided — whether the success count has already passed the threshold, or the threshold can no longer be reached. While any child is still running, the node returns `Running`.

### Properties

| Node | Property | Description |
|---|---|---|
| `WeightedRandomSelector` | `WeightsText` | Comma-separated weights, e.g. `3,1,1`. Children without a weight default to `1.0`. A weight of `0` is never picked. If the total is `0`, the pick falls back to uniform. |
| `SwitchSelector` | `SwitchKey` | Name of the Blackboard variable (`Integer`) holding the switch value. The value is the 1-based index of the child to run (`1` = first child). |
| `SwitchSelector` | `DefaultChildIndex` | Child to run when the switch value is outside the child range. `Failure` if it is `0` or outside the child range. |
| `ParallelSuccessThreshold` | `SuccessThreshold` | Number of successful children required for `Success`. Setting it above the child count makes the node always fail. |

`ShuffleSequence` and `ShuffleSelector` have no properties.

---

## Console warnings

A misconfigured node still produces a usable tree — it just runs against a setup nobody intended, with nothing on screen to say so. These cases are reported to the Console instead. Each warning is printed once, not on every pass.

`SwitchSelector` warns when `SwitchKey` is empty or the `BlackBoard` is unavailable. Either case leaves the node on `DefaultChildIndex`; no error is raised.

`WeightedRandomSelector` warns about `WeightsText`:

| Situation | Behaviour | Warning |
|---|---|---|
| More weights than children (`3,1,1,5` on 3 children) | The extra weights are ignored | yes |
| Fewer weights than children (`3,1` on 3 children) | The remaining children default to `1.0` | yes |
| A field that is not a number `>= 0` (`3,x,-1`) | That field defaults to `1.0` | yes |
| Every weight is `0` (`0,0,0`) | The pick falls back to uniform | yes |
| Empty `WeightsText` (the default) | Every child weighs `1.0` | no — this is the documented default |
| An empty field inside the text (`1,,8`) | That child keeps `1.0` | no — an empty field is a deliberate slot |

---

## Usage

You can use any node from this package by adding it to a `BehaviourTree` entry.

- Child indices are 1-based (`1..ChildCount`).
- Every node returns `Failure` when it has no children. No error is raised.

**Running**

While a child returns `Running`, every node here holds on to that child: the shuffled order, the weighted pick, and the switch selection all stay put, and the same child runs again on the next frame.

`SwitchSelector` therefore does not switch the moment the Blackboard value changes — the change takes effect on the tick after the current child finishes.

**Comma-separated input**

In `WeightsText` an empty field still occupies a slot.

| Input | Result |
|---|---|
| `1,,8` | child 1 = `1`, child 2 = `1` (default), child 3 = `8` |

Weights that are non-numeric or negative fall back to `1.0`. Surrounding spaces are ignored, so `2 , 2 ,6` is read as `2,2,6`.

**Runtime tree changes**

Attaching or detaching children at runtime (`AttachChild` / `DetachChild`) on a tree that is already running is **not recommended**. These nodes assume the child set is fixed once the tree starts, and a mid-pass change is not picked up until the next entry. If your use case does change children at runtime, additional handling code is required on your side.

---

## Sample

The `Sample/` folder contains two demo trees driven by a log-only action node, plus one ready-to-spawn model per tree.

Composite nodes control **flow**, not motion, so the sample reports what ran and in what order to the Console instead of moving an entity. Watch the Console while it plays.

### `Sample/ActionNodes/SampleLog.mlua`

A leaf that logs `Label` when it is entered and again when it finishes, stays `Running` for `Duration` seconds, and returns `Failure` instead of `Success` when `ReturnFailure` is set.

### `Sample/ShuffleAndWeight.behaviourtree`

Shows `ShuffleSequence` reordering its children every pass, with two other composites nested underneath. This tree uses no Blackboard.

- **Node graph**
  - `ShuffleSequence` *(root)*
    - `WeightedRandomSelector` (`WeightsText = 6,3,1`)
      - `SampleLog` (`W-heavy(6)`, `Duration = 0.3`)
      - `SampleLog` (`W-mid(3)`, `Duration = 0.3`)
      - `SampleLog` (`W-rare(1)`, `Duration = 0.3`)
    - `ShuffleSelector`
      - `SampleLog` (`S-fail-a`, `Duration = 0.2`, `ReturnFailure = true`)
      - `SampleLog` (`S-fail-b`, `Duration = 0.2`, `ReturnFailure = true`)
      - `SampleLog` (`S-ok`, `Duration = 0.2`)
    - `SampleLog` (`Tail`, `Duration = 0.3`)
- **What to look for**
  - The three children finish in a different order every pass. `Tail` is sometimes first, sometimes last.
  - The selector always stops at `S-ok`; nothing runs after it in that pass.
  - Over many passes `W-heavy` is picked roughly six times as often as `W-rare`.
  - Set `WeightsText` to something invalid, such as `6,3,x,9`, to see the Console warning.

### `Sample/SwitchAndParallel.behaviourtree`

Shows `SwitchSelector` branching on a Blackboard value, with `ParallelSuccessThreshold` on one of the branches.

- **Blackboard variables**
  - `bb_State` (`Integer`, default `1`) - 1-based child index. Driven at runtime by `AISwitchSample`.
- **Node graph**
  - `SwitchSelector` *(root)* (`SwitchKey = bb_State`, `DefaultChildIndex = 1`)
    - `SampleLog` (`Idle`, `Duration = 1.0`) - case `1`
    - `ParallelSuccessThreshold` (`SuccessThreshold = 2`) - case `2`
      - `SampleLog` (`P-fast(0.5s)`, `Duration = 0.5`)
      - `SampleLog` (`P-slow(1.5s)`, `Duration = 1.5`)
      - `SampleLog` (`P-fail(0.8s)`, `Duration = 0.8`, `ReturnFailure = true`)
- **What to look for**
  - When `bb_State` flips, the child that is already `Running` finishes first. The switch only takes effect on the next tick.
  - All three parallel children start on the same tick, and the pass ends when the slowest one finishes, not when the threshold is already met.

### `Sample/AIComponents/AISwitchSample.mlua`

An `AIComponent` subclass that cycles `bb_State` through `1..StateCount` every `SwitchInterval` seconds so the switch is visible without any input.

| Property | Description |
|---|---|
| `SwitchInterval` | How often (seconds) `bb_State` moves to the next value. Default `4.0`. |
| `StateCount` | Number of switch cases `bb_State` cycles through (`1..StateCount`). Default `2`. |

It belongs to `SwitchAndParallel` only — `ShuffleAndWeight` has no `bb_State` to write to.

### `Sample/Model_SwitchSample` / `Sample/Model_ShuffleSample`

Two ready-to-spawn models, one per tree. Drop either into a map to see that tree run without further setup.

| Model | `BehaviourTreeId` | `AIComponent` |
|---|---|---|
| `Model_SwitchSample` | `SwitchAndParallel.behaviourtree` | `AISwitchSample` — drives `bb_State` |
| `Model_ShuffleSample` | `ShuffleAndWeight.behaviourtree` | native `AIComponent` — the tree needs no Blackboard |

A tree that reads no Blackboard needs no `AIComponent` subclass: point a plain `AIComponent` at it through `BehaviourTreeId`, as `Model_ShuffleSample` does. Write a subclass only when something has to drive Blackboard values at runtime.

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding!
