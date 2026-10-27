# Code map

The template place has 109 scripts. Added or removed a service or controller — update the table and the types ([code-style](../../rules/code-style.md#types)).

## Place tree

```
ServerScriptService.ServerBootstrap        → ServerCore:Start()
ServerStorage
  ServiceTemplate, ServerTypes, SpawnerTemplate, ProfileStore,
  PathfindingAgent, SimplePath           NPC movement, not services ([framework](framework.md#pathfindingagent))
  AdminGui                                 admin panel: AdminPanel (LocalScript), AdminConfig, CommandRequested
  ServerCore/                              10 services, below
ReplicatedStorage
  ClientCore/                              16 controllers, below
  Modules/                                 ClientTypes, SharedTypes, UtilsTypes, ControllerTemplate,
                                           Gizmos, UI/ (9), Utils/ (20)
  Configs/                                 20 configs
  Remotes/, GameValues/ (empty)
  Assets/                                  GuideBeam, UI/ (Popups — the Cash example)
StarterPlayer.StarterPlayerScripts.ClientBootstrap → ClientCore:Start()
StarterGui                                 MainGui, FadingGui, MessagesGui, PopupsGui, TooltipsGui,
                                           ConfirmationPromptGui, CustomPrompts (all ResetOnSpawn = false)
SoundService.SFX                           ClickSound, HoverSound (empty ids — silent)
Workspace                                  Baseplate, SpawnLocation, Debris
```

`StarterPlayer.CameraMode` = `Classic` (third person), `StarterGui.ScreenOrientation` = `LandscapeSensor`, `Workspace.StreamingEnabled` = `true`.

## Server (`ServerStorage.ServerCore`)

| Module | Responsibility |
|---|---|
| `SavesLoader` | ProfileStore session, hands sections to owners, join attributes; own `Stats` section; `WipeSave` for the admin panel |
| `Monetization` | routes developer product and game pass receipts, handler registry; free developer purchases behind the `DeveloperPurchasesEnabled` module attribute |
| `AdminService` | gives `ServerStorage.AdminGui` to `DevelopersConfig` players, checks and runs panel commands |
| `RewardService` | grants a reward from a `{FunctionName, Data}` description; knows `Cash` now |
| `SoftCurrencyService` | `Currencies` section, `Cash` attribute; adding and spending soft currency |
| `ServerMessagesSender` | server messages into the player interface |
| `Analytics` | `AnalyticsService` wrapper; reads the balance from `SoftCurrencyService` |
| `RagdollService` | ragdoll state, immunity, stand up timer, collision groups |
| `LeaderboardService` (+ `Leaderboard`, `Champion`) | global boards on `OrderedDataStore` from `LeaderboardsConfig.Stats`: reads player attributes, `Max` / `Set` writes, a page every `UpdatePeriod`; a memory store under `UseEmptySave` in Studio; off with empty `Stats` |
| `Leaderstats` | mirrors `LeaderboardsConfig.Leaderstats` attributes into the Tab list (`Cash`); a value appears once its attribute exists |

**Board model** (tag `Leaderboard`, attribute `Stat` = a `Stats` key): `GuiPart.BoardGui.LeaderboardFrame` with `ScrollingFrame.Template` (`Rank`, `Name`, `Value`), `Status`, optional `EmptyState`; optional `Podium` part with a `LeaderName` label inside turns the champion on. The template ships no board model.

`ServerStorage.ProfileStore` is the MadStudio library (not the legacy `ProfileService`). It is updated as a whole, the file is not edited.

## Admin panel (`ServerStorage.AdminGui`)

Outside DI on purpose ([decision](../../decisions/framework.md#admin-panel-outside-di)). Layout: `Toggle` at the right screen edge opens `Panel`; `Panel.Close`, `Panel.Content` (wrapped `UIListLayout` of transparent cells: `TitleCell`, `AmountCell` with `Field.Amount`, one cell per command button). `AdminConfig.Commands` lists the commands; the button in the panel has the command name. Default: `AddCash` (number from `Amount`), `FreePurchases` (label and color from `AdminConfig.FreePurchasesText` / `FreePurchasesColor`, state from the `DeveloperPurchasesEnabled` attribute mirrored on the panel clone), `Reset` (`NeedsConfirm`: second press within `ConfirmWindow`). A new command — an entry in `AdminConfig.Commands`, a copy of a command cell with the button renamed, a handler in `AdminService`.

## Client (`ReplicatedStorage.ClientCore`)

`GuiController` (+ `MainGui` with `CashLabel` — the `Cash` attribute, `FadingGui`, `MessagesGui`, `PopupsGui`, `TooltipsGui`, `ConfirmationPromptGui`), `ButtonsInteractionsConnector` (`ConnectButton` — press with bounce, `ConnectHoldButton` — hold), `ToolsController` (tap anywhere — `ControlsConfig.Use.TapAnywhere`, off), `RagdollController` (+ `RagdollCamera`), `CameraController`, `CharacterMovementController` (`RefreshSpeed` keeps disabled movement disabled), `InputController` (cursor hidden only in `LockFirstPerson`), `SoundPlayer`, `BackgroundMusicController`, `CustomPromptsController` (+ `CustomPromptTemplate`, `GuiOpenPrompt`, `DevProductPrompt` — the `ActionLabel` shows the price through `PriceController`), `PriceController` (client Robux prices: one cache, retries, labels released on success), `LeaderboardController` (champion dance on every client; off with empty `DanceAnimationId`), `TooltipsController`, `ClientMessagesSender` (shown by `MessagesHolder`), `GuideBeamController`, `ChatController`.

## UI modules (`ReplicatedStorage.Modules.UI`)

`GuiTemplate`, `AnimatedGuiTemplate`, `Widget`, `InfoPin`, `KeyBadge` (key label by `PreferredInput`), `ProgressBar`, `CooldownMask`, `Slider`, `GenerateViewport`. Not used by the template itself: they wait for the first game screen.

## Utilities (`ReplicatedStorage.Modules.Utils`)

```lua
CharacterUtils.GetAlive(player)      -- -> (Model?, BasePart?, Humanoid?)
CharacterUtils.GetAliveRoot(player)  -- -> BasePart?
CharacterUtils.GetTorso(character)   -- UpperTorso -> Torso -> HumanoidRootPart

FlatVector.Mask                          -- Vector3.new(1, 0, 1)
FlatVector.Distance(a, b)                -- flat distance
FlatVector.Flatten(vector, fallback?)
FlatVector.Direction(from, to, fallback?)

-- the only right way to raycast water and ground
WorldQuery.RaycastWater(origin, direction)
WorldQuery.IsOverWater(position, height?)
WorldQuery.RaycastGround(origin, direction, ignore?)  -- excludes the water tag itself
WorldQuery.GetWorldFolder(name)                       -- finds or creates
WorldQuery.CollectTaggedAndFoldered(tag, folderName)

TweenPivot(model, cframe, duration, easingStyle?, easingDirection?) -- -> Tween
LoadModules(folder)                      -- -> {[name] = module}, warns on failed ones
FormatNumber(value)                      -- 999, 1.38K, 13.8K, 138K, -2.5K: three significant digits, truncation
```

Others: `Event`, `Timer`, `GenerateGuid`, `GetChancedRewardIndex`, `Lerp`, `EvaluateGradient`, `FormatTime`, `DeepCopy`, `GetKeys`, `CubicBezier`, `TimeUtils`, `TranslateRange`, `MakeNumberDividers`, `GetPlayerProfileIcon`. Product prices come only from `PriceController`.

`Modules.Gizmos` lives **outside** `Utils` and is required lazily only in `InputController` under `TestingConfig.IsTesting` — [why](../../decisions/framework.md#gizmos-outside-utils).

## Configs (`ReplicatedStorage.Configs`)

`CameraConfig`, `CollisionConfig` (groups `Character`, `Ragdoll`), `ConflictingGuisConfig` (empty), `ControlsConfig` (`Use` section: keys, `TapAnywhere`, labels), `DevProductsConfig` (empty), `DevelopersConfig` (developer UserIds: admin panel and free developer purchases; empty), `FadingConfig`, `GamepassesConfig` (`Attributes` empty), `InputConfig`, `LeaderboardsConfig` (`Stats` empty, `Leaderstats` — `Cash`, texts, rank colors), `MessagesConfig`, `NumberPostfixes`, `PathfindingConfig` (`PathfindingAgent` defaults), `PopUpsConfig` (`BurstPopups` — popups that fly as a burst), `SavesConfig`, `SoundConfig` (`Sfx` with `PitchVariation`), `SoundNames`, `TestingConfig`, `ToolsConfig` (`Priority`, `Actions` empty), `WorldConfig` (water tag).

## Interface style

One look for every template window: a dark panel (a frame with a soft vertical gradient and a thin edge stroke), BuilderSans Bold and Medium, empty buttons with `Shadow`, `Fill` and `Fill.Label`. Colors: green — soft currency and confirm, red — destructive, blue-grey — neutral. A new window copies an existing one: `ServerStorage.AdminGui.Panel`, `StarterGui.ConfirmationPromptGui.Prompt`.

## Empty on purpose

The template knows no game, so these stay empty until a project fills them:

- `ClickSound`, `HoverSound` have no id and stay silent (`SoundPlayer` guard); the id goes on the instance, `SoundConfig.Sfx` mirrors it for reference.
- `ToolsConfig.Priority` and `Actions`: no tool is ever equipped, there is no on-screen use button.
- `LeaderboardsConfig.Stats` and the board model: a game adds a stat entry, publishes its attribute and places a tagged board.
- No NPC rig uses `PathfindingAgent`: a game creates agents from its own service.
- `GamepassesConfig.Attributes`, `DevProductsConfig`, `DevelopersConfig`, `ConflictingGuisConfig`; `Assets.UI.Popups` has only the `Cash` example.
- The game icon and name in Creator Hub.
