# Full Crew 0.14.1

A R.E.P.O. mod that gives larger groups more valuables to collect, with optional extra rooms, partial extraction loads, and a load counter.

**Status:** experimental prototype. Built against Steam build 23363152. Compilation, math tests, and offline checks against the installed game code pass. Live room generation, physics, guest synchronization, UI placement, and extraction payouts still need verification. “Host-only” describes the implementation design, not a completed multiplayer certification.

## Requirements and who installs it

- The host needs **BepInExPack** and **Full Crew**.
- Guests do not need Full Crew for the intended gameplay behavior.
- Guests may install Full Crew to see the synchronized extractor counter. Use the same version on host and modded guests.
- REPOConfig is optional. The settings use supported BepInEx types, but its in-game menu integration has not been live-tested.
- REPOLib and Timer Plugin are not required. Full Crew creates its own map section using native game text assets. Timer Plugin 1.3.2 was inspected for layout compatibility; live coexistence still needs testing.
- A lobby-expanding mod is separate and may have its own installation requirements. Full Crew does not raise the player limit.

## Install or update through Gale

1. Close R.E.P.O.
2. Select R.E.P.O. and your active profile in Gale.
3. Install BepInExPack if the profile does not already have it.
4. Extract the Full Crew release ZIP anywhere convenient.
5. If updating, remove the old Full Crew mod entry. Keep a copy of your configuration if needed.
6. Choose **Import → Local mod** and select `BepInEx/plugins/FullCrew/FullCrew.dll` inside the extracted ZIP.
7. Keep only one Full Crew DLL installed, then launch the modded game through Gale.

The release contains the plugin DLL, not BepInEx or the game's assemblies. First launch creates the configuration and adds new settings. Updates preserve existing saved settings; new defaults do not overwrite your choices.

## Where to configure it

Inside the active Gale profile, open:

```text
BepInEx/config/local.fullcrew.cfg
```

Close the game, edit with Notepad, save, then relaunch through Gale. You can also use Gale's configuration editor. For reliable results, change gameplay settings before starting the next level.

Testing overrides are off by default. The host controls gameplay settings. `Active Extractor Information Position` is a local preference on each computer.

## Default configuration

These are the defaults for a fresh 0.14.1 configuration—not necessarily the values already saved in your profile.

```ini
[Scaling]
Make Large Items Smaller = false
Keep Vanilla Total Map Value = true
Enabled = true
BaselinePlayers = 6
MaxTotalItems = 150

[Loot Item Type Distribution]
Tiny Item % = 50
Small Item % = 35
Medium Item % = 15
Large Item % = 0

[Map]
ExtraRooms = 0

[Extraction]
RepeatableLoads = true
Items Per Load = 25

[UI]
Active Extractor Information Position = Middle Left
Active Extractor Font Size = 18
Active Extractor Visibility = Everywhere

[Testing]
SimulatedPlayerCount = 0
MinimumExtractionPoints = 0
```

## How a level is generated

### 1. Build the map

With `ExtraRooms=0`, the normal map layout is used.

With a positive value, Full Crew requests that many extra room modules before normal placement. It recalculates the requested extraction-room count using vanilla's room thresholds. This happens before loot generation, not in response to running out of loot spots afterward.

### 2. Let vanilla choose the original loot

Vanilla generates its normal selection for that map and level. The normal initial valuables pass has a ceiling of 50, but value budgets, size limits, random selection, and available spawn locations often produce fewer.

Full Crew uses the number that actually spawned. It does not assume that every level starts with 50.

### 3. Calculate the total item target

```text
Player multiplier = max(1, players / BaselinePlayers)
Requested total = round UP(original count × player multiplier)
```

`MaxTotalItems` limits additions, but Full Crew never deletes original loot just to meet that cap. Available spawn locations can limit additions further.

Example: vanilla generates **10**, and the multiplier is **3×**. The target is **30 total**, so Full Crew attempts to add **20**. It does not add 30 on top of the original 10.

The normal generation result is taken from the expanded map if extra rooms are enabled. More rooms can change that result, but do not guarantee more original loot because vanilla's value and count limits still apply.

### 4. Add valuables using the configured size mix

Only extras use the configured size mix. Vanilla's original items can still include all seven categories: Tiny, Small, Medium, Big, Wide, Tall, and VeryTall.

The default target for extras is:

| Category | Share |
|---|---:|
| Tiny | 50% |
| Small | 35% |
| Medium | 15% |
| Large | 0% |

For 20 extras, that targets 10 tiny, 7 small, and 3 medium when all categories have enough spots.

The mod selects an eligible size, then prefers unused matching spots farther from existing loot. It does not create arbitrary positions or guarantee equal loot in every room. If a category runs out, another eligible category is used. At the default Large Item %=0, running out of tiny/small/medium spots stops spawning. A positive Large Item % allows large extras; their subtype is selected using vanilla chances and suitable large spawn volumes.

### 5. Apply the selected value mode

- `Keep Vanilla Total Map Value=true`: rebalance original and added item values to preserve the original total money.
- `Keep Vanilla Total Map Value=false`: keep original and added items at their normal rolled values, allowing more money and higher vanilla quotas.

### 6. Play with the selected extraction behavior

If repeatable loads are enabled, each extractor can bank partial loose-item loads toward its existing quota. The allowance is based on the actual starting valuable count, physical extractor count, and Items Per Load setting. It is frozen once generation finishes. The map menu shows the load allowance, current load, next-load threshold, and full extractor quota.

## Every configuration option

### [Scaling] Enabled

**Default:** `true` · **Options:** `true` / `false`

Enables Full Crew's host-side generation and extraction changes. Set false before generating a new level to use vanilla behavior. It does not undo rooms, loot, or partial loads already generated in an active level.

### [Scaling] BaselinePlayers

**Default:** `6` · **Range:** `1–100`

The player count that corresponds to a 1× multiplier. Smaller groups also use 1×; the mod does not remove loot for them.

| Players | Multiplier with baseline 1 | Multiplier with baseline 6 |
|---|---:|---:|
| 1 | 1× | 1× |
| 2 | 2× | 1× |
| 3 | 3× | 1× |
| 4 | 4× | 1× |
| 5 | 5× | 1× |
| 6 | 6× | 1× |
| 7 | 7× | About 1.17× |
| 8 | 8× | About 1.33× |
| 12 | 12× | 2× |

This multiplier affects the item target and optional large-item shrinking. Extraction loads now depend on actual generated loot count instead. Player count is captured at host loot generation. Joining or leaving mid-level does not recalculate these settings.

### [Scaling] MaxTotalItems

**Default:** `150` · **Range:** `1–500`

The maximum total the mod attempts to reach through additions, including the original valuables.

Example: 40 originals at 6× request 240 total. With a cap of 150, the target becomes 150, so at most 110 extras are added. Spawn capacity may produce fewer.

If the originals already exceed your configured cap, they are retained and no extras are added. This setting is not a lifetime limit on enemy drops or other later-spawned objects.

### [Loot Item Type Distribution]

| Setting | Default | Range |
|---|---:|---:|
| Tiny Item % | 50 | 0–100 |
| Small Item % | 35 | 0–100 |
| Medium Item % | 15 | 0–100 |
| Large Item % | 0 | 0–100 |

These four independent settings control the size mix of **added valuables**. They do not change the original vanilla items. Small no longer fills the remainder automatically.

Aim for a total of 100. Other totals are used proportionally: 20/30/10 gives approximately 33% tiny, 50% small and 17% medium. Setting all four to zero uses the default 50/35/15/0 mix. The saved values are not automatically rewritten.

These are target shares, not hard exclusions. If positive-share categories run out of usable spots, a zero-share category can still be used as a fallback. **Large Item %=0 strictly excludes large extras, including fallback spawning.** It does not remove vanilla's original large valuables. With a positive share, Large includes Big, Wide, Tall and VeryTall. Eligible subtypes use vanilla's highest-random-roll selection with their native chance values and suitable volume/prefab pools. Their level-based count caps are multiplied by the loot multiplier, rounded up, with original items counted against them. Subtypes with a vanilla cap of zero stay excluded. This lets vanilla rules determine the large mix without introducing four more settings.

### [Scaling] Make Large Items Smaller

**Default:** `false` · **Options:** `true` / `false`

- **False:** original valuables keep their normal dimensions.
- **True:** Big, Wide, Tall, and VeryTall dimensions are divided by the player multiplier.

With baseline 1:

| Players | Large-item dimensions when enabled |
|---|---:|
| 1 | 100% |
| 2 | 50% |
| 3 | 33% |
| 4 | 25% |
| 5 | 20% |
| 6 | About 17% |

This affects dimensions along all three axes, not just volume. Mass and durability remain unchanged for both original and added items. Tiny, Small, and Medium are not shrunk.

Shrinking follows the requested player multiplier even if the item cap or map capacity prevents all extra loot from spawning. It is not an automatic “shrink only when too large for the extractor” feature. Keep it false to preserve the original items' sizes.

### [Scaling] Keep Vanilla Total Map Value

**Default:** `true` · **Options:** `true` / `false`

#### True: more objects, same total money

The original haul establishes the budget. After adding loot, a common factor reduces values across both original and added valuables. Integer rounding preserves that original budget.

Example: original loot totals $30,000, and extras have $15,000 in normal value. The combined $45,000 is rebalanced down to $30,000. An item that would be worth $900 becomes approximately $600.

Balancing uses what actually spawned. Limited map capacity does not reduce the original total money just because the requested target was missed. Expensive items retain their relative value, subject to rounding.

#### False: normal item values, more available money

Original items and extras keep their normal rolled values. In the same example, $45,000 remains available.

Vanilla calculates its total quota using the generated loot value, so the quota generally rises too. At the same level, a 50% increase in total value generally produces roughly a 50% increase in total quota, subject to rounding. This adds both earning potential and required dollar value; it does not provide extra rewards while freezing the quota.

More items do not guarantee exactly proportionate money, because tiny, small, and medium valuables have different values and available spots may limit additions. Actual earnings depend on successful extraction, damage, and vanilla's reward rules. More money can accelerate purchasing progression.

There is no separate quota multiplier or “keep original quota” setting in this version.

### [Map] ExtraRooms

**Default:** `0` · **Range:** `0–10`

- `0`: normal map layout.
- `3`: request three additional room modules.

This is a fixed addition per generated map, not an addition per player. Rooms are added before loot is generated, using the existing grid and room-placement process. The mod does not enlarge that grid. Reachable grid capacity limits additions; small special scenes and debug layouts are excluded.

Extraction-room requests follow vanilla thresholds for the expanded module count:

| Room modules | Additional extraction rooms requested | Total if all requested rooms are placed |
|---|---:|---:|
| Up to 5 | 0 | 1 |
| 6–7 | 1 | 2 |
| 8–9 | 2 | 3 |
| 10–14 | 3 | 4 |
| 15+ | 4 | 5 |

The starting extractor accounts for the first one. The generator must still find suitable locations for the additional extraction rooms.

More rooms can mean more exploration and loot locations. They do not guarantee the full item target, and they do not automatically get added when spawning later runs out of space.

### [Extraction] RepeatableLoads

**Default:** `true` · **Options:** `true` / `false`

- **False:** normal one-completion extraction behavior.
- **True:** each extractor allows `ceil(actual starting items / actual extractor count / Items Per Load)` loads, minimum 1, toward its existing quota.

### [Extraction] Items Per Load

**Default:** `25` · **Range:** `1–500`

| Actual starting valuables | Physical extractors | Loads per extractor at 25 items/load |
|---:|---:|---:|
| 25 | 1 | 1 |
| 26 | 1 | 2 |
| 100 | 2 | 2 |
| 100 | 4 | 1 |
| 150 | 2 | 3 |

This is an allowance calculation, not a requirement to place exactly 25 items on the pad. Money still triggers each load. The item count includes original and added starting valuables, after capacity limits. Later destruction, collection, enemy drops and player joins do not recalculate the allowance. More extractors spread the same starting item count across more points.

For a $9,000 quota and three allowed loads:

| Stage | Delivered loose valuables | Result |
|---|---:|---|
| Load 1 | $3,000 | Items clear; $3,000 is credited |
| Load 2 | $3,000 | Items clear; credit reaches $6,000 |
| Load 3 | $3,000 | Normal final extraction and payout |

Partial loads begin a three-second alarm countdown when their loose-value threshold is reached. The tube then closes and items clear at closure, approximately 3.7 seconds after the warning begins. Removing loot below the threshold cancels the pending partial load without clearing or banking it. The next threshold is the remaining quota divided by remaining loads, rounded up. Oversized partial loads receive their full value as credit, so the next threshold can be lower.

Delivering the full quota at once triggers normal completion immediately. You are not required to make the maximum number of loads. Each extractor has its own allowance, and multiple loads do not multiply that extractor's quota.

Partial banking applies to loose valuables. Boxed valuables and cosmetic objects remain for normal final extraction. Partial loads play a warning countdown followed by a cosmetic tube cycle on the host and guests with Full Crew 0.14.0. They do not enter the full vanilla completion state, award currency, count as completed extraction points, or trigger final-extraction revival behavior. The extractor stays active between loads.

The implementation uses the game's synchronized credit field for banked value and leaves completion and payout to the final vanilla path. This is intended to avoid duplicate rewards; it still needs live verification, particularly for surplus refunds and final-extractor payouts.

### [UI] Active Extractor Information Position

**Default:** `Middle Left` · **Scope:** each computer independently

Options: `Top Left`, `Middle Left`, `Bottom Left`, `Top Right`, `Middle Right`, `Bottom Right`. The setting moves the map section immediately and does not affect other players.

Shows an **Active Extractor** section at the selected position in the **map menu**, using the game's Upgrades heading and body font assets. Labels are warm yellow and values are white. The old proximity/facing overlay has been removed.

```text
Active Extractor
Number of Loads: 8
Current Load: 1 of 8
Current Load Quota: $3,750
Extractor Quota: $0 / $30,000
```

- **Number of Loads:** maximum allowance for this extractor, not a required number of deliveries.
- **Current Load:** the next load to bank, including the final load. It advances when items are actually cleared and credited.
- **Current Load Quota:** loose value required on the pad to trigger the next partial load; on the last load, the remaining full quota. It is not the amount still missing after the valuables currently on the pad. The threshold uses `ceil((extractor quota - banked credit) / remaining loads)`.
- **Extractor Quota:** banked/extracted credit **/ full quota** for this point. Valuables waiting on the pad are not yet extracted. The completed amount is capped at the quota; surplus refunds remain part of vanilla final extraction.

The section shows `No active extractor` when no point is selected. In Everywhere mode it follows the active point without requiring proximity or looking at it. Extractor Room Only mode hides the section until the local player is in the active point's room/module. If a full quota is delivered at once, vanilla final extraction takes over, even with unused loads remaining.

The host publishes its authoritative thresholds and active-point selection through display-only metadata. Guests with Full Crew **0.14.0** receive this data; use matching versions. Guests without Full Crew can play without the added map section or partial-load warning animation. The old ShowLoadCounter toggle is removed; the section appears only while the mapped map control (Tab by default) is held or toggled open. It is hidden whenever the host has RepeatableLoads disabled.

Data refreshes roughly twice per second, plus network delay. Visibility follows the native map. Full Crew owns its own text objects; it does not patch Timer Plugin, load its assembly, or alter its settings/UI. Placement follows the selected screen region. Choose another position if your other map information overlaps it; Full Crew does not move another mod's text. Fonts are read from native game assets; no Timer Plugin code or assets are bundled. Custom positions and crowded layouts still need live visual checks.

### [UI] Active Extractor Font Size

**Default:** `18` · **Range:** `9–100`

Controls body text. The heading is 10% larger for body sizes below 50 and 5% larger for sizes 50 or above: 18 gives 19.8, 50 gives 52.5, and 100 gives 105. A panel larger than the screen is scaled to fit.

### [UI] Active Extractor Visibility

**Default:** `Everywhere` · **Options:** `Everywhere` / `Extractor Room Only`

Both options require the map menu to be open and repeatable loads to be enabled on the host. Everywhere permits the map section anywhere on the level. Extractor Room Only additionally checks the local player's room against the active extractor's module. No active extractor means no section in room-only mode. These UI settings are local, including on modded guests.

### [Testing] SimulatedPlayerCount

**Default:** `0` · **Range:** `0–100`

Zero uses the actual player count. A positive value substitutes that many players only in Full Crew's loot target, optional size scaling, and extraction-load allowance. It does not spawn bots, change lobby capacity, change the game's networking readiness checks, or simulate other players' movement or enemy interactions. It works for the host in solo or multiplayer test runs.

The normal baseline still applies: simulated count 3 with baseline 1 gives 3x; simulated count 3 with baseline 4 gives 1x. Simulated count 6 with baseline 4 gives 1.5x loot; the actual generated count determines loads. Set the value before generating the level, and return it to 0 for normal play.

### [Testing] MinimumExtractionPoints

**Default:** `0` · **Range:** `0–5`

Zero keeps normal extraction-room rules. A positive value requests at least that many total extraction points, including the starting point. The mod also requests enough main room modules to reach that extractor count's vanilla threshold: 6 for two, 8 for three, 10 for four, and 15 for five.

This testing override can request additional rooms even with ExtraRooms=0. When both settings are used, it takes the larger room requirement rather than adding the requirements together. It does not reduce an already higher natural extraction count. Small special scenes and debug layouts are still excluded.

It is a request, not guaranteed placement: the existing grid and eligible extraction tiles remain finite. Full Crew logs the actual extraction-point count after generation and warns if the requested minimum was not reached. Do not assume a multi-extractor test is valid until that count is confirmed. Return this setting to 0 for normal play.

## Solo testing: partial loads and overflow

For a solo simulation of three players with at least two requested extraction points:

```ini
[Scaling]
Make Large Items Smaller = false
Keep Vanilla Total Map Value = true
Enabled = true
BaselinePlayers = 1
MaxTotalItems = 150

[Map]
ExtraRooms = 0

[Extraction]
RepeatableLoads = true
Items Per Load = 25

[UI]
Active Extractor Information Position = Middle Left
Active Extractor Font Size = 18
Active Extractor Visibility = Everywhere

[Testing]
SimulatedPlayerCount = 3
MinimumExtractionPoints = 2
```

Edit the existing sections rather than appending duplicate section headings. The testing override requests any necessary room increase despite ExtraRooms=0.

1. Start a fresh test run. Check `BepInEx/LogOutput.log` for actual players=1, simulated players=3, multiplier=3x, then the separate Load allowance log for the actual item count, extractor count and resulting loads.
2. Check `Generated layout: ... extraction points` and confirm at least two actually spawned. If the request was not met, try another generated map or test later in a run with a larger layout.
3. Activate the first extractor and open the map and verify Current Load agrees with the Load allowance log. If the map has too few items for partial loads, temporarily lower Items Per Load for testing. Note that extractor's own quota, not the entire level quota.
4. Deliver the Current Load Quota in loose valuables. Expect three alarm beats before the tube closes and banks the load, without payout or unlocking another point. On another attempt, remove items during the warning: falling below the threshold must cancel it without banking. Carry the items back and retry.
5. Test another partial load. Then deliberately exceed the remaining quota on the final load with enough loose value to create an overpayment.
6. On an earlier extractor, expect the normal completion reward and a carryable tax-return valuable for the excess. Expected excess is total value banked plus final delivered value minus that extractor's quota, measured after any item damage. This expectation still requires live verification.
7. Take the refund to the next point and confirm it contributes correctly. Verify completion is possible and no credit was duplicated or lost. Test the last extractor's payout separately.
8. Repeat with Keep Vanilla Total Map Value=false to test increased total value and quotas after the basic balanced case works.

Solo testing checks local generation, banking, and money behavior. It does not validate an unmodded guest's synchronization or optional guest UI. If a test fails, keep the log and note the quota, banked amounts, final delivered value, and refund value.

After testing, set both testing options back to 0 before generating a normal level. This does not undo an already generated test level.

## Worked example: rooms off versus rooms on

Assume three players, baseline 1, item cap 150, the default size mix, no shrinking, and repeatable loads enabled.

### Rooms off

1. Vanilla generates its normal map.
2. Suppose it spawns 10 valuables.
3. Full Crew targets 30 total, adding up to 20 in unused tiny/small/medium spots.
4. If capacity permits, the extras target 10 tiny, 7 small, and 3 medium.
5. Original item sizes stay normal.
6. If all 30 items spawn and there is one extractor, it has two loads at 25 items/load. With two extractors it has one load each.
7. Total money stays at the original budget when Keep Vanilla Total Map Value=true; otherwise normal-valued extras increase available money and influence vanilla quotas.

### Rooms on: ExtraRooms=3

1. The generator requests three more room modules, subject to grid capacity.
2. Extraction-room requests are recalculated using the resulting module count.
3. Vanilla chooses its initial loot on that expanded map. Suppose it now generates 14 items—this is an example, not a guaranteed increase.
4. Full Crew targets 42 total, adding up to 28. The same size mix, value choice, and capacity rules apply.
5. With 42 actual items and three extractors, each has one load at 25 items/load. More physical extractors distribute both the level quota and the starting item count.

## Configuration examples

### Start scaling above one player, preserve money and original sizes

```ini
[Scaling]
Make Large Items Smaller = false
Keep Vanilla Total Map Value = true
Enabled = true
BaselinePlayers = 1
MaxTotalItems = 150

[Loot Item Type Distribution]
Tiny Item % = 50
Small Item % = 35
Medium Item % = 15
Large Item % = 0

[Map]
ExtraRooms = 0

[Extraction]
RepeatableLoads = true
Items Per Load = 25

[UI]
Active Extractor Information Position = Middle Left
Active Extractor Font Size = 18
Active Extractor Visibility = Everywhere

[Testing]
SimulatedPlayerCount = 0
MinimumExtractionPoints = 0
```

### Allow increased rewards

Use the same configuration, changing only this entry under `[Scaling]`:

```ini
Keep Vanilla Total Map Value = false
```

### Request more map space

Change this entry under `[Map]`:

```ini
ExtraRooms = 3
```

## What the mod does not currently do

- Add a separate level-based multiplier. Vanilla progression still affects the starting map and loot.
- Guarantee a particular number of items when spawn space is exhausted.
- Invent spawn spots, expand the map grid, or append rooms after loot generation.
- Keep the original quota when increased loot value is enabled.
- Automatically resize only objects that fail an extractor-fit check.
- Guarantee that the entire haul fits on one platform simultaneously.
- Recalculate spawned loot when players join or leave mid-level.
- Change lobby size or remove other mods' client requirements.

## Upgrade notes

- Keep only one Full Crew DLL in the profile.
- Legacy TinyExtraPercent and MediumExtraPercent migrate to the new distribution section. The previously implied small share becomes an explicit Small Item % value; the old effective mix is preserved.
- Existing Make Large Items Smaller=true remains true. Set it false explicitly to retain normal original sizes.
- The old numeric LargeItemScale option from 0.2.0 is no longer used and can be removed.
- Keep Vanilla Total Map Value defaults to true, preserving the previous value-balancing behavior.
- First launch of 0.12.0 migrates old setting names and sections and removes ShowLoadCounter. A `.pre-0.12.bak` copy is saved beside the config first. Existing new-name settings take precedence. Your installed Gale profile was not modified when this ZIP was built.

## Testing and troubleshooting

### Completed offline checks

- Compilation against installed game assemblies and BepInEx/Harmony.
- Applying the production code-insertion hooks to the installed game's method instructions offline.
- Item targets, caps, size shares, fallback categories, and optional shrinking calculations.
- Value conservation across 500 randomized cases and preservation of normal values in increased-reward mode.
- Room-capacity calculations, vanilla extraction thresholds, and partial-load thresholds.
- Counter formatting and rejection of missing, malformed, or incompatible metadata, including quotas and active-point selection.
- Warning travel range, no immediate deletion in the threshold hook, revalidation before banking, removal of OnGUI overlay, no Timer Plugin assembly dependency, collision timing windows, and native damage API/field compatibility.

### Still needs live testing

Use a test run to check one host, an unmodded guest, and optionally a modded guest:

1. Confirm every player finishes loading and sees the same item values and sizes.
2. Check extra-room connections, extraction-room placement, navigation, and performance.
3. Bank partial loads; confirm no premature money, revival, or extraction completion.
4. Complete both an earlier extractor and the last extractor, with and without surplus; verify credit is counted once.
5. Test both value modes and confirm increased-value quotas and payouts.
6. Compare counters before banking, after banking, during completion, and on the next level.
7. Check all six map positions and independent guest position settings.
8. Check all six positions of the map section with Timer Plugin both installed and absent, including different resolutions and a long upgrade list.
9. Verify the warning delay, removing loot during a countdown, and synchronized closure on a modded guest.
10. In a disposable test run, check standing/tumbling player detection inside the closing extractor, escape during the warning, and no damage outside its volumes or after reopening. Repeat with a guest to verify native damage networking.

Lobby expanders, custom maps/valuables, other economy/extraction mods, late joining, and host migration remain unverified. The counter decoder validates display data but does not make those gameplay scenarios tested.

Check `BepInEx/LogOutput.log` for the Full Crew version, requested and actual loot counts, original and resulting total values, size counts, room requests, and banked loads. If a game update changes a required code hook, the plugin attempts to remove its patches and logs an unsupported-code error.

If fewer items appear than requested, check the item cap and size-compatible spawn capacity. If large objects still shrink, check for a saved Make Large Items Smaller=true. If a guest has no panel, confirm compatible Full Crew versions, and that the map is open; unmodded guests intentionally have no panel.

## Source and building

Source and build instructions are maintained privately. Testers use a compiled test build provided by the maintainer.

Game Assembly-CSharp SHA256 used for the build: `CE995A182DDC884EA965E87786F1986248D9616300FA825BCC04BCA671EE6526`.


## Partial-load warning and animation (0.11.0)

Earlier versions destroyed loose valuables immediately and played a 2.5-second cosmetic cycle afterward. Version 0.11.0 instead starts with the native alarm sound and a **3–2–1 countdown**, matching vanilla's three-second warning interval. The tube remains raised during this warning, closes over the following 0.7 seconds, clears and credits the eligible valuables at closure, then reopens. The complete cycle lasts approximately 5.5 seconds.

The host recomputes pad value during the warning and again before banking. If value falls below the threshold, the partial load cancels. If enough value is added to meet the full remaining quota, the partial cycle cancels and the normal vanilla completion sequence takes over. Boxed valuables and cosmetic objects still wait for normal final extraction.

The tube mesh is animated independently of the final-extraction state. During closure, the host queries the extractor's native damage-volume shapes: rim volumes while descending, then the interior volume while closed. A player still inside receives lethal damage through the native HurtOther network path, including guests without Full Crew. Standing and tumbling player colliders are detected. Damage is disabled throughout the three-second warning and once reopening starts. Native invulnerability rules still apply. The existing HurtCollider components are not enabled, preventing them from destroying valuables before banking. No extra completion rewards are triggered. Final extraction keeps its native behavior and warning sequence. Another partial load cannot begin until the current cycle finishes.

Modded guests use the host's shared start timestamp for the partial animation instead of starting an animation after items have disappeared. **Unmodded guests can be killed by partial-load closure, but do not receive Full Crew's additional partial-load alarm/countdown or tube visuals.** This is a current host-only presentation limitation; installing Full Crew on guests adds that warning. They still receive synchronized item removal and credit. A completed historical cycle is not replayed.

Compilation and offline tests pass. Live appearance, sound timing, item removal, UI placement, multiplayer synchronization, and payouts still require testing.

## Default baseline update (0.10.0)

Fresh configs now use BaselinePlayers=6. One through six players get normal loot quantity, seven target 7/6 times normal, and twelve target twice normal. This does not change the maximum number of players allowed to join. Existing configs retain their saved baseline: edit BaselinePlayers to 6 explicitly when upgrading if desired. Solo tests using SimulatedPlayerCount=3 should still set BaselinePlayers=1 to increase loot; the actual item count and Items Per Load determine whether partial loads are available.


## Settings update (0.12.0)

Added the Loot Item Type Distribution section with Tiny Item %, Small Item %, and Medium Item %. Moved Make Large Items Smaller and Keep Vanilla Total Map Value to Scaling. Removed ShowLoadCounter and added six positions for Active Extractor Information under UI. The map is still the only place this information appears. Timer Plugin is optional.

## Changes in 0.13.0 and cart diagnostics

- Loads now use actual starting item count and actual extractor count, with 25 Items Per Load by default.
- Added body font size, proportional heading size, map-input visibility, room-only visibility, hidden UI when repeatable loads are off, and extracted-value progress.
- Added Large Item %, default 0, using vanilla large subtype selection and size-compatible volumes. Shrinking still changes neither mass nor durability.
- Explicit tests verify that balanced values apply to the initial items as well as extras. **Damage loss is unchanged**: native percentages use the adjusted starting value; no extra multiplier reduction was added.
- Partial banking excludes carts explicitly. A failure clearing one valuable is logged and no longer aborts the rest of the batch.
- New logs identify partial-load warnings, loose items remaining, vanilla final extraction starts, and cart destruction requests with call stacks. A full quota can trigger final extraction on the first delivery; this is recorded even if load allowance remains.

The reported cart destruction has **not been reproduced or confirmed fixed**. These diagnostics preserve the vanilla cart-destruction request instead of making carts invulnerable. Keep BepInEx/LogOutput.log after the incident and before relaunching. Normal final extraction and objects outside the recognized collection volume can behave differently from partial banking.

Offline checks cover item-count boundaries, four-category mixes, heading thresholds, original-item value balancing, synchronized quota progress, and native field/hook compatibility. Live UI layout, room detection, cart behavior and multiplayer playtests remain required.

## Changes in 0.14.0: optional confirm-button integration

Full Crew automatically detects ExtractionPointConfirmButton 1.2.0. No new setting or required dependency is added.

- Without the button mod, ready partial loads still start automatically.
- With the supported button mod and RepeatableLoads enabled, a ready partial load waits for a button press. The Active Extractor map section shows “Ready - press confirm”.
- Pressing below the current load threshold does nothing. A valid partial press starts the existing warning countdown and tube cycle. Repeated presses during that cycle do not start another load.
- Removing enough loot during the warning cancels the partial load. If the full quota becomes satisfied during that warning, the partial cycle cancels and a fresh button press can authorize normal final extraction.
- At the full extractor quota, the button uses its normal final-extraction path. Full quota can finish an extractor before every load allowance is used.
- Shops and RepeatableLoads=false retain the button mod's own behavior.
- An unsupported button-mod version disables Full Crew's partial loads for that level and logs a warning, leaving normal final extraction available. Loot scaling remains enabled.

The button mod is optional, but its author requires everyone to install it when it is used: [ExtractionPointConfirmButton](https://thunderstore.io/c/repo/p/Zehs/ExtractionPointConfirmButton/). Full Crew itself still needs only the host for gameplay. Guests need matching Full Crew versions for the map information and partial warning animation; guests without Full Crew do not receive that partial animation even though host-authoritative crush damage still applies.

### Update and test

Close the game. In Gale, open the active profile folder and replace its existing BepInEx/plugins/FullCrew/FullCrew.dll with the DLL from this archive. Keep the existing configuration and other mods. Avoid duplicate FullCrew DLLs.

Use a test level with more than one load per extractor. Test below-threshold presses, a ready partial load waiting for confirmation, the countdown, removing loot during the countdown, repeated presses, and a full-quota final extraction. Also test a guest pressing the button and a run without the button mod.

Build, installed-game hook checks, display-protocol checks, and reflection against the published 1.2.0 button DLL passed offline. This integration has not yet been tested in a live multiplayer game. Cart destruction remains an unconfirmed issue; the diagnostic logging from 0.13.0 is retained.

## Changes in 0.14.1: optional loot-value diagnostics

A new setting records the actual values before and after Full Crew's generation-time adjustment:

```ini
[Debug]
Log Loot Values = false
```

Set this to true on the host before generating a level to investigate prices. After updating the DLL, launch and close the game once to create the new setting, or add this section manually while the game is closed. Existing configurations remain supported.

Search BepInEx/LogOutput.log for `[Loot values]`. One summary per level records actual/simulated effective player count, baseline, quantity multiplier, original/extra counts, original budget, before/after totals, percentage retained, and whether an adjustment was applied. Each item records its internal object name, local instance ID, size category, whether it was vanilla or extra, and its before/after values. Internal names may differ from the names players use.

For example, an item might log `before=$12,000.00; after=$6,500.00; difference=$-5,500.00`. This is illustrative, not a measured Golden Swirl result. All original and extra items are included. Runs with no price adjustment also log unchanged values. Whole-dollar allocation can make individual ratios differ slightly from the map ratio.

This setting defaults to false and writes at Info level when enabled so the normal log captures it. It only observes Full Crew's generation-time values; it does not record subsequent damage, enemy drops, or later price changes by other mods. It does not change loot or extraction behavior and does not require guests to install Full Crew. Copy the log before relaunching after a test. Set the option back to false when finished to reduce log volume.

Build and existing offline hook/calculation checks passed. Live game logging still requires a playtest.

