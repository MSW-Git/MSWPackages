# StateSet

This package provides basic StateType entries, ConditionType entries, sample StateSet entries, and utility logic for samples.

It is intended to help creators understand how StateSet graphs can be built from small reusable states and conditions, then adapt those entries to their own worlds.

---

## Features

1. Basic State Type Entries

Movement states:

- MoveBase : Provides shared movement properties InputSpeed and Tolerance, stops movement on enter and exit, and flips SpriteRendererComponent by movement direction.
- MoveToLocation : Moves the entity toward a fixed world position.
- MoveToEntity : Moves the entity toward the configured Target entity.
- MoveToNearestTaggedEntity : Finds the nearest entity with the configured Tag and moves toward it.

Jump states:

- JumpBase : Provides shared MovementComponent lookup logic.
- Jump : Calls MovementComponent:Jump().
- DownJump : Calls MovementComponent:DownJump().

Force states:

- ForceBase : Provides shared RigidbodyComponent lookup logic.
- AddForce : Adds Force to RigidbodyComponent.
- SetForce : Sets RigidbodyComponent.Force.

Entity states:

- EntityBase : Provides shared parent Entity and TransformComponent lookup logic.
- SpawnEntity : Clones Source at SpawnPosition under Parent. Name, IncludeChildren, and Spawned are available for spawned entity control.
- SpawnModel : Spawns an entity from ModelId at SpawnPosition under Parent. Name and Spawned are available for spawned entity control.
- DestroyEntity : Destroys Target after Delay.
- DestroyNearestTaggedEntity : Destroys the nearest entity with Tag after Delay.
- SetEntityEnable : Sets Target enable state.

Particle states:

- ParticleBase : Provides shared particle properties: ParticleType, Instigator, Position, ZRotation, Scale, IsLoop, and AttachToParentEntity.
- PlayBasicParticle : Plays a BasicParticleType effect.
- PlayAreaParticle : Plays an AreaParticleType effect with AreaSize.
- PlaySpriteParticle : Plays a SpriteParticleType effect with SpriteRUID.

Sound states:

- SoundBase : Provides shared parent Entity.
- PlaySound : Plays sounds with SoundId and Volume.

2. Basic Condition Type Entries

Time conditions:

- TimeBase : Provides nothing.
- HasTimePassed : Checks whether Duration seconds have passed from the source state enter time.

Detect conditions:

- DetectBase : Provides nothing.
- DetectTaggedEntityBase : Provides shared Tag and Tolerance search logic.
- IsTaggedEntityDetected : Checks whether at least one entity with Tag exists within Tolerance.
- IsTaggedEntityNotDetected : Checks whether no entity with Tag exists within Tolerance.

Random conditions:
- RandomBase: Provides basic things for random stuff.
- BasicRandom : Rolls once when the previous state is entered, then returns true when the stored random result is within Chance.

Rigidbody conditions:

- RigidbodyBase : Provides shared RigidbodyComponent lookup logic.
- CanRigidbodyMove : Checks RigidbodyComponent:PredictFootholdEnd(Distance, IsForward).
- IsRigidbodyOnGround : Checks RigidbodyComponent:IsOnGround().
- IsRigidbodyJumping : Checks whether the entity is moving upward and is not on the ground.

3. Network Execution Helpers

This package includes NetworkLogic in the Util folder.

NetworkLogic is used by StateType entries to decide where each state should run.

- HasAuthority(entity) : Checks whether the current execution side has authority over the entity.
- Entity ownership states use HasAuthority. If an entity has OwnerId, only the owning client runs the state. If an entity has no OwnerId, localized entities run on the client and non-localized entities run on the server.
- Localized effect and spawn states use Localize directly. Spawn, particle, and sound states run on the client for localized entities and on the server for non-localized entities.

4. Sample StateSet Entries

The Sample folder contains sample StateSets and Models.

- HungryMonster : Detects Food, moves toward the nearest Food entity, waits when close, eats one Food, then returns to DEFAULT so stacked Food can be consumed one by one.
- Speaker : Alternates between two sound states every two seconds.
- Firecracker : Uses particle states in a timed loop.
- Gacha : Waits, rolls random reward transitions by OrderInTransit, spawns Potion or Scroll reward models, and returns to READY when no reward is selected.

---

## Installation

This package is available for use after import, but StateSet setup is required.

1. Add StateComponent
    - Add StateComponent to the entity that will run a StateSet.
2. Assign StateSetId
    - Set StateComponent.StateSetId to the StateSet entry ID you want to use.
3. Add Required Components
    - Add the components required by the states and conditions used in the StateSet.
        - Movement states require TransformComponent and MovementComponent.
        - SpriteRendererComponent is optional for movement direction flipping.
        - Jump states require MovementComponent.
        - Force states and Rigidbody conditions require RigidbodyComponent.
        - Tag detection requires target entities with TagComponent.
        - SpawnModel requires a valid ModelId.
        - Particle states require valid ParticleType values or SpriteRUID values.
        - Sound states require valid SoundId values.
4. Include Util
    - NetworkLogic in the Util folder is required by StateType entries.
    - When importing or exporting this package, include the Util folder together with StateSet.

---

## Usage

- Prepare a MapleTileMap and place some tiles.
- Drag models from Sample and drop them onto the MapleTileMap you want.
- Use the sample StateSets as graph patterns, not only as feature demos.

---

## Core

**StateType entries are organized by feature under Core/State.**

Core/State/Move:

- MoveBase
- MoveToLocation
- MoveToEntity
- MoveToNearestTaggedEntity

Core/State/Jump:

- JumpBase
- Jump
- DownJump

Core/State/Force:

- ForceBase
- AddForce
- SetForce

Core/State/Entity:

- EntityBase
- SpawnEntity
- SpawnModel
- DestroyEntity
- DestroyNearestTaggedEntity
- SetEntityEnable

Core/State/Particle:

- ParticleBase
- PlayBasicParticle
- PlayAreaParticle
- PlaySpriteParticle

Core/State/Sound:

- SoundBase
- PlaySound

**ConditionType entries are organized by feature under Core/Condition.**

Core/Condition/Time:

- TimeBase
- HasTimePassed

Core/Condition/Detect:

- DetectBase
- DetectTaggedEntityBase
- IsTaggedEntityDetected
- IsTaggedEntityNotDetected

Core/Condition/Random:

- RandomBase
- BasicRandom

Core/Condition/Rigidbody:

- RigidbodyBase
- CanRigidbodyMove
- IsRigidbodyOnGround
- IsRigidbodyJumping

---

## Util

Util/NetworkLogic:

- Provides authority checks used by StateType entries.
- Uses Entity.OwnerId to allow the owning client to run states for owned entities.
- Uses Entity.Localize to separate client-only localized entities from server-side normal entities.
- This logic is required for Move, Jump, Force, Entity, Particle, and Sound states.

---

## Sample

HungryMonster:

- HungryMonster.model uses StateComponent, MovementComponent, RigidbodyComponent, SpriteRendererComponent, and TransformComponent.
- Food.model uses TagComponent with the Food tag.
- HungryMonster.stateset transitions from DEFAULT to HUNT when Food is detected.
- HUNT uses MoveToNearestTaggedEntity(Tag = Food, InputSpeed = 0.35, Tolerance = 0.5).
- WAIT is entered when Food is close, then EAT destroys the nearest Food.
- EAT returns to DEFAULT with NCAlways so multiple Food entities at the same position can be eaten one at a time.

Speaker:

- Speaker.model uses StateComponent, RigidbodyComponent, SpriteRendererComponent, and TransformComponent.
- Speaker.stateset alternates between two PlaySound states every two seconds.
- Firecracker.stateset is also included in this sample folder as a timed particle-loop example.

Gacha:

- Gacha.model uses StateComponent, RigidbodyComponent, SpriteRendererComponent, and TransformComponent.
- Gacha.stateset follows READY -> TRY -> reward or READY.
- TRY evaluates reward transitions by OrderInTransit. Higher OrderInTransit values are evaluated first.
- SPAWN_SCROLL uses BasicRandom(Chance = 0.3) with higher priority.
- SPAWN_POTION uses BasicRandom(Chance = 0.15) after Scroll fails.
- READY is the NCAlways fallback when no reward is selected.
- Potion (Reward).model and Scroll (Reward).model use Reward.stateset, which applies AddForce when spawned.

---

## Notes

- BasicRandom rolls once in OnPreviousStateEnter and reuses that result during condition checks for the transition.
- Multiple BasicRandom transitions from the same state are evaluated in OrderInTransit order. Because each condition has its own roll, lower-priority rewards are evaluated only after higher-priority transitions fail.
- Use NCAlways fallback transitions with lower OrderInTransit when you need a default path.

---

## License

This project is licensed under the MIT License. You are free to use, modify, and distribute this project.

However, the software is provided "as is", without warranty of any kind. For more details, please see the [LICENSE](https://opensource.org/licenses/MIT).

---

## Happy Coding!