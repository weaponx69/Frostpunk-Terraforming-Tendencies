# FrostKit Editor Checklist — Stage Spine

Do this in **FrostKit / Frostpunk 2 Mod Editor** (Windows). Binary `.uasset` Blueprints cannot be wired from the Linux CLI.

Content path: `Content/Scenarios/Terraforming-Tendencies`  
Design numbers: [`FROSTPUNK_ADAPTATION.md`](FROSTPUNK_ADAPTATION.md)

## 0. Open project

1. Launch `Frostpunk2ModEditor` from the FrostKit install.
2. Content Drawer → **Scenarios → Terraforming-Tendencies**.
3. Duplicate sample assets before heavy edits if you want a rollback (or rely on git).

## 1. Rebrand sample hub (copy)

| Asset | Change |
|-------|--------|
| `Configs/Messages/Choristers/DA_FrostkitSample_Intro_Chorister` | Intro: corporation charter, tax + terraforming quota, weeks |
| `Configs/Messages/Dialogs/DA_FrostkitSample_ProvideCoal` | Retitle → Round 1 briefing (tax 80 Materials, quota 20%, ~50 weeks) |
| `Configs/Messages/Dialogs/DA_FrostkitSample_BasicsMessage` | Mid-round tip: stockpile tax separately; quota from heat/food/outposts |
| `Configs/Messages/Dialogs/DA_FrostkitSample_EndMessage` | Win: charter cleared / habitability proven |
| Scenario display names on `DA_FroskitSample_Scenario` / Startup | “Terraforming Tendencies” in menus |

Optional: rename assets from `FrostkitSample_*` → `TT_*` only if you update all references (FlowHub, ScriptPacks, Startup). Safer MVP: keep names, change **player-facing text only**.

## 2. VariableStorage

Open `BlueprintsSerialized/VariableStorage/BP_FrostkitSample_VariableStorage`.

Add ints (or equivalent):

- `RoundIndex` (default 1)
- `TaxDue` (default 80)
- `TerraformMeter` (0)
- `RoundQuotaTarget` (20)
- `RoundQuotaBanked` (0)
- `GraceWeeksLeft` (1)
- `WeekDeadline` (set when round starts)

## 3. Unlocks

Create under `Configs/Unlocks/` (or duplicate `DA_FrostkitSampleGameStart`):

- `DA_TT_Round1Complete`
- `DA_TT_Round2Complete`
- `DA_TT_Round3Complete` / win
- `DA_TT_Draft_Solar` / `Habitat` / `Shipment_R1` (and Round 2 draft unlocks)
- Keep using Shared `DA_FrostlandAvailableContext` / logistics unlock as needed

## 4. Quests (replace Coal / Basics role)

Duplicate `BlueprintsSerialized/Quests/BP_FrostkitSample_Coal` → `BP_TT_Round1`.

**Round 1 objectives (ImportantMainQuest or equivalent):**

1. Stockpile Materials ≥ 80 (statistic / resource objective).
2. Terraform progress ≥ 20 — if no native objective type, drive via FlowHub polling VariableStorage and complete quest when meter hits target.
3. Soft tutorial: Housing district + Extraction on coal (keep sample basics so the city boots).

Duplicate for `BP_TT_Round2` (Materials ≥ 160, meter ≥ 50) and `BP_TT_Round3` (Materials ≥ 280, meter ≥ 100).

## 5. FlowHub spine

Open `BlueprintsSerialized/BP_FrostkitSample_FlowHub`.

Replace Coal → Basics sequence with:

1. Wait loading / cutscene (keep).
2. Intro chorister + Round 1 dialog.
3. Set vars: `TaxDue=80`, `RoundQuotaTarget=20`, `WeekDeadline=now+50`.
4. `StartQuest` Round1 → `WaitForQuest`.
5. Open **Pay Corporation** dialog → on Pay: consume Materials, set unlock Round1Complete.
6. Open **Draft** dilemma (3 choices) → `SetUnlockActive` for picked card.
7. Activate frostland / territory unlock if not already.
8. Repeat for Round 2 (tax 160, quota +30 → meter 50, +60 weeks) and Round 3 (tax 280, meter 100).
9. End message.

**Deadline / grace:** `WaitForDuration` or week statistic watch. If deadline hits and quest incomplete: if `GraceWeeksLeft>0`, decrement and extend deadline +10; else fail / end.

**Quota ticking:** periodic check (or story-ark need/statistic): when heat/food/outpost conditions met, add points to `RoundQuotaBanked` / `TerraformMeter` per GDD table.

## 6. Pay + draft dilemmas

- Duplicate `Configs/Dilemmas/DA_FrostkitSample_Generic_StockpileMaterials5` for **Pay Corporation** (consume Materials = TaxDue; only enable when stockpile ≥ TaxDue and quota met — or gate in FlowHub before opening).
- New dilemmas: `DA_TT_Draft_Round1`, `DA_TT_Draft_Round2` with 3 choices each (see GDD).

## 7. Sector / frostland gates

- Keep early Logistics → frostland for survival (sample `BP_FrostkitSample_UnlockFrostland`).
- Gate **extra** tiles / sites / second ring behind `DA_TT_Round1Complete` / `DA_TT_Round2Complete` via scenario scripts (duplicate UnlockFrostland pattern).

## 8. Playtest tune

| Knob | If too easy | If too hard |
|------|-------------|-------------|
| Week budget | −10 | +10 |
| TaxDue | +20 | −20 |
| Quota target | +5 | −5 |
| Grace | 0 | 2 |

Success: player always sees deadline / tax / quota; payment feels like Combolands tax; draft matters between rounds.

## 9. Commit

After editor saves, from this repo:

```bash
git status
git add -A
git commit -m "Wire Combolands-style tax and terraforming quota stage spine."
```
