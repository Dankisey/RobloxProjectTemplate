# Framework public API

The yield contracts here are mandatory: purchase delivery depends on them ([saves](saves.md#purchases)).

```lua
SavesLoader:IsLoaded(player)            -- the profile session is active
SavesLoader:Save(player)                -- does not yield; false = no session to write to
SavesLoader:RequestSave(player)         -- never yields: allowed in a purchase handler
SavesLoader:WipeSave(player)            -- wipes the whole save and kicks; false = no session

Monetization:RegisterProduct(productId, handler)  -- the handler MUST NOT YIELD
Monetization:RegisterPass(passId, handler)        -- called by the service that owns the pass effect; after a pass purchase and on a free developer purchase; ownership on join is read by that owner
Monetization:PromptProduct(player, productId) / :PromptPass(player, passId)  -- only a whole id > 0 with a registered handler: an unregistered pass cannot be bought
Monetization:AreDeveloperPurchasesEnabled()       -- -> boolean; the module attribute DeveloperPurchasesEnabled
Monetization:SetDeveloperPurchasesEnabled(isEnabled)  -- server wide, resets to false on a new server
Monetization:GetDeveloperPurchasesChangedSignal() -- -> RBXScriptSignal

RewardService:GiveReward(player, {FunctionName = ..., Data = ...})
SoftCurrencyService:GetCurrency(player, currencyName)          -- -> number?; nil = save not loaded
SoftCurrencyService:AddCurrency(player, currencyName, amount)  -- -> boolean; also adds to Earned
SoftCurrencyService:TrySpendCurrency(player, currencyName, amount, transactionType, sku?)

LeaderboardService:UpdateTotals(player)  -- blocks: writes changed scores now, nothing before IsLoaded; the loop calls it every UpdatePeriod
LeaderboardService:Refresh()             -- blocks: reads the pages of boards present on this server

RagdollService:Knockdown(player, {Duration, Immunity, Impulse} | duration)
RagdollService:SetRagdoll(player, enabled, impulse)  -- no timers, for scenes
RagdollService:IsImmune(player) / :GrantImmunity(player, duration)
RagdollService.RagdollStarted / .RagdollEnded        -- Event

-- server NPC movement, not a service: ServerStorage.PathfindingAgent
PathfindingAgent.new(options)           -- asserts: Humanoid, HumanoidRootPart, rig in Workspace; server network owner
agent:MoveToward(targetPosition, {RepathDistance}?)  -- blocks while a path is computed: straight when visible, SimplePath otherwise
agent:Stop() / agent:HasLineOfSight(targetPosition)
agent:RunLoop(think, locomote, thinkInterval, locomotionStep)  -- blocks until the agent is destroyed; errors are warned, not raised
agent:Destroy()                         -- safe twice, destroys the model; Humanoid.Died calls OnDied, then Destroy
agent.Model / .Humanoid / .Root / .IsAlive
PathfindingAgent.TryTakeToken()         -- -> boolean; shared path budget per frame

ToolsController:RegisterAction(toolType, onUse, onRelease?)  -- called by the tool owner
ToolsController:UseEquippedTool() / :ReleaseEquippedTool()
ToolsController:GetEquippedTool() / :GetEquippedToolType()
ToolsController:SetToolsBlocked(isBlocked) / :AreToolsBlocked()

CharacterMovementController:EnableMovement() / :DisableMovement()
CharacterMovementController:RefreshSpeed()  -- recalculates speed only while movement is enabled

ButtonsInteractionsConnector:ConnectButton(button, callback, ...)        -- callback on press, bounce
ButtonsInteractionsConnector:ConnectHoldButton(button, onPress, onRelease) -- hold: the end of the same input
ButtonsInteractionsConnector:DisconnectButton(button)

-- client Robux prices, a shared session cache by type and id; no method blocks
PriceController:GetPrice(id, infoType)                  -- -> number?; nil = not loaded yet, starts loading
PriceController:AttachLabel(label, id, infoType, format?) -- -> boolean; %s = glyph + price, "..." while loading, "??" after failures
PriceController:DetachLabel(label)                      -- call before replacing the price with OWNED or another caption

KeyBadge.Apply(badge, hint)             -- label by UserInputService.PreferredInput; hint is a ControlsConfig section
ProgressBar.new(frame, maxValue, useText?)  -- :SetValue / :SetMaxValue / :SetCustomText; frame.Bar fill, zero max gives an empty bar
CooldownMask.new(maskFrame, cooldownTime)   -- :Activate(time?) / :Cancel; draws only, the caller owns the timer
Slider.new(bar, initialPercent?)            -- share 0..1: .Percent, .Changed (Event), :SetPercent, :Destroy; bar.Knob, optional NumberValue Percentage
GenerateViewport(viewport, model, distance?, yawDegrees?)  -- -> Camera?; nil = the model has no PrimaryPart
KeyBadge.Bind(badge, hint)              -- -> RBXScriptConnection; Apply now and on every PreferredInput change

SoundPlayer:PlaySound(soundName, onPart?) -- -> Sound?; an empty SoundId is silently skipped

RagdollController.IsRagdolled           -- a FIELD, not a method
RagdollController:ApplyState(target, enabled, impulse?)  -- pose + blocking + camera

FormatNumber(value)                     -- 150, 1.38K, 13.8K, 138K: three significant digits, truncation
```

A mechanic that a ragdoll must abort (a button hold, a mini game) subscribes to `RagdollService.RagdollStarted` and aborts itself: the service does not know about it.
