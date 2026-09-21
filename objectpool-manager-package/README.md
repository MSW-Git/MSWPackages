# ObjectPool

This package pre-creates entities and reuses them by enabling and disabling, instead of spawning and destroying them on demand.

---

## Features

A pool owns a container entity, a hidden template entity cloned from your model or source entity, and the pooled entities themselves. You take entities out with `Spawn` and give them back with `Release`; nothing is destroyed in between.

| Script | Role |
|---|---|
| `Core/ObjectPoolManagerLogic` | Logic singleton. Creates and destroys pools. |
| `Core/ObjectPoolContainerComponent` | One pool. Hands entities out and takes them back. |
| `Core/ObjectPoolOption` | Struct holding the pool sizes. |
| `Core/PooledObjectInfo` | Struct holding one pooled entity and whether it is currently in use. |

---

## Usage

### Creating a pool

```lua
local option = ObjectPoolOption()
option.InitialSize = 20

local pool = _ObjectPoolManagerLogic:CreatePoolByModelId(monsterModelId, self.Entity.CurrentMap, option)
```

| Method | Description |
|---|---|
| `ObjectPoolContainerComponent CreatePoolByModelId(string modelId, Entity parentEntity, ObjectPoolOption options)` | Creates a pool whose template is spawned from `modelId`. |
| `ObjectPoolContainerComponent CreatePoolByEntity(Entity sourceEntity, Entity parentEntity, ObjectPoolOption options)` | Creates a pool whose template is cloned from `sourceEntity`. |
| `ObjectPoolContainerComponent GetPool(string containerId)` | Looks a live pool up by its container entity `Id`. Returns `nil` for an unknown id, and for a pool that is already destroyed or still tearing down. |
| `boolean DestroyPool(ObjectPoolContainerComponent pool)` | Destroys every pooled entity and the container, whether or not entities are in use. |

`parentEntity` is required. Pass `nil` for `options` to take the defaults.

### Using the pool

```lua
local monster = pool:Spawn(Vector3(3, 1, 0))
pool:Release(monster)

_ObjectPoolManagerLogic:DestroyPool(pool)
```

| Method | Description |
|---|---|
| `Entity Spawn(Vector3 position)` | Takes one entity out, places it at `position`, then enables it. Expands the pool when nothing is available, and returns `nil` once `MaxSize` is reached. |
| `boolean Release(Entity pooledObject)` | Disables the entity and makes it available again. |

`PooledCount` reports how many entities the pool owns. It is the only field either script exposes to the inspector; everything else is internal state.

### Options

| Field | Default | Description |
|---|---|---|
| `InitialSize` | 10 | How many entities to pre-create. Clamped to `MaxSize`. |
| `MaxSize` | 50 | Hard cap on the total. Must be greater than 0. |
| `ExpandSize` | 5 | How many to add when nothing is available, capped by the remaining capacity. A non-positive value stops the pool from expanding: creation warns when the pool starts below `MaxSize`, and a later `Spawn` with nothing available returns `nil`. |

Options are copied at creation, so mutating the same `ObjectPoolOption` afterwards does not affect the pool.

---

## Rules

**Parent kind** — The container follows `parentEntity`: a UI parent gets a `uiempty` container, anything else a `mapempty` one. A world entity cannot live inside a UI container, so pass a UI template when `parentEntity` is UI. A world parent accepts both, and a UI template placed there becomes world-space UI.

**Owning side** — A pool belongs to the side that called `CreatePool*`, because pool creation carries no `@ExecSpace` and therefore runs only there. Calling `Spawn`, `Release`, or `DestroyPool` on the client instance of a server-created pool is refused.

**Never `@Sync` the pool state** — The container entity and its component do replicate, so a server-created pool has a client instance too. Its state does not: `TemplateEntity`, `PooledEntities`, `AvailableEntities`, `PooledCount`, `IsDestroying`, and `Option` carry no `@Sync`, so the client instance keeps the declared defaults. That is exactly what the refusal above tests — `TemplateEntity` being empty means "not my side". Adding `@Sync` to any of them makes the client instance look like an owner, and the three methods start operating on a pool that is not theirs. Nothing warns you; the guard just stops working.

**Placement** — `Spawn` applies the position before enabling the entity, so it never appears at its previous position for a frame. A UI entity is placed through `UITransformComponent.anchoredPosition`, anything else through `TransformComponent.Position`. `UITransformComponent` derives from `TransformComponent`, so a UI entity answers both accessors with the same object — that is why the UI one is checked first. The two position fields are not interchangeable: `anchoredPosition` is anchor-relative while `Position` is parent-relative, and they only agree while the anchors sit at the center.

**Release** — The position is left as is, since `Spawn` always sets it.

**Ownership** — Do not destroy a pooled entity yourself; hand it back with `Release`. One destroyed after being released is discarded when the pool next reaches it, which is reported to the Console. One destroyed while still in use keeps its slot until the pool is destroyed, so it counts against `MaxSize` for good.

**Identity** — The manager tracks live pools by their container entity `Id`, and `GetPool(containerId)` looks one up, so you can keep the id instead of the `ObjectPoolContainerComponent` returned by `CreatePool*`. Container names (`PoolContainer_N`) are for the Hierarchy only. When the world session ends, any pool still tracked has its entities released.

**Cleanup** — A container destroyed without going through `DestroyPool`, such as on map unload, performs the same cleanup.

Failures return `nil` or `false` and report the cause through `log_error`.

---

## License

This project is licensed under the **MIT License**.
You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind.
For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

Happy Coding!
