# Full Crew user acceptance test guide

**Version under test:** 0.14.1  
**Prepared:** September 24, 2026  
**Status:** Test plan only. No in-game tests in this document have been marked as passed.

Use this guide to check whether Full Crew works during actual play. Automated build and code checks are separate and do not prove multiplayer behavior. Start with the quick pass, then run the full relevant checks before a release.

**New to testing?** Copy [TEST-REPORT.md](TEST-REPORT.md) to record your session in plain language. Use [ISSUE-REPORT.md](ISSUE-REPORT.md) for each problem. For testing optional mods and your normal group profile, follow [MOD-COMPATIBILITY.md](MOD-COMPATIBILITY.md). You do not need to diagnose the code or finish every test in one session.

## Before you start

1. Create a separate Gale testing profile. Start with Full Crew 0.14.1 and required BepInEx components. Add other mods only when a test requests them; install their own dependencies normally.
2. Close the game before replacing FullCrew.dll or editing BepInEx/config/local.fullcrew.cfg. Keep a copy of your usual configuration. Ensure only one FullCrew.dll is installed.
3. Launch once to generate missing settings, then close and configure. Generate a fresh gameplay level after changing generation settings. Do not use a shop as the loot-generation test map.
4. Record the game build, map, level, Full Crew version, all other mod versions, actual players and settings. Different generated maps have different original loot; compare each map to its own logged original count/budget, not a previous map.
5. Copy BepInEx/LogOutput.log before relaunching. For multiplayer failures, collect host and affected guest logs. Review personal information before sharing logs.

## Finding saving and reading the log

The log is a text record of what BepInEx, Full Crew and other mods reported during a game session. You do not need to understand every line. The most useful thing you can do is save the correct session's file and describe what happened.

### Find the right file through Gale

1. Note which Gale profile you used to launch the test. If a bug just happened, do not start another game session yet.
2. In Gale, select R.E.P.O. and that same profile. Use the profile's folder-opening action (wording/location can vary by Gale version; look for an action to open the profile folder in its menu or settings).
3. In the File Explorer window that opens, open the **BepInEx** folder.
4. Find **LogOutput.log**. Its location relative to the profile is `BepInEx/LogOutput.log`. If Windows hides file extensions, it may display as **LogOutput**. It is not the `plugins` or `config` folder.
5. Check the file's **Date modified** against your test session. It should be recent. Later, check the startup lines for the expected Full Crew version too.

The profile's root directory can differ between computers and Gale installations, so use Gale's folder action instead of copying someone else's absolute path. A Steam game-folder log may belong to a different installation/session. For a Gale test, start with the active Gale profile.

### Save the evidence before launching again

1. Write down what happened, which level/extractor was involved, and roughly when. A video timestamp is useful if you have one; log lines may not contain clock times.
2. If possible, finish collecting evidence and exit the game normally. If it crashed, keep the existing file. Do not relaunch just to obtain a cleaner log.
3. Select LogOutput.log and press **Ctrl+C**. Open a folder for your test results and press **Ctrl+V**. Copy the file rather than moving it out of BepInEx.
4. Rename the copy to something recognizable, for example `2026-09-24-UAT12-host-cart-disappeared.log`.
5. Ask an affected guest to save their own copy too, with `guest` in its filename. Their log is on their computer; the host log does not contain every guest's local messages.

A new launch can replace the log. If you copy it while the game is still running, the copy only includes information written so far; save another copy after the event/session if needed.

### Open and search it

1. Right-click the saved copy, choose **Open with**, and select **Notepad** (or another text editor). If double-clicking asks which app to use, choose a text editor.
2. Press **Ctrl+Home** to go to the beginning. Look for the plugin-loading messages and confirm Full Crew's version is the one you intended to test.
3. Press **Ctrl+F**, type a phrase from the table below and use **Find next** or Enter. Continue through all matches; a session may contain several levels or extraction cycles.
4. Read a few lines before and after a match. If there is an error followed by lines beginning with `at`, those lines are its call history, called a stack trace. Keep the entire block.
5. Read the file without changing it. Record the relevant lines in your report and attach the full saved log where possible, rather than only a cropped screenshot.

| Search text | What it helps you find |
|---|---|
| `Full Crew` | Plugin version, startup and messages attributed to the mod |
| `[Loot values]` | Optional map adjustment summary and each item's before/after generation value |
| `Extra loot:` | Counts of extra tiny, small, medium and large valuables |
| `Generated layout:` | Actual extraction-point count, rather than just the requested minimum |
| `Load allowance:` | Actual item/extractor counts and calculated loads per extractor |
| `integration enabled` | Confirmation that supported button integration was enabled |
| `Confirm-button integration unavailable` | Unsupported or failed integration; partial loads disabled |
| `Confirmed partial load` | A button press accepted for a partial load |
| `Partial load warning:` | Automatic partial-load trigger details; button mode uses a different confirmation message |
| `Banked load` | Cleared value and cumulative credit from a partial load |
| `Loose items remaining after banking` | Loose objects still recognized after a partial clear |
| `Vanilla final extraction begins` | Normal final completion, even if unused load allowance remains |
| `Cart destruction requested` | A cart destruction request and its diagnostic call history |
| `Partial load could not clear item` | A per-item clearing failure |
| `Error` or `Exception` | Potential errors across all mods; not automatically caused by Full Crew |

### Understand a price example

These are invented, shortened examples to explain the fields, not actual test evidence:

```text
[Loot values] original budget=$10,000; before adjustment=$20,000.00;
after adjustment=$10,000.00; value retained=50%; reduction=50%

[Loot values] Item 1/60; source=vanilla; name="ExampleValuable";
instance=12345; size=Big; before=$3,000.00; after=$1,500.00;
difference=$-1,500.00
```

- **original budget:** the starting loot budget before Full Crew added items.
- **before adjustment:** total generated value including extras, before our price adjustment.
- **after adjustment:** total after our adjustment. With preservation enabled, this should match the original budget.
- **value retained:** the percentage of value kept, not the percentage removed. Retained 67% means reduced by about 33%.
- **source=vanilla / source=extra:** an original generated item or an item added by Full Crew.
- **name:** the game's internal object name; it may not match the nickname players use.
- **instance:** a local identifier useful for distinguishing duplicates within that session, not a permanent cross-session or cross-computer ID.
- **before / after / difference:** that object's prices immediately around Full Crew's adjustment. Later damage can make the displayed price lower. Whole-dollar rounding can slightly alter individual percentages.

For a price report, save both the map summary and the particular item's line. Do not compare an item's current damaged price with a remembered maximum price and assume scaling is wrong.

### If detailed prices are missing

1. Confirm the host has Full Crew 0.14.1 or a later version that supports this setting.
2. With the game closed, open `BepInEx/config/local.fullcrew.cfg` in Notepad.
3. Find `[Debug]` and set `Log Loot Values = true`. Edit the existing setting rather than adding duplicate sections/entries. If absent, launch the updated mod once and close it to generate the setting.
4. Save, launch through the correct Gale profile and generate a fresh gameplay level with Full Crew enabled. The host's log should contain `[Loot values]` entries. A guest's log is not the authoritative generation-price record.

This option writes ordinary Info-level messages. You do not need to change BepInEx's global debug-log filtering. It cannot recover original prices from a level generated before logging was enabled. When finished, you can set it back to false to reduce log size.

### If the log itself is missing or looks wrong

- Verify the exact profile used to launch the game and refresh File Explorer. Search within that profile for `LogOutput.log` if necessary.
- If you have not launched that modded profile yet, launch and exit it once. If you already experienced a bug, first preserve any existing logs before trying another launch.
- A stale date, wrong Full Crew version or no Full Crew loading entry can indicate the wrong profile, a failed load or a different launch method. Record what you found rather than guessing.
- Disk logging may have been disabled in a customized BepInEx configuration. Report that possibility; do not reset the whole profile or remove other mods just to find a log.
- If no log is available, submit the report anyway with screenshots, settings and your steps. Mark the log as unavailable.

### What to send with a problem

Attach the copied host log, affected guest log if relevant, mod list, host Full Crew settings and completed [problem report](ISSUE-REPORT.md). Include the relevant excerpt in the report to help the developer find it. Review shared copies for personal paths, player names or other details you do not want to share; note any redactions. Never include passwords or access tokens.

Info means a normal recorded event. Warning means something needs attention but may be expected, such as running out of eligible spawn locations. Error/Exception means an operation reported a problem, but the source and surrounding lines matter. A cart diagnostic warning records a request; it does not by itself prove why the cart was destroyed. A log with no errors does not prove there was no gameplay bug. Testers are responsible for describing observations, not identifying which code caused them.

## Reference settings

Use these values unless a test says otherwise. They are a test setup, not a complete replacement configuration.

```ini
[Scaling]
Enabled = true
BaselinePlayers = 6
MaxTotalItems = 150
Make Large Items Smaller = false
Keep Vanilla Total Map Value = true

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
Active Extractor Font Size = 18
Active Extractor Visibility = Everywhere
Active Extractor Information Position = Middle Left

[Testing]
SimulatedPlayerCount = 0
MinimumExtractionPoints = 0

[Debug]
Log Loot Values = true
```

Log Loot Values normally defaults to false; enable it for these tests. SimulatedPlayerCount only changes Full Crew calculations. It does not create guests or test networking. Never compare a guest test to a simulated solo run as evidence of network compatibility.

### Results and evidence

Use **Pass**, **Fail**, **Blocked** or **Not run**. If a required item, layout or guest is unavailable, mark Blocked rather than Pass. Record constrained spawn capacity separately from a defect. Re-run after changing settings; do not silently alter the setup halfway through a result.

For each case, copy this form into your test notes:

```text
Test ID and date:
Tester / host / guests:
Game build / Full Crew version / other mods:
Map and level:
Settings or profile:
Result: Not run / Pass / Fail / Blocked
Actual result and steps that differed:
Expected and observed numbers:
Log filename and relevant timestamps / screenshot or video:
Bug reference and retest result:
```

### Quick pass

Run UAT-01, UAT-03, UAT-04, UAT-09, UAT-10, UAT-11, UAT-13 and UAT-16 first. Include UAT-17 and UAT-18 before claiming multiplayer acceptance. These checks prioritize the newest logging and confirmation behavior and the reported extraction issues.

## Solo tests

### UAT-01 Installation and configuration

**Setup:** Reference settings; no confirm-button mod.

1. Launch through Gale, load a level and open the log.
2. Confirm Full Crew 0.14.1 loaded once and the expected configuration exists.
3. Open and close the map. Save the log.

**Expected:** No Full Crew startup/patch exceptions; one active installation; configuration remains saved. No persistent extractor overlay outside the map. Other profile mods/files are not removed. Record game-build incompatibility as a failure, not a price-scaling issue.

### UAT-02 Baseline and below baseline

**Setup:** BaselinePlayers=6. Run separate fresh levels with SimulatedPlayerCount=1 and then 6.

1. Read the [Loot values] summary for each level.
2. Compare originals, extras, multiplier, and individual before/after prices.

**Expected:** Multiplier=1; no added valuables; prices unchanged by Full Crew. Original large items may or may not be selected by vanilla generation. Repeatable loads can still be allowed if actual item count warrants them; baseline is not an extractor-disable switch.

### UAT-03 Quantity scaling and cap

**Setup:** BaselinePlayers=3. Generate separate levels with simulated counts 4 and 6. Then test count 6 with MaxTotalItems=20.

1. Record original count N, target T and actual count A from each run's logs.
2. Calculate M=max(1, simulated count / baseline). Expected T=max(N, min(cap, ceil(N*M))).
3. Check the extra-item count equals A-N. Explore for obvious duplicate placements or inaccessible clusters.

**Expected:** Fractional multipliers work; count 6 gives 2x. A never deletes originals or exceeds T. If A<T, inspect the logged spawn-capacity/subtype constraint. Original counts above 20 remain above 20 in the cap case. Extra placement uses available compatible slots; report overlapping/unusable placement with screenshots.

### UAT-04 Preserved value and individual price logging

**Setup:** BaselinePlayers=3, SimulatedPlayerCount=6, cap=150, preservation=true, debug logging=true. Require at least one extra item.

1. Save the [Loot values] summary and all item records from that level.
2. Record original budget B, total before adjustment R and total after adjustment F.
3. Verify F=B. Calculate the retained fraction B/R.
4. Check at least one source=vanilla and one source=extra record. Each after price should be floor(before*B/R) or that value plus $1 due to allocation rounding.
5. If a Golden Swirl or another disputed valuable appears, record its internal name and before/after price. Identify it in-game before handling or damaging it if possible.

**Expected:** Both originals and extras participate. Prices use B/R, not automatically 1/M. Item records identify source, size and instance, and total original+extra records match A. No duplicate dump every frame. The approximate two-decimal log display may introduce pennies of arithmetic uncertainty; allow that display precision when calculating expected whole dollars. Do not assume the Golden Swirl always starts at $12,000. If that item is absent, mark only that subcheck Blocked.

### UAT-05 Increased rewards and debug off

**Setup:** Repeat UAT-04 on a fresh level with preservation=false. Then repeat another level with debug=false.

1. In the first run, compare all before/after item values and totals.
2. In the second run, search only the new session log for [Loot values].

**Expected:** With preservation off, each price remains unchanged by Full Crew and added positive-value items increase the total. Native quotas may change with that total. With debug off, detailed [Loot values] entries disappear; ordinary Full Crew logs remain. Logging itself must not change gameplay.

### UAT-06 Extra item distribution

**Setup:** A 2x or higher multiplier; shrinking=false. Test 50/35/15/0, then 25/25/25/25, then 0/0/0/0 on separate levels.

1. Compare the Extra loot summary with debug entries marked source=extra.
2. Inspect the map for the reported sizes. Keep original items separate in your count.

**Expected:** Large=0 adds no Big/Wide/Tall/VeryTall extras; vanilla large items are allowed. Positive large share permits eligible native large subtypes but does not guarantee them on every map. Mixes approach configured proportions as available slots allow; exact percentages are not guaranteed for small samples. All-zero shares use 50/35/15/0. Save capacity warnings if a type cannot spawn.

### UAT-07 Optional large item shrinking

**Setup:** Baseline=3, simulated=6; run shrinking=false, then true. Obtain an eligible large valuable in each run; otherwise Blocked.

1. Capture screenshots and logged dimensions for comparable large item types.
2. Check originals as well as extra large items when available.
3. Compare handling and damage behavior using the same type and player upgrades; use instrumented mass/durability inspection if available.

**Expected:** Off retains normal scale; on uses 50% of normal dimensions at 2x. Mass and durability must not be deliberately scaled. Different physics impacts are not proof of changed durability. Without controlled evidence, mark exact mass/durability verification Blocked; record qualitative handling separately. Ordinary native percentage-based damage remains in effect.

### UAT-08 Rooms and extractor minimum

**Setup:** Separate runs with ExtraRooms=0/MinimumExtractionPoints=0, then 5/2.

1. Record requested room-module and extractor counts in the logs.
2. Explore the generated layout and verify the logged actual extraction-point count.
3. Check rooms and extraction points are reachable.

**Expected:** Room increase is limited by the existing grid and eligible tiles. MinimumExtractionPoints=2 means at least two requested, not exactly two. Five extra modules can lead to more than two extractors. Ordinary module thresholds correspond to 1 point up to 5 modules, 2 at 6-7, 3 at 8-9, 4 at 10-14 and 5 at 15+. Actual placement must be verified. Special/debug layouts may skip expansion. If the needed two-point layout does not generate, mark dependent tests Blocked and try another level.

### UAT-09 Load allowance and automatic partial extraction

**Setup:** No confirm-button mod; RepeatableLoads=true. Use Baseline=1, simulated=6 and Items Per Load=25. If allowance remains 1, lower Items Per Load to 5 for a new level and record that change.

1. Record actual items A and extractors E. Calculate L=max(1, ceil(A/(E*Items Per Load))). Compare with the logged allowance and panel.
2. Activate a point. Require L>1 and a positive quota Q. Let C be banked credit and R be remaining loads; current threshold is ceil((Q-C)/R).
3. Put eligible loose valuables below that threshold, then add enough to reach it while staying below the full quota.
4. Stand outside the extractor and observe a full partial cycle.

**Expected:** Below threshold does not clear. Reaching threshold starts automatically; approximately 3 seconds of warning precede descent, clearing occurs around 3.7 seconds and reopening finishes around 5.5 seconds. Eligible cleared value is credited once; the load advances once. Partial banking does not complete the point or award final payout/revival. Current Load Quota is the needed loose load total, not the amount still missing after counting objects on the pad.

### UAT-10 Repeatable loads and button matrix

**Setup:** Run all four rows on fresh levels. Install ExtractionPointConfirmButton 1.2.0 and its dependencies only for the requested rows. For on rows, require L>1.

| RepeatableLoads | Button mod | Expected activation and UI |
|---|---|---|
| false | Absent | Native full-quota extraction; no partial loads or Active Extractor section |
| false | 1.2.0 | Button mod's normal full-quota confirmation; no partial loads or Active Extractor section |
| true | Absent | Ready partial loads activate automatically; map information available |
| true | 1.2.0 | Ready partial loads wait for a press; map shows Ready - press confirm |

1. Try a below-full-quota load, then a full-quota load in each setup.
2. For the button/on row, press below the partial threshold, reach the threshold without pressing, then press once and press repeatedly during the cycle.

**Expected:** Below-threshold presses in button/on mode do not clear or cancel the active point. A ready load waits; one valid press starts one cycle. Repeated presses do not duplicate banking. A full-quota load takes the normal final path after confirmation. Test a shop visit too: no Full Crew partial load or extractor map section there; the button mod retains its shop behavior.

### UAT-11 Cancel a warning and replace a partial with final quota

**Setup:** RepeatableLoads=true, L>1. Run without the button, then with 1.2.0.

1. Trigger a partial load and remove enough eligible loot during the initial warning to fall below its threshold. Do this before descent.
2. Check credit and load number. Place the loot again and trigger a new partial.
3. On another attempt, add enough during the warning to meet the full quota before the slam.

**Expected:** Removing loot cancels the partial without destroying removed items, banking credit or consuming a load. Adding enough for full quota cancels the partial path. Without button integration, normal final extraction may follow; with integration, a fresh confirmation is required. No simultaneous partial and final clear or duplicated value.

### UAT-12 Clearing consistency and cart regression

**Setup:** L>1; include a cart and assorted loose sizes. Use two actual points if possible. Repeat in both button modes.

1. Load recognized loose valuables using a cart; keep the load below full quota and qualify for a partial.
2. Record loose count/value before and after the partial, and inspect objects left behind.
3. Repeat at the second point, especially its first partial load. Record at least three partial cycles across attempts.

**Expected:** Eligible collected loose valuables clear and contribute credit once; the cart survives a partial. Boxed/cosmetic objects may remain for native final extraction. An object outside the recognized collection volume is not automatically a clearing failure. Unexpected cart destruction, eligible leftovers or credited-but-present items are failures; save video and the cart stack trace if logged. This reported bug has not been confirmed fixed. Record native final-extraction behavior separately from partial behavior.

### UAT-13 Completion overflow and avoiding a level lock

**Setup:** At least two actual extraction points; L>1. Repeat without and with the button mod.

1. Bank a partial at point one, then supply enough to complete its full quota. Record all values and any refund.
2. Activate the next point and use the refund where applicable. Finish every required point and return to the truck.
3. In a separate run, supply the entire first point quota immediately, with extra value if possible, rather than using each partial allowance. Try placing most available loot there.
4. Track whether surplus is returned and whether the remaining points can be completed. If an exact one-dollar excess can be arranged, record it; otherwise test a measurable excess.

**Expected:** Full quota can finish a point with unused load allowance. Only the completed point advances; a partial alone does not unlock the next point. Credit and native surplus/refund are not lost or duplicated, later points remain completable when sufficient total value exists, and normal truck departure works after all required points. Do not mark an all-loot lock scenario passed if it was not actually exercised; record insufficient loot from damage separately.

### UAT-14 Player collision during a partial

**Setup:** A disposable test run; a qualifying partial and a player without an invulnerability effect. Multiplayer is preferable so another person can observe.

1. Stand clear during one cycle and verify no crush damage outside the volumes.
2. In another cycle, remain inside as the tube closes. Observe warning timing and damage.

**Expected:** No partial crush damage during the first 3 seconds or reopening; eligible players inside active crush volumes receive lethal damage during closure/closed phase. Native invulnerability can prevent this and must be recorded. An absent warning for an unmodded guest is a known limitation, not proof that the cycle did not run. Do not use this test during a valuable normal run.

### UAT-15 Map panel layout and visibility

**Setup:** RepeatableLoads=true; active point. Test initially without Timer Plugin or Minimap, then with the versions recorded in your normal profile.

1. Open the map with its mapped input, close it, and test the game's map-toggle mode if used.
2. Try all six positions; body sizes 9, 18, 49, 50 and 100; both visibility modes. Enter and leave the active room.
3. Check all four value labels and confirmation status when applicable. Finish the point and inspect the next point's data.

**Expected:** Panel appears only while the actual map control is open, never as a look-at-extractor HUD. Room-only mode restricts it to the active room/module; Everywhere may say No active extractor when none is selected. Labels are yellow and values white. Nominal heading size is body*1.10 below 50 and body*1.05 at 50 or above; the full panel can scale to fit the canvas. No clipping or interference with Timer Plugin. Credit/Quota shows banked credit, not unbanked pad value. If another panel overlaps, record both configurations and try another position.

### UAT-16 Fresh level reset and diagnostic limits

**Setup:** Complete or abandon a test level after banking a partial; generate another. Also test Enabled=false in a fresh run.

1. Check the new level's load counts, active selection, timestamps and value summary.
2. Damage an item after its generation record; compare displayed value with its saved log entry.
3. With Enabled=false, confirm Full Crew does not add loot/rooms or partial loads/UI. Other mods may still affect gameplay.

**Expected:** No stale credit, animation or previous-level selection. Debug records describe generation prices only and do not update with damage. Disabled Full Crew does not apply these gameplay changes. After testing, restore SimulatedPlayerCount=0, MinimumExtractionPoints=0 and your normal settings; normally turn Log Loot Values off.

## Real multiplayer tests

### UAT-17 Guests without Full Crew

**Setup:** Host has 0.14.1; at least one guest has no Full Crew. Set SimulatedPlayerCount=0 and baseline low enough for actual players to scale. First test without the button mod, then install button 1.2.0 on everyone for that test.

1. Compare visible item prices, sizes and object presence between host and guest.
2. Perform a partial, a final extraction, the next point and normal departure. Have the guest press the button in the button-enabled run.

**Expected:** Shared loot/value changes and clearing agree; guest button press reaches the host; credit changes once and progression completes. Guest without Full Crew has no added map section or custom partial warning/animation. Host crush damage can still apply: communicate the test timing. Capture this as a known experience limitation requiring release-owner acceptance, not as a fully equivalent guest experience. No missing-custom-RPC errors or duplicate objects caused by Full Crew.

### UAT-18 Guests with Full Crew

**Setup:** Host and guests all use 0.14.1. RepeatableLoads=true, L>1. Test with and without button 1.2.0 installed on everyone.

1. Compare map quota, banked credit, load number and ready status on host and guests.
2. Trigger a partial, cancel during warning, repeat normally and finish. Alternate who presses confirm.
3. Give guests different UI positions/font sizes and different local scaling settings.

**Expected:** Gameplay follows the host's captured settings. Guest UI preferences remain local. Status converges after the normal approximately half-second publishing interval plus network delay; animations align using the host timestamp. No second bank from a guest, stale ready message after clearing, or persistent animation after cancellation. Record latency/video when assessing timing.

### UAT-19 Lobby expander and joining or leaving

**Setup:** Record the exact lobby-expander version and use its required installation arrangement. Actual player count must exceed the baseline; simulated players do not satisfy this test.

1. Generate a level with the expanded group. Verify logged player count and multiplier.
2. Have a player leave mid-level; if the game/expander supports joining a running level, test that separately.
3. Compare existing item prices and load allowance before/after. Generate a subsequent level with the changed group size.

**Expected:** Existing level values/allowance are not recalculated merely because the group changes. The next generated level uses the new actual count. Host authority and object agreement remain intact. Mark unsupported joining or host migration Blocked/out of scope rather than claiming support. A test of one expander does not certify every expander.

## Acceptance and issue reporting

Before accepting a release, review all applicable results. No unresolved issue should silently be treated as acceptable if it causes lost/duplicated money, cart/item loss during partial banking, inability to finish a valid level, incorrect host/guest state, or missing confirmation when required. Record explicit acceptance of known limitations. Optional setups not tested must be listed as Not run or Blocked.

```text
Release candidate:
Test dates and people:
Passed IDs:
Failed IDs and bug references:
Blocked or not run IDs and reasons:
Known limitations accepted by:
Decision: Accept / Accept with listed limitations / Retest required
Decision owner and date:
```

Use [.github/ISSUE_TEMPLATE/bug_report.md](.github/ISSUE_TEMPLATE/bug_report.md) for a failure, even before GitHub is connected. Include the test ID, exact settings, expected vs actual result and saved evidence. Reproduce in the smallest relevant profile before attributing the problem to Full Crew. Player configuration details are in [PLAYER-GUIDE.md](PLAYER-GUIDE.md). Development source and build instructions are maintained separately in a private repository.


