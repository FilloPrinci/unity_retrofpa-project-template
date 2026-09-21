# Retro FPA Project Template

The Unity project side of the Retro FPA template: a working Unity 6 (URP)
project that consumes the [`retrofpa-core`](https://github.com/FilloPrinci/unity_retrofpa-core)
package and demonstrates it end-to-end with a small playable demo.

See [`unity_retrofpa_kickoff_brief.md`](unity_retrofpa_kickoff_brief.md) for
the full design brief and rationale behind this project's structure.

## What lives here vs. in `retrofpa-core`

- **`retrofpa-core`** is the reusable "engine": manager singletons, the
  player controller, UI shell, `ScriptableObject` data types, shaders, and
  editor tooling. It contains no game-specific content.
- **This repository** is an actual Unity project: engine/render pipeline
  configuration (URP, the new Input System, Localization), and a small demo
  built from `retrofpa-core`'s systems, proving the two repos work together.

## Demo content

- **`Assets/Scenes/Persistent.unity`** — the bootstrap scene: every manager
  singleton, the `Player` prefab instance, and the full UI Canvas (main
  menu, pause menu, settings, dialogue, inventory, interaction prompt).
  Always loaded, never unloaded; levels load additively on top of it.
- **`Assets/Scenes/DemoLevel.unity`** / **`DemoLevel2.unity`** — two small
  levels sharing the same layout (an NPC with a branching dialogue, a
  pickupable key, an equippable test knife) but each with its **own**
  `SceneAtmosphere` (`Assets/StyleProfiles/DemoLevelAtmosphere.asset` — blue-
  grey fog/sky — vs. `DemoLevel2Atmosphere.asset` — pure black), proving
  fog/skybox are per-level while `Assets/StyleProfiles/N64VisualStyle.asset`
  (ambient, color grading, bloom, texture filtering) stays the same global
  look across both. Same idea for audio: each level has its own
  `SceneAmbientAudio` (birds in `DemoLevel`, an underground hum in `DemoLevel2`). A `PortalToDemoLevel2` / `PortalToDemoLevel`
  object in each level (an `Interactable` + `InteractableSceneChangeTrigger`)
  lets you walk between them.
- **`Assets/Items/`** — `RustyKey` (a plain key, not equippable),
  `TestKnife` (equippable, exercises the inventory's Equip/Unequip toggle
  and its live 3D preview), `TestItemA`/`TestItemB` (inventory-grid filler),
  `HeldItemBehavior`/`TestKnifeBehavior` (their `EquippableBehavior` assets),
  `ItemDatabase` (lists all of the above, for `SaveManager` to resolve a
  saved item id back to its asset).
- **`Assets/AudioProfiles/N64AudioProfile.asset`** — the game's global
  sounds (UI hover/confirm, main menu music, default footstep/pickup/
  interact), filled with clips from a third-party sample pack that is not in
  this repository (see *Demo audio* below). The whole system (UI sounds on
  every button, `FootstepAudio` on the Player, `SceneAmbientAudio` in each
  level) is wired up and safe to run with the clips missing.
- **`Assets/Dialogues/TestDialogue.asset`** — a 4-node branching conversation
  (greeting → yes/no question → two endings), fully localized (EN/IT).
- **`Assets/Prefabs/`** — the `Player` prefab, item world/equipped-model
  prefabs, and the UI prefabs (`InventorySlot`, `DialogueChoiceButton`).

Press Play, "Nuova Partita"/"New Game" from the main menu, and you're in
`DemoLevel`: walk up to the NPC to talk, the key/knife to pick them up, open
the inventory (Tab) to equip the knife, Escape for the pause menu (Settings
reachable from either menu, Save writes a single save file), and the portal
cube to try the other level. "Continue" on the main menu (only enabled once
you've saved) restores the exact level, position, inventory, and equipped
item — and any collectibles you already picked up stay gone.

## Scene Templates

Two Unity Scene Templates live in `Assets/SceneTemplates/` (they show up in
*File → New Scene*):

- **Retro FPA Level (Base)** — a new level: a `SpawnPoint`, a `SceneAtmosphere`
  and a `SceneAmbientAudio`. Its atmosphere profile is **cloned** for every
  new level, so editing one level's fog/sky never touches another's. The
  Player, UI and managers are *not* in it (they live in the persistent
  scene). After creating a level, add it to *Build Settings* and load it by
  name (`GameBootstrapper`, `SceneChangeTrigger`, `InteractableSceneChangeTrigger`).
  Assign a track to its `SceneAmbientAudio` for the level's music.
- **Retro FPA Persistent (Bootstrap)** — the always-loaded scene: every
  manager, the Player and the full UI, wired together (a copy of
  `Persistent.unity`). Everything it uses is a shared *reference* (prefabs,
  input actions, the demo's item/audio/style assets) except the global Volume
  profile, which is cloned because `StyleManager` writes into it at runtime.
  Re-point the demo assets to your own after creating it.

## Requirements

- Unity **6000.3.17f1** (see [`ProjectSettings/ProjectVersion.txt`](ProjectSettings/ProjectVersion.txt))
- Universal Render Pipeline (URP), new Input System, Localization — all
  already configured in this project

## Setup

Clone this repository and open it in Unity Hub. The core package is pulled
from git, pinned to a release tag, in [`Packages/manifest.json`](Packages/manifest.json):

```json
"com.filloprinci.retrofpa": "https://github.com/FilloPrinci/unity_retrofpa-core.git#v0.1.0"
```

To co-develop the core alongside this project, clone
[`unity_retrofpa-core`](https://github.com/FilloPrinci/unity_retrofpa-core) as a
sibling folder and temporarily point the dependency at it
(`"file:../../unity_retrofpa-core"`; adjust the path if you place it elsewhere).

Open the project in Unity Hub once cloned; Unity will resolve the package
dependency and import automatically.

## Demo audio

To try the audio system, `N64AudioProfile` (and the `SurfaceAudio` /
`SceneAmbientAudio` overrides in the demo levels) point at clips from the
free Asset Store pack **96 General Library (Free Sample Pack)**. That pack is
third-party and not redistributable, so it is **not in this repository**
(`.gitignore`d). Without it the demo runs silently, with no errors, and
those fields show as "Missing".

To hear it: import the pack from the Asset Store (Package Manager → My
Assets). It ships with its own `.meta` files, so the existing references
should re-link automatically; if any don't, re-assign them in
`Assets/AudioProfiles/N64AudioProfile.asset`.

This is only a test setup for this template: `retrofpa-core` ships no audio
of its own — see its README ("Using your own audio") to wire up your own.

## Known gaps

Mirrors `retrofpa-core`'s own list - no combat/health/HUD yet (equipping the
test knife has no gameplay effect beyond the input path).

## License

[MIT](LICENSE)
