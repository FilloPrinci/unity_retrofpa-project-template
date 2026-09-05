# Brief: Retro FPA template — porting Godot → Unity

## Contesto
Esiste già un template Godot 4.6.3 ("Retro FPA") per giochi first-person
horror/adventure low-poly (PS1/N64/GameCube). Vogliamo l'equivalente
funzionale su **Unity 6.3 LTS + URP**, non un porting letterale del codice
GDScript: stessa filosofia architetturale (sistemi decoupled via eventi,
contenuto data-driven, livelli costruiti posizionando prefab pronti),
reimplementata in modo idiomatico Unity.

## Workspace di questa sessione
Due cartelle sorelle, entrambe repo git indipendenti, aperte insieme in un
workspace VSCode multi-root:

- `retrofpa-core/` — **package UPM**: il "motore" riutilizzabile.
  ```
  retrofpa-core/
    package.json
    Runtime/   ← manager (GameManager, SceneManager, SettingsManager, ...),
                 componenti (Interactable, Grabbable, Collectible, Rotator,
                 FresnelPulse, DialogueTrigger, SceneChangeTrigger, SpawnPoint...),
                 ScriptableObject (ItemData, DialogueData, VisualStyleProfile,
                 EquippableBehavior + sottoclassi melee/ranged...), shader
    Editor/    ← dock/EditorWindow, PropertyDrawer custom, wizard di
                 scaffolding, validator (equivalenti di addons/dialogue_tools
                 e addons/item_tools in Godot)
    Samples~/  ← prefab base pronti da importare per progetto
                 (NpcBase, WorldItemBase, SceneChangeTrigger, SpawnPoint...)
  ```
  Versionato a tag semver. Nessun contenuto specifico di un gioco qui dentro.

- `retrofpa-project-template/` — **progetto Unity vero**: URP configurato,
  Input System (nuovo), Localization package, Scene Template package.
  Referenzia retrofpa-core in `Packages/manifest.json` via percorso file
  locale mentre si sviluppa (`"com.yourstudio.retrofpa": "file:../retrofpa-core"`),
  da convertire in URL git+tag più avanti quando serve la modalità
  "aggiornamento per-progetto" invece che sviluppo in coppia. Contiene anche
  gli Scene Template veri e propri (es. "Livello base" con Volume/fog già
  impostato, SpawnPoint, riferimenti cablati) e un piccolo livello demo
  costruito con i prefab del core, come prova che tutto funziona insieme.
  In futuro, da questo stesso progetto si può anche generare un **Unity
  Project Template** installabile in Hub (package.json + cartella
  ProjectData~ + .tgz copiato nella cartella ProjectTemplates dell'Editor) —
  passo opzionale, da fare per ultimo, non blocca lo sviluppo.

## Perché questa struttura (non rider derivare, è già deciso)
- Godot: autoload singleton + signal, Resource (.tres) data-driven, shell
  persistente (main.tscn) con livelli caricati/scaricati come figli di un
  nodo "CurrentLevel". Su Unity: singleton MonoBehaviour con
  DontDestroyOnLoad + eventi C#, ScriptableObject al posto di Resource,
  scena persistente + scene di livello caricate SEMPRE in additive tramite
  SceneManager.LoadSceneAsync/UnloadSceneAsync (mai Single, romperebbe la
  scena persistente).
- I nodi "self-building" di Godot (@tool script che si costruiscono da soli
  in editor, es. WorldItem/NpcBody/SceneChangeTrigger) NON vanno riportati
  come pattern: Unity risolve lo stesso bisogno meglio, con **Prefab
  Variant** veri (mesh/materiale/animazioni visibili subito in Scene view,
  zero codice di costruzione a runtime). Se serve un'azione tipo "rigenera
  il collider in base alla mesh", va fatta come bottone in un Custom Editor
  chiamato esplicitamente, mai in OnValidate automatico.
- Lo stile "retro" del progetto Godot NON usa vertex snapping/affine texture
  warping vero da PS1 — è ottenuto con: nearest-filter + niente
  anisotropic + downsample texture forzato, un Environment (fog/ambient/
  tonemap/color grading/glow) applicato da un profilo dati, un materiale
  custom a 2 layer texture indipendenti (tiling/offset/scroll, blend
  Alpha-Over/Multiply/Additive/Subtract/Divide) con fresnel rim-light
  sempre attivo + un "flash" additivo pilotato da codice per gli hit-flash
  (mai animato su un materiale condiviso — va reso locally-unique prima,
  su Unity: MaterialPropertyBlock invece di clonare il Material).
- Su Unity: VisualStyleProfile come ScriptableObject che scrive in un URP
  VolumeProfile (Fog/Color Adjustments/Bloom override) invece che in un
  Environment Godot; shader custom a 2 layer + fresnel + flash come Shader
  Graph URP Lit (preferito per velocità di sviluppo; HLSL a mano solo se
  serve parità matematica esatta); cambio stile = assegnare un
  VisualStyleProfile diverso a uno StyleManager singleton.
- AnimationSet di Godot (mappa stato logico → nome clip reale, per
  personaggio) → **AnimatorOverrideController** nativo di Unity: un
  Animator Controller condiviso definisce gli stati logici con clip
  placeholder, ogni personaggio ha un override controller che rimappa sulle
  clip vere. Non serve un ScriptableObject custom per questo, Unity lo offre
  già pronto.
- Editor tooling di Godot (dock, dropdown dinamici al posto di id liberi,
  wizard di scaffolding, validator) → copertura diretta con EditorWindow,
  CustomPropertyDrawer, ScriptableWizard, MenuItem statico che scansiona
  AssetDatabase — nessun compromesso, va semplicemente scritto in C#.

## Ordine di sviluppo consigliato (step piccoli, uno alla volta, commit separati)
1. Scheletro dei due repo (package.json, manifest.json con dipendenza
   file-locale, .gitignore Unity, Assembly Definitions Runtime/Editor)
2. GameManager + SceneManager (stato di gioco, load additivo scene, player
   persistente, SpawnPoint)
3. VisualStyleProfile + VolumeProfile URP + StyleManager (cambio stile)
4. Shader Graph a 2 layer + fresnel + flash, componente FresnelPulse
5. Prefab base: NpcBase (Animator + AnimatorOverrideController slot),
   WorldItemBase (Physical/Pickupable), SceneChangeTrigger, SpawnPoint,
   componenti sciolti (Rotator, Interactable, Collectible)
6. InventoryManager + ItemData + EquippableBehavior (melee/ranged)
7. DialogueManager + DialogueData (riscritto da zero, runner leggero come
   in Godot, non una libreria esterna — il valore è nel tool di authoring)
8. Editor tooling: dock/wizard/validator per dialoghi e oggetti
9. Scene Template Unity + eventuale Project Template per Hub (per ultimo)

## Non fare
- Non replicare il pattern "nodo che si autocostruisce in editor" (Godot
  @tool) con OnValidate — fragile e non necessario, Unity ha i Prefab.
- Non animare mai un Material condiviso direttamente — usare sempre
  MaterialPropertyBlock per effetti per-istanza (flash, pulsazione fresnel).
- Non usare LoadSceneMode.Single per i livelli — romperebbe la scena
  persistente. Sempre Additive con unload esplicito del livello precedente.
