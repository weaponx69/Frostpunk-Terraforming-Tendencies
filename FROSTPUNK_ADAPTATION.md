# Terraforming Tendencies — Frostpunk Adaptation GDD

Combolands-style **stage loop** as the scenario spine. Unity RTS systems in `PROJECT_KNOWLEDGE.md` are design archaeology only. **Foundry Crawler / EnergyPipeline are deprecated for good.**

## Fantasy

A corporate terraforming charter on the Frostland. You run a city under a corporation contract: each round you must **pay tax** and **hit a terraforming quota %** before the week deadline, or the charter is revoked.

Systems stay Frostpunk 2 (Heat, Temperature, Materials, Food/Soil, Research, Outposts, Frostland). Copy talks tax / quota / habitability.

## Spine (every round)

1. Player receives **N weeks** (Frostpunk calendar).
2. Within that window they must:
   - Stockpile **corporate tax** (Materials).
   - Generate / bank the **terraforming quota** for this round (% of run-long habitability meter).
3. On success: **pay corporation** (consume tax stockpile) → **between-round draft** → unlock next sector/frostland ring → next round.
4. On failure: spend limited **grace weeks** once, or **fail the run**.

Weeks are a **deadline clock**, not a placement cost. One district build does not consume one week.

```
StartRound → Play(week budget) → Tax+Quota met?
  yes → PayCorporation → Draft → UnlockSector → next round
  no  → GraceOrFail
```

## VariableStorage counters

Track in scenario `VariableStorage` (or Constants + unlocks if vars are awkward):

| Key | Meaning |
|-----|---------|
| `RoundIndex` | 1, 2, or 3 |
| `TaxDue` | Materials required this round |
| `TaxPaid` | 0/1 after consume |
| `TerraformMeter` | Run-long progress 0–100 |
| `RoundQuotaTarget` | Points required this round |
| `RoundQuotaBanked` | Points earned this round |
| `GraceWeeksLeft` | Starts at 1 (or 2); decrements on overtime |
| `WeekDeadline` | Absolute week number when round ends |

## Round numbers (vertical slice — tune in playtest)

| Round | Week budget | Tax (Materials) | Quota (% of 100) | Cumulative habitability |
|-------|-------------|-----------------|------------------|-------------------------|
| 1 — Charter dues | 50 weeks | 80 | 20 | 0 → 20 |
| 2 — Expansion dues | 60 weeks | 160 | 30 | 20 → 50 |
| 3 — Habitability proof | 70 weeks | 280 | 50 | 50 → 100 |

**Grace:** +10 weeks once per run if tax+quota not met at deadline (Combolands overtime). Second miss = fail.

### How quota fills (sources)

Award `RoundQuotaBanked` / `TerraformMeter` from scripted checks (pick what FlowHub can observe reliably):

| Source | Suggested points |
|--------|------------------|
| Generator / Heat capacity sustained above tier | +4 / check |
| City temperature band improved (warmer Heat level) | +6 once per tier |
| Food surplus maintained (nourishment OK) | +3 / check |
| Soil / Food frostland outpost established | +8 once |
| Greenery / “atmosphere” building unlock active + built | +5 once |
| Materials stockpile above tax (surplus engine) | +2 / check |

Cap per-round banking at `RoundQuotaTarget`. Overflow can spill into meter flavor only; do not skip next round’s quota.

### How tax works

1. Quest objective: **Stockpile Materials ≥ TaxDue** (do not auto-consume while playing).
2. When both tax stockpile and quota are met (or at deadline with both met): open **Pay Corporation** dialog.
3. Choice **Pay** → remove `TaxDue` Materials from stockpile → set `TaxPaid=1` → complete round quest → draft.

Use / adapt sample dilemma pattern: `Configs/Dilemmas/DA_FrostkitSample_Generic_StockpileMaterials5.uasset`.

## Between-round draft (MVP)

After payment, present **3 choices** (dilemma or dialog). Pick 1. Persist via Unlocks.

### Round 1 draft pool

| Card | Effect |
|------|--------|
| Solar Array Project | Unlock / emphasize heat-adjacent efficient heating building path |
| Modular Habitat Dome | Housing / population path unlock |
| Heavy Alloys Shipment | +40 Materials immediate |

### Round 2 draft pool

| Card | Effect |
|------|--------|
| Atmosphere Processor | Survival / heat-adjacent “atmosphere” building path |
| High-Power Induction Drills | Extraction / Materials gather buff (economy modifier if available) |
| Bio-Dome Culture Serum | +Food or Soil outpost favor |

### Round 3

No draft after win — end message. Optional mid-round draft if pacing needs it.

Full 29-card climate-gate deck is **deferred**.

## Sector / frostland unlocks

Extend sample pattern (`BP_FrostkitSample_UnlockFrostland`: Logistics built → unlock → frostland context):

| After round | Unlock |
|-------------|--------|
| Round 1 paid | First frostland ring / exploration context (if not already open via Logistics) |
| Round 2 paid | Second territory slice / additional outpost sites |
| Round 3 paid | Win — charter cleared |

Do **not** gate Round 1 so hard that the player cannot earn Materials or quota. Logistics tutorial can still run early; **tax payment** gates *expansion*, not survival basics.

## Quest / FlowHub outline

Replace sample Coal → Basics chain with:

1. **Intro chorister / dialog** — corporation charter, explain tax + quota + weeks.
2. **Quest Round1_TaxAndQuota** — stockpile Materials ≥ 80; terraform meter ≥ 20; optional soft tutorials (housing, extraction, heat).
3. **WaitForQuest** → Pay dialog → Draft dilemma → SetUnlocks → bump RoundIndex.
4. **Quest Round2_…** (tax 160, meter ≥ 50, week deadline).
5. Pay → Draft → unlocks.
6. **Quest Round3_…** (tax 280, meter ≥ 100).
7. Pay → **End message** (charter complete).

Fail path: at deadline if unmet → grace dialog once → else fail/end.

Wire from `BlueprintsSerialized/BP_FrostkitSample_FlowHub.uasset` (rebrand to Terraforming Tendencies FlowHub when renaming assets in editor).

## Explicitly out

- Foundry Crawler / EnergyPipeline (deprecated for good)
- RTS drones, combat units, Unity power graph
- Week-as-placement currency (1 build = 1 week)
- Full climate-gate card deck, guild pairings, combo score engine
- 20-level per-unit tech trees

## Success criteria

- Player always knows: week deadline W, tax T, quota Q this round.
- Paying advances the run; draft permanently changes options.
- Missing both tax and quota after grace fails the run.
- Playable 3-round vertical slice on the existing sample map.
