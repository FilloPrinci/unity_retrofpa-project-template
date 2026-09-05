# Retro FPA Project Template

The Unity project side of the Retro FPA template: a working Unity 6 (URP)
project that consumes the [`retrofpa-core`](https://github.com/FilloPrinci/unity_retrofpa-core)
package and demonstrates it end-to-end with real Scene Templates and a small
demo level.

See [`unity_retrofpa_kickoff_brief.md`](unity_retrofpa_kickoff_brief.md) for
the full design brief and rationale behind this project's structure.

## What lives here vs. in `retrofpa-core`

- **`retrofpa-core`** is the reusable "engine": manager singletons,
  components, `ScriptableObject` data types, shaders, and editor tooling.
  It contains no game-specific content.
- **This repository** is an actual Unity project: engine/render pipeline
  configuration (URP, the new Input System, Localization, Scene Template
  package), the real Scene Templates (e.g. "Base Level" with fog/Volume
  already configured, a `SpawnPoint`, wired references), and a small demo
  level built from `retrofpa-core`'s prefabs, proving the two repos work
  together.

## Requirements

- Unity **6000.3.17f1** (see [`ProjectSettings/ProjectVersion.txt`](ProjectSettings/ProjectVersion.txt))
- Universal Render Pipeline (URP), new Input System — both already configured
  in this project

## Setup

Clone this repository **as a sibling of `unity_retrofpa-core`**, e.g.:

```
some-folder/
  unity_retrofpa-core/
  unity_retrofpa-project-template/
```

While `retrofpa-core` is under active co-development, this project references
it by local file path in [`Packages/manifest.json`](Packages/manifest.json):

```json
"com.filloprinci.retrofpa": "file:../../unity_retrofpa-core"
```

If you place the two repositories somewhere else relative to each other,
update that path accordingly. Once `retrofpa-core` is versioned
independently, this will switch to a git URL pinned to a release tag.

Open the project in Unity Hub once cloned; Unity will resolve the package
dependency and import automatically.

## License

[MIT](LICENSE)
