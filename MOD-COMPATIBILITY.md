# Full Crew testing with other mods

This checklist tests Full Crew 0.14.1 alongside other mods. None of the combinations below is certified as compatible merely because it is listed. Record exact versions and results with [TEST-REPORT.md](TEST-REPORT.md); report problems with [ISSUE-REPORT.md](ISSUE-REPORT.md).

## Start small then use your normal mod setup

Create separate Gale profiles so your regular setup stays available. Keep required dependencies for every mod you enable. “No other mods” still includes BepInEx and the components Full Crew needs. Do not remove loader files manually.

1. **Reference profile:** Full Crew plus its required loader components. Run the quick pass in [TESTING.md](TESTING.md).
2. **Pair profile:** Full Crew plus one other mod and its dependencies. Run the checks for that mod below. Keep Full Crew settings the same as the reference profile unless the test says to change them.
3. **Normal group profile:** Your group's usual collection of mods. Play a complete level with partial and final extraction. Record all versions and installation differences between host and guests.
4. **Only if a problem occurs:** Try the relevant pair profile. If practical, try the other mod without Full Crew. Report all outcomes, including failures that do not repeat.

Use fresh levels after changing gameplay settings. Random maps differ: compare each run with its own logged original values/counts. Do not assume identical generation across profiles. Simulated players help test scaling but cannot prove guest compatibility.

## Combinations to check

| Setup | What to try | What should happen |
|---|---|---|
| Full Crew alone | UAT-01 through relevant solo tests | Establish behavior without optional-mod interactions |
| ExtractionPointConfirmButton 1.2.0 | UAT-10 and UAT-11 with RepeatableLoads both on and off; then guest presses in UAT-17/18 | On: ready partial waits for press; off: normal button behavior; one press never banks twice |
| Timer Plugin | UAT-15; try default and alternate panel positions | Both panels remain readable; closing map hides Full Crew information; neither mod needs the other to start |
| Minimap | UAT-15 with minimap visible while full map is closed | Full Crew panel stays hidden until actual map input opens the map; test held and toggled map modes |
| REPOConfig | Change a UI setting and a generation setting; reload for generation changes | Correct labels/settings are saved; UI changes display correctly; next generated level uses new generation settings |
| Lobby expander | UAT-19 with real players above baseline, following the expander's installation requirements | Correct actual count, stable current-level values, synchronized items and normal completion |
| NoItemSpawnLimit or other spawn mods | UAT-03, UAT-04 and UAT-06; record load order/version | Count and value logs are coherent; originals are not deleted by Full Crew's cap; no repeated spawning, duplicates or generation hang |
| MapValueTracker or other value displays | UAT-04/05 and partial/final banking; compare at the same moment | Initial totals agree when tools count the same objects; banked objects do not remain shown as collectible value indefinitely |
| MoreShopItems or other shop mods | Visit shop after successful extraction and buy an item normally | No Full Crew partial cycle or map extractor section in shop; purchase and next-level generation work |
| Other extraction or money mods | UAT-09 through UAT-13 | No double activation, lost credit, duplicated refund or blocked progression |
| Other physics, item-size or damage mods | UAT-07, UAT-12 and UAT-14 | No unexpected cart loss, repeated scaling, desynchronization or unintended damage beyond the combination's documented behavior |
| Normal group collection | Full level, two extraction points if available, partial/final cycles, shop and next level | No new failures; record every mod rather than assuming pair tests cover the whole collection |

The button integration explicitly supports 1.2.0. For another button version, current Full Crew should log a warning and disable its partial loads, leaving normal button-mod final behavior. Record this as an unsupported-version fallback, not successful partial-load compatibility. Do not install arbitrary old/new versions just to exercise this branch.

A value tracker may count different categories, such as enemy drops or surplus bags. Compare what each display measures before filing a mismatch. Full Crew's detailed value log captures generation time, not later damage or another mod's later edits. Another mod that intentionally changes prices or physics can alter the combined result; record the combination and expected behavior rather than blaming either mod without evidence.

## Small multiplayer matrix

For the reference, button and normal group profiles, test these installation arrangements where their own dependencies allow them:

| Arrangement | Checks |
|---|---|
| Host has Full Crew; guest does not | Guest sees shared items/prices/clearing and can finish; no Full Crew map panel or custom partial warning is expected on guest |
| Everyone has Full Crew 0.14.1 | Map data agrees after network delay; warning/partial cycle is visible; guest pressing confirm triggers one host-controlled action |

The button mod must be installed on everyone when used. Other mods have their own requirements. A host-only Full Crew install does not make every optional mod host-only. Guests without Full Crew currently lack the partial warning animation even though crush damage applies; record acceptance or concern about this limitation explicitly.

## Practical session checklist

- [ ] Copy the tester report and record the profile/mod versions.
- [ ] Start a fresh level and check players, loot and prices.
- [ ] Check the map panel while the map is open and closed.
- [ ] Try a partial load below full quota and watch the cart/items.
- [ ] Finish the point, use another point if available, and leave normally.
- [ ] Visit the shop and generate the next level.
- [ ] Save host/guest logs before relaunching.
- [ ] Report what was actually tried; leave unavailable checks as Could not test.

## What a completed check looks like

The following is a fictional example, not a recorded test result:

> Check: UAT-10, repeatable loads on, button 1.2.0. We put $3,200 on a pad whose current load quota was $3,000 and full quota was $9,000. It waited until the guest pressed the button, gave the warning, then cleared once. The host panel showed $3,200 credited. Result: Worked for this check. Evidence: sessionA-host.log and clip01.mp4.

If instead it starts before the press, report that observation, the settings and evidence. You do not need to identify the failed function or propose a code fix.
