# PES6 Master League Editor v1.0 — Technical Documentation & Extension Guide

> **Project:** PES6 Master League Editor  
> **Version documented:** v1.0 Stable  
> **Author:** jackcohle  
> **Target:** Pro Evolution Soccer 6 (PC, 32-bit) + Cheat Engine  
> **Purpose of this document:** explain the internal architecture of the stable table so other Cheat Engine developers can study it, fork it, or add features without breaking the existing safety model.

> **Reverse-engineering note:** Function and structure names used in this document are descriptive names assigned by this project during reverse engineering; they are **not official symbols from the PES6 executable**. Fixed addresses, module offsets, byte signatures and inferred structure roles are documented for the **tested builds only** and may differ in other patches. Where the document describes observed behavior, it refers to behavior verified for the stable v1.0 implementation rather than claiming an official internal specification from Konami.

---

## 1. Scope

This document is for developers who want to:

- add new Player Editor fields or actions;
- add new player profiles;
- add new squad-wide presets;
- add new Master League or match controls;
- reuse the roster/player resolver in another CT;
- understand the Score Controls and native Game History handling;
- port individual systems to another PES6 patch;
- debug compatibility problems without replacing the whole architecture.

This is **not** a list of user-facing features. For normal usage, see `README.md`.

The technical details below describe the final stable v1.0 CT as shipped. Internal symbols, offsets, and helper functions should be treated as **implementation details**, not a permanent API.

---

## 2. Tested Builds

The stable v1.0 table was validated against these two executables:

| Environment | Executable | Size | SHA-256 |
|---|---|---:|---|
| Standard / unpatched PES6 | `PES6.exe` | 21,880,832 bytes | `53263fb86b10b1bd2a9a962816c55ba23954e8f0596da80e8adebb4fead3295e` |
| PES 6 Original Season patch | `pes6.exe` | 21,880,832 bytes | `cd30427917be6a903ea4624147ca8506c7db5462a4a4e9f50ee8dd6c9d494628` |

Stable CT SHA-256:

`ce9c35802e9a5aca2a29e2e273aa889eca41fcd30d75230796562fb3bfc44b1d`

The executables are not byte-identical. Any new hook, call into an identified PES6 routine, or fixed module offset should therefore be revalidated on **both** tested builds before it is considered compatible.

---

## 3. Design Goals

The stable table follows several rules that are more important than any individual address:

1. **Use one verified Master League roster map.** Player Selector, Player Editor, Squad Ability Presets and Squad Fitness Overview must not independently guess different player lists.
2. **Do not trust Player ID alone.** A matching ID is not sufficient proof that a memory record is the intended live Master League player.
3. **Prefer verified/cached targets over repeated process-wide scans.**
4. **Do not continuously write when an event-driven or change-driven write is enough.**
5. **Suspend live writes during unsafe post-match / cutscene transitions.**
6. **Treat match history as a native variable-length structure, not as a fixed-size array.**
7. **Snapshot and verify complex transactions before committing them.**
8. **A failed verification should abort safely rather than guess.**
9. **All temporary state must be cleared when `[ACTIVATE]` is disabled.**

These rules are the reason some parts of the CT look more defensive than a simple static-address cheat table.

---

## 4. High-Level Architecture

The table has four main layers.

### 4.1 Auto Assembler hook layer

The main `[ACTIVATE]` record installs the core hooks and allocates the editor's shared memory.

Important hook families include:

- current-player resolver capture;
- Master League name/root capture;
- Master League ability synchronization capture;
- optional Infinite Stamina hook.

Hooks are found with `aobscanregion` / `aobscanmodule` and protected with `assert`.

The main resolver hook is intentionally not the whole editor. Its job is to expose useful runtime identity/pointer information to the higher-level Lua logic.

### 4.2 Shared symbol / mirror layer

The CT allocates and registers symbols such as:

- `ml3_team_ids`
- `ml3_team_base_record_ptrs`
- `ml3_team_current_ability_ptrs`
- `ml3_selected_id`
- `ml3_selected_ptr`
- `ml3_contract_record_ptr`
- `ml3_ability_values`
- `ml3_position_flags`
- `ml3_special_flags`
- `pes6_ml_match_time_selector`
- `pes6_ml_home_score_status`
- `pes6_ml_away_score_status`

Most Player Editor rows point to these editor-owned symbols rather than directly to a volatile in-game player pointer.

This separation is important: the UI edits a stable editor buffer, and the runtime decides when and where it is safe to write the result back.

### 4.3 Lua orchestration layer

The main activation record defines the resolver, caching, roster, preset, recovery, score/clock and safety logic.

The primary runtime timer runs every **60 ms**.

The runtime is responsible for:

- maintaining display-only score mirrors;
- detecting post-match unsafe states;
- maintaining the custom match-time controller;
- preparing/recovering the verified Master League roster;
- refreshing Player Selector / Overview state;
- resolving the selected player;
- tracking fitness/contract records;
- applying write-back only when the Player Editor buffer changes;
- maintaining squad preset persistence without writing continuously.

### 4.4 One-shot action layer

Most visible `[ACTION]`, `[PROFILE]` and `[SELECT]` records are intentionally very small wrappers.

They call a shared Lua function and then automatically deactivate themselves.

Examples:

- `PES6MLApplyEditorAction(...)`
- `PES6MLApplyPlayerPreset(...)`
- `PES6MLSetSingleTeamSlot(...)`
- `PES6MLApplySelectedRecovery(...)`
- `PES6MLApplyTeamRecovery(...)`
- `PES6MLRequestTeamPreset(...)`

When extending the CT, prefer adding logic to the shared implementation instead of duplicating a large copy of the same code into multiple visible records.

---

## 5. Activation and Shutdown Lifecycle

### 5.1 Enable

When `[ACTIVATE]` is enabled, the table:

1. scans for required code signatures;
2. validates expected bytes with `assert`;
3. allocates shared buffers and pointer arrays;
4. registers symbols used by visible memory records;
5. installs the resolver/name/ability hooks;
6. defines the Lua helper functions and caches;
7. resets old Match Clock runtime state;
8. caches stable score/status addresses used by the 60 ms loop;
9. starts the main runtime timer;
10. builds/refreshes the initial Overview / Player Selector state.

### 5.2 Disable

Disabling `[ACTIVATE]` is a real session boundary.

The shutdown path:

- releases Raw clock freeze state;
- restores an active squad preset where appropriate;
- destroys runtime timers;
- removes dynamic Overview records;
- clears player/preset/name/scan caches;
- clears old resolver pointers;
- clears post-match state;
- unregisters symbols;
- restores original hooked bytes;
- deallocates allocated memory.

This is why the public instructions require the user to manually untick and re-tick `[ACTIVATE]` after leaving Master League completely.

### Extension rule

If you add any global table, timer, dynamically created record, cache, allocated buffer, or registered symbol, add its cleanup to the `[DISABLE]` path as part of the same change.

---

## 6. The Shared 32-Slot Master League Roster

The editor uses a maximum 32-slot squad map.

Core symbol:

`ml3_team_ids`

This is an array of 32 WORD Player IDs.

Associated pointer arrays:

- `ml3_team_base_record_ptrs`
- `ml3_team_current_ability_ptrs`
- `ml3_team_source_ability_ptrs`

The table does not assume that every slot contains a player. Empty/invalid IDs are ignored.

### 6.1 Roster preparation

`PES6MLPrepareTeamFromFitness()` builds the stable roster from the same data used by Squad Fitness Overview.

A complete roster is accepted only when enough valid Master League players are detected. The current code requires at least 11 players before treating the map as a usable team.

If the roster signature changes, caches tied to the previous team are cleared.

### 6.2 Why the roster is "frozen"

Once a valid team has been prepared, Player Selector and squad-wide actions operate against that same slot map.

This prevents the resolver from silently changing the meaning of "player 5" while the user is editing or while a squad preset is active.

### 6.3 Selecting one player

Player Selector calls:

`PES6MLSetSingleTeamSlot(slot, true)`

which resolves through:

`PES6MLLockTeamSlot(slot)`

Selection uses:

- roster slot;
- Player ID;
- captured base/current pointers;
- name validation;
- recovery logic used by the squad-preset target map.

The selected player remains locked until another slot is selected.

### 6.4 Duplicate IDs and unusual/transferred players

Do **not** resolve a player by ID only.

The CT contains additional name/root/pointer checks because different records can share an ID or because PES6 can surface temporary/live copies that are not the intended full player record.

For unusual players, Player Editor can operate in a **shadow mode**:

- a local 0x7C player-shaped buffer is created;
- the recovered 72-byte live block is copied into `shadow + 0x34`;
- the Player Editor reads/writes the shadow;
- verified write-back synchronizes changes to the actual live copy.

When adding player features, make sure they work in both:

1. normal full-record mode;
2. recovered/shadow live-block mode.

---

## 7. Player Record Model

### 7.1 Record sizes

The code treats a normal player record as:

- full record size / stride: `0x7C`
- current editable ability block start: `+0x34`
- current ability block size used by the editor: **72 bytes**

Many squad/preset routines therefore work on a 72-byte block with `origin = 0x34`.

### 7.2 26 core ability values

The 26 abilities are exposed through `ml3_ability_values[0..25]`.

Offsets relative to the full player record:

| Ability index | Offset |
|---:|---:|
| 0–19 | `0x36` through `0x49` |
| 20 | `0x4A` |
| 21 | `0x4C` |
| 22 | `0x4D` |
| 23 | `0x4E` |
| 24 | `0x4F` |
| 25 | `0x4B` |

The lower 7 bits hold the ability value.

Do not blindly overwrite the whole byte: some of these bytes also carry position/special flags in bit 7.

### 7.3 Playable positions

There are 12 `ml3_position_flags` values.

The position flags are stored in **bit 7** of full-record offsets:

`0x36` through `0x41`

The lower 7 bits are ability data, so any new code touching these bytes must preserve the other bits.

### 7.4 Special Abilities

There are 23 `ml3_special_flags`.

The current v1.0 mapping uses these full-record offsets/masks:

| Group | Storage |
|---|---|
| First 14 specials | bit `0x80` at offsets `0x44,0x45,0x46,0x47,0x48,0x49,0x4A,0x4B,0x4C,0x4D,0x4E,0x4F,0x43,0x42` |
| Next 8 specials | masks `01,02,04,08,10,20,40,80` at offset `0x52` |
| Long Throw | bit `0x80` at offset `0x54` |

Use read-modify-write semantics when altering these fields.

### 7.5 Packed performance/style fields

The Player Editor decodes several packed bytes.

#### Full record `+0x34`

- bit 0: Preferred Foot
- bits 1–4: Free Kick Style
- bits 5–7: Penalty Kick Style minus 1

#### Full record `+0x35`

- bits 0–1: Dribbling Style minus 1
- bits 2–3: Drop Kick Style minus 1
- bits 4–7: Registered Position

#### Full record `+0x50`

- bits 0–2: Consistency minus 1
- bits 3–5: Weak Foot Frequency minus 1
- bits 6–7: Injury Tolerance

#### Full record `+0x51`

- bits 0–2: Condition minus 1
- bits 3–5: Weak Foot Accuracy minus 1
- bits 6–7: Favoured Side

### 7.6 Identity / physical encoding

#### Full record `+0x58`

- low 6 bits: encoded Height
- displayed height = low6 + `0x94` (148)
- high 2 bits: Skin Colour

#### Full record `+0x59`

- low 7 bits: Weight raw value

#### Full record `+0x70`

- low 7 bits: Nationality

#### Full record `+0x71`

- bits 1–5: encoded age
- displayed age = encoded value + 15

These encodings are already handled by the stable Player Editor code. Reuse the existing bit helpers rather than writing full bytes.

---

## 8. Player Editor Buffer and Write-Back

Visible Player Editor fields normally edit the CT-owned symbols, not a volatile game record directly.

The runtime builds a signature of the current editor state.

When `Instant Live Write-Back` is enabled:

1. the 60 ms loop compares the current editor signature with the last signature;
2. if the signature changed, `PES6ML40WriteCurrent()` runs;
3. the selected verified target is updated;
4. known stale/live copies are reconciled when required;
5. the live-write counter is updated.

### Extension rule for a new Player Editor value

If a new value belongs to the editable player block:

1. allocate/register an editor symbol;
2. decode it when loading a player;
3. include it in the editor pack/signature;
4. encode it during write-back;
5. clamp/validate its legal range;
6. preserve unrelated bits in the same byte;
7. verify shadow-mode behavior;
8. add a visible memory record that points to the editor symbol.

Do not create a visible row that directly follows `ml3_selected_ptr` unless there is a strong reason. The buffer/write-back architecture is intentionally safer than raw pointer editing.

---

## 9. Contracts, Fitness and Shirt Numbers

These values are resolved through:

`ml3_contract_record_ptr`

Current offsets relative to the resolved contract/fitness record:

| Value | Offset / bits |
|---|---|
| Shirt Number | `+0x03`, Byte |
| Match Condition | `+0x0C`, bits 0–2 |
| Pre-Match Stamina | `+0x0D`, Byte |
| Accumulated Fatigue | `+0x0E`, Byte |
| Contract Years Remaining | `+0x11`, bits 2–7 |
| Yearly Salary | `+0x12`, 2 Bytes |

These values are separate from the temporary 72-byte live ability/preset block.

Some of them can persist when the user saves Master League in-game, so new fields in this area should be documented as potentially permanent.

---

## 10. Quick Actions and Player Profiles

### 10.1 Quick actions

Visible actions call:

`PES6MLApplyEditorAction(kind)`

Current action keys include the existing 99 abilities / all specials / max performance / superstar logic.

A new quick action should modify the **Player Editor buffer**, then let the existing write-back path synchronize the selected player.

### 10.2 Player profiles

Profiles call:

`PES6MLApplyPlayerPreset(key)`

The stable design keeps profile definitions centralized rather than embedding full write logic in every visible memory record.

### Recommended pattern

Visible record:

```text
[ENABLE]
{$lua}
if syntaxcheck then return end
if not PES6MLApplyPlayerPreset then
  error('Activate the main editor first.')
end

PES6MLApplyPlayerPreset('my_new_profile')

local r=memrec
local t=createTimer(nil,false)
t.Interval=100
t.OnTimer=function(x)
  x.destroy()
  if r and r.Active then r.Active=false end
end
{$asm}

[DISABLE]
```

Then add the real profile definition to the shared profile table / shared preset function.

---

## 11. Squad Recovery

Squad Recovery reuses the same verified roster map.

Public entry points:

- `PES6MLApplySelectedRecovery(mode)`
- `PES6MLApplyTeamRecovery(mode)`

Do not write a second independent "find my squad" implementation for a new recovery feature. Resolve the player/team through the existing roster architecture.

---

## 12. Squad Ability Presets

### 12.1 Public entry point

Visible actions call:

`PES6MLRequestTeamPreset(mode)`

which uses:

`PES6MLSetTeamPreset(mode)`

### 12.2 Current preset modes

| Mode | Meaning |
|---:|---|
| 0 | Restore original squad ability values |
| 1 | Complete Squad Boost |
| 2 | Set all 26 abilities to 99 |
| 3 | Enable all Special Abilities |
| 4 | Max Performance Settings |
| 5 | +5 abilities |
| 6 | +10 abilities |
| 7 | +15 abilities |
| 8 | +20 abilities |
| 9 | Ultimate Squad by Position |

The byte mutation is centralized in:

`PES6MLApplyModeToBytes(bytes, origin, mode)`

### 12.3 Backup-before-write model

Before a preset writes to a target, the original 72-byte target is backed up.

If target resolution or writing fails, the preset path attempts to restore the original state.

This is important for both:

- user-facing Restore Original;
- transactional safety during partial failures.

### 12.4 Session persistence

The preset is session-based, but PES6 can rebuild player ability copies after a match.

The CT does **not** constantly rewrite all players every 60 ms.

Instead:

- a newly surfaced verified resolver record may receive the active overlay once;
- a lightweight mass-reload detector compares known preset targets;
- a preset is repaired only if a meaningful part of the squad changed together.

Current mass-reload threshold:

- at least 6 valid checked targets;
- repair only when at least `max(6, ceil(checked * 0.25))` targets mismatch.

A short cooldown prevents repeated write bursts during menu transitions.

### 12.5 Adding a new squad preset

Recommended procedure:

1. choose a new unused mode number;
2. add its transformation to `PES6MLApplyModeToBytes`;
3. if role-dependent, derive from Registered Position using the existing role logic;
4. add a visible one-shot action calling `PES6MLRequestTeamPreset(newMode)`;
5. update Active Squad Preset dropdown text;
6. verify Restore Original;
7. verify persistence after a normal match;
8. verify a championship/celebration transition;
9. verify individual Player Editor overrides still layer correctly above the team preset.

---

## 13. Post-Match / Celebration Safety

This is one of the most important systems to preserve.

`PES6MLUpdatePostMatchSafety()` watches the raw match clock and resolver lifecycle.

### 13.1 Arm condition

The guard does not arm merely because Raw time is 0 at a menu.

It first requires a genuinely running match to have been observed repeatedly.

When a running period ends and the raw time reaches 0, the table enters a protected state.

### 13.2 Protected state

During the protected window:

- Player Editor live writes stop;
- roster preparation does not write;
- squad preset resolver-follow writes stop;
- stale resolver/scan caches are discarded;
- the code waits for the old resolver generation to unload.

### 13.3 Second-half handling

Half-time also sets Raw time to 0.

If the same match resumes and Raw becomes positive again, the guard is released without treating that as a complete Master League teardown.

### 13.4 Resume after a real post-match teardown

The table waits for:

- resolver unload evidence;
- a short delay;
- a newly valid Master League roster.

Only then are new pointers accepted and an active squad preset rebuilt on the new verified targets.

### Extension rule

Any new feature that performs live player/team writes during a match should check the existing post-match safety state instead of inventing an independent full-time detector.

---

## 14. Score Controls Architecture

The final v1.0 Score Controls are deliberately conservative.

### 14.1 Display-only score rows

`[STATUS] Home Score` and `[STATUS] Away Score` do **not** point directly to the game score.

They point to:

- `pes6_ml_home_score_status`
- `pes6_ml_away_score_status`

The 60 ms timer mirrors the real score into these buffers only when the value differs.

Therefore a user can type another number into Cheat Engine, but the real match score is not changed; the mirror returns to the real value.

### 14.2 Tested-build implementation addresses

The final implementation uses the following addresses on the two tested builds. The labels are **project-assigned descriptive names**, and the values below should be treated as implementation references for those builds rather than a universal PES6 memory map.

| Project label / purpose | Tested-build address |
|---|---|
| Raw remaining match time | `pes6.exe+37E098C` |
| Home gameplay score | `pes6.exe+C17B30` |
| Home display/result score | `pes6.exe+C17B38` |
| Away gameplay score | `pes6.exe+C17E24` |
| Away display/result score | `pes6.exe+C17E2C` |
| Home goal counter base | `pes6.exe+C17B5C` |
| Home assist counter base | `pes6.exe+C17B60` |
| Away goal counter base | `pes6.exe+C17E50` |
| Away assist counter base | `pes6.exe+C17E54` |
| Played-player map | `pes6.exe+C18DE8` |
| Home roster IDs | `pes6.exe+CD4536` |
| Away roster IDs | `pes6.exe+CD775E` |
| Raw Game History stream | `pes6.exe+C1DD70` |
| Game History metadata/index | `pes6.exe+C18F50` |
| Game History raw-byte offset | `pes6.exe+C24170` |
| Game History record count | `pes6.exe+C24172` |
| Native event-size table | `pes6.exe+C24174` |
| Native match-stat dispatcher | `pes6.exe+1FFDE0` |
| Native goal-history builder | `pes6.exe+200350` |
| Native history appender path | `pes6.exe+203DA0` |
| Native history metadata rebuild | `pes6.exe+203E80` |

These addresses are **not a compatibility contract for other patches**. Revalidate each address and the surrounding code/data semantics before using the same implementation elsewhere.

### 14.3 Add Home/Away Goal

Final stable Add Goal behavior is **scorer-only**.

It intentionally does not create a synthetic assistant.

High-level transaction:

1. require Raw remaining time > 0;
2. resolve the played-player map and roster;
3. choose one verified real played outfield scorer;
4. snapshot score, goal/assist counters and hidden per-period goal counters;
5. build a native event with assistant identity `0xFF` ("no assistant");
6. call the verified native stat dispatcher with event kind `6`;
7. verify:
   - total goal counter increased by exactly 1;
   - exactly one hidden per-period goal counter increased;
   - assist counters did not change;
8. update gameplay + display score bytes;
9. call PES6's native goal-history builder;
10. verify native history count and byte offset increased exactly as expected;
11. roll back touched counters/scores on failure.

Goalkeepers are excluded from the random scorer candidate list.

### 14.4 Hidden per-period goal counters

PES6 keeps four WORD goal counters immediately before the visible/total goal counter:

`goalCounterAddress - 8 + period*2`, for period `0..3`.

These counters matter to Master League post-match goal statistics.

Changing only the visible total goal counter is not sufficient.

### 14.5 Native Game History is variable-length

A critical rule:

> **Do not assume every Game History record is 8 bytes.**

The stable Remove/Reset implementation parses records using:

- metadata offsets;
- metadata event type;
- the native event-size table.

The raw stream is therefore treated as a variable-length event stream.

### 14.6 Remove Home/Away Goal

Remove:

1. requires Raw > 0;
2. verifies current scores and native history agree;
3. parses the full native event stream;
4. selects the most recent goal for the requested side;
5. resolves the scorer from the goal event;
6. resolves an assistant only if the native record actually has one;
7. snapshots raw history, metadata, score and player counters;
8. removes only the selected native goal bytes;
9. decrements:
   - scorer total;
   - one hidden per-period scorer counter;
   - assistant match counter when the removed natural goal had an assistant;
10. decrements the requested side score;
11. calls PES6's native history metadata rebuild routine;
12. reparses and verifies the rebuilt history;
13. restores the full snapshot if any step fails.

This is why Remove can safely remove both synthetic scorer-only goals and natural goals with assists.

### 14.7 Reset Score to 0–0

Reset does not wipe the entire Game History.

It:

- removes native **goal events**;
- preserves non-goal events;
- zeros both scores;
- zeros goal and assist match counters;
- zeros hidden per-period goal counters;
- rebuilds native Game History metadata;
- verifies the final structure.

### 14.8 Raw = 0 guard

Add / Remove / Reset are blocked when `Remaining Match Time (Raw) == 0`.

Raw 0 can represent:

- stoppage-time / period transition windows;
- half-time;
- full-time;
- celebration / teardown transitions.

Do not remove this guard just to make the buttons callable at every moment.

---

## 15. Native Routine Signature Verification

Score Controls verify important PES6 routines before changing match state.

The routine names below are **internal descriptive labels used by this project**. They describe how the stable CT uses the routines; they are not executable symbol names or a claim about Konami's original naming.

Current stable signatures include:

| Project label | Module offset | Expected beginning |
|---|---:|---|
| Match-history appender path | `+203DA0` | `0F B7 05 70 41 02 01` |
| Native match-stat dispatcher | `+1FFDE0` | `56 8B 74 24 0C 85 F6` |
| Native goal-history builder | `+200350` | `83 EC 18 53 56 57 8B 7C 24 28` |
| Native metadata rebuild | `+203E80` | `51 55 0F B7 2D 72 41 02 01` |

If a patch changes these bytes, the safe behavior is to abort the action and add compatibility for that build deliberately.

Do not simply delete signature checks to make an unsupported EXE "work".

---

## 16. Match Clock Controls

### 16.1 Native match-time setting

Native match-time mode:

`pes6.exe+388F1B0`, low 3 bits.

### 16.2 Custom selector

Editor-owned selector:

`pes6_ml_match_time_selector`

Allowed values:

- `0` = PES6 Native
- `3` = 3 Minutes
- `7` = 7 Minutes
- `12` = 12 Minutes

Mapping:

| Custom target | PES6 native base | Native mode |
|---:|---:|---:|
| 3 min | 5 min | 0 |
| 7 min | 10 min | 1 |
| 12 min | 15 min | 2 |

These two duration settings should be changed **from the Master League menu before the match begins**.

Once the match is running, the pre-match selector/native duration is locked. `Remaining Match Time (Raw)` is the only direct live clock control.

### 16.3 Raw time

Raw clock:

`pes6.exe+37E098C`

The custom-time controller observes the normal raw-time decrease and applies an additional controlled decrease to reach the selected target duration without changing the native base after kickoff.

For a selected custom duration, the raw safety limit is based on the matching native base:

`safeMax = baseMinutes * 1800`

Out-of-range edits are rejected and the previous safe value is restored.

### 16.4 End Current Period

`End Current Period` is a one-shot action that asks for confirmation and writes Raw time to 0.

---

## 17. Infinite Stamina

Infinite Stamina uses a dedicated AOB hook rather than a 60 ms write loop.

It determines the runtime Home/Away side for the user's Master League team and only replaces stamina depletion on that side.

Implementation requirements to preserve if extending it:

- keep side detection runtime-based;
- do not permanently force Home or Away in normal operation;
- preserve registers/flags around the injected code;
- restore the original instruction bytes on disable;
- clean up every allocated/registered symbol.

---

## 18. Master League Settings and Finances

Current tested-build implementation references include:

| Value | Address | Portability note |
|---|---|---|
| Match Time (PES6 Native) | `pes6.exe+388F1B0`, low 3 bits | Tested-build module offset |
| Game Difficulty | `pes6.exe+388F1B1`, low 3 bits | Tested-build module offset |
| Transfer Frequency | `pes6.exe+388F1B5`, bits 0–1 | Tested-build module offset |
| Master League Difficulty | `pes6.exe+388F1B5`, bits 3–5 | Tested-build module offset |
| Current Funds | `03CE229C` | **Patch-sensitive absolute address** |

`Current Funds` is intentionally marked **patch-sensitive** because the stable CT currently references it through a direct absolute address rather than a runtime-resolved module-relative target. Revalidate it independently before claiming compatibility with another executable or patch.

---

## 19. Performance / CPU Rules

The stable v1.0 runtime was cleaned up specifically to avoid unnecessary continuous work.

### Do

- cache stable addresses once when possible;
- compare before writing;
- use per-player/per-team caches;
- invalidate caches when team signatures or generations change;
- perform expensive recovery scans only when normal resolution fails;
- use cooldowns for repair logic;
- make visible actions one-shot;
- prefer AOB hooks for truly event-driven game behavior.

### Do not

- run `AOBScan` across the full process every 60 ms;
- resolve the same fixed symbol repeatedly inside a hot loop when it can be cached;
- rewrite all 26 players continuously to "keep a preset active";
- repeatedly write a value that is already correct;
- scan all memory every time the user clicks a normal Player Editor field;
- leave one-shot action records permanently active.

### Current runtime examples

The 60 ms loop caches the real/mirror score addresses once and only writes a status mirror if its displayed value differs from the real score.

Squad preset persistence also uses event/change detection rather than rewriting the whole squad every timer tick.

---

## 20. Safe AOB Hook Pattern

For a new code hook, follow this pattern conceptually:

```text
[ENABLE]

aobscanmodule(MY_HOOK,pes6.exe,<specific signature>)
assert(MY_HOOK,<expected original bytes>)
alloc(MY_MEM,1024,MY_HOOK)

label(return)

MY_MEM:
  // preserve every register/flag your code changes
  // perform the minimum required work
  // reproduce overwritten original instructions
  jmp return

MY_HOOK:
  jmp MY_MEM
  nop
return:

registersymbol(MY_HOOK)

[DISABLE]

MY_HOOK:
  db <exact original bytes>

unregistersymbol(MY_HOOK)
dealloc(MY_MEM)
```

Rules:

- choose a signature that is specific enough to be unique;
- verify both supported EXEs;
- preserve original semantics;
- do not assume a register survives if your injected code modifies it;
- do not skip `assert`;
- restore exactly the bytes you replaced.

---

## 21. Safe Lua Action Pattern

A visible action should normally be one-shot:

```text
[ENABLE]
{$lua}
if syntaxcheck then return end

local ok,err=pcall(function()
  -- validate state
  -- perform one transaction
  -- verify result
end)

if not ok then
  showMessage('Action failed:\n\n'..tostring(err))
end

local r=memrec
local t=createTimer(nil,false)
t.Interval=100
t.OnTimer=function(x)
  x.destroy()
  if r and r.Active then r.Active=false end
end
{$asm}

[DISABLE]
```

For complex memory edits, add a snapshot/rollback phase inside the transaction.

---

## 22. Adding a New Player Editor Field — Checklist

Example workflow:

1. identify the correct value and whether it belongs to:
   - the 72-byte live player block;
   - contract/fitness record;
   - another Master League structure;
2. verify it on both supported EXEs;
3. if it is in the Player Editor block:
   - allocate a stable editor symbol;
   - load/decode it in the shared player-load path;
   - include it in pack/signature state;
   - encode it in the shared write-back path;
   - preserve unrelated bits;
4. add validation/clamping;
5. add a visible CheatEntry pointing to the editor symbol;
6. test normal player;
7. test goalkeeper;
8. test transferred/unusual player shadow mode;
9. test after a match reload;
10. test after leaving/re-entering Master League.

---

## 23. Adding a New Match Feature — Checklist

Match features require stricter rules than ordinary Player Editor fields.

1. identify whether the value is:
   - UI only;
   - gameplay state;
   - post-match persistent state;
   - native event/history state;
2. determine all duplicate/derived copies that PES6 expects to remain synchronized;
3. add a match-running / Raw-time guard if the action is unsafe during transitions;
4. signature-check every native function called with `executeCodeEx`;
5. snapshot every memory area you will change;
6. make the smallest possible change;
7. call native rebuild/update routines when the game owns metadata;
8. verify final counters and structure;
9. roll back on failure;
10. test:
    - first half;
    - stoppage/Raw 0;
    - half-time;
    - second half;
    - full-time;
    - match result screen;
    - Master League menu after the match.

Never infer that a value is "just visual" because changing one byte appears to work in real time.

---

## 24. Lessons From Score / History Development

These are worth keeping in the repository because they prevent future regressions.

### 24.1 A visually correct edit can still corrupt later processing

A score/history edit may look correct on screen but crash when PES6 processes the structure at half-time or full-time.

Always test transition boundaries.

### 24.2 Game History has metadata

Raw event bytes and the metadata/index table must agree.

If records are removed, rebuild metadata with the native rebuild routine.

### 24.3 Event sizes are native and variable

Use the game's event-size table. Do not hard-code a universal 8-byte event size.

### 24.4 Goal statistics have hidden state

The total goal counter is not the whole story. The four preceding per-period WORD counters participate in Master League post-match goal statistics.

### 24.5 Synthetic assists are intentionally outside the stable v1.0 scope

Stable v1.0 deliberately creates **scorer-only synthetic goals** because persistent Master League assist-commit behavior was not validated to the release standard required for this project.

This is a stability boundary, not an assumption that the live assist field is unimportant. Incrementing the live assist counter or placing an assistant identity in a goal-history record is not, by itself, sufficient evidence that the assist will be committed correctly to persistent Master League statistics.

If synthetic assist support is revisited, treat it as a separate reverse-engineering feature: identify and verify the complete post-match assist commit path, test it across match transitions, and only then promote it into a stable release.

---

## 25. Diagnostics

The stable table exposes read-only diagnostic rows including:

- Last Resolver Player ID
- Selected Player Record Address
- Live Write Counter
- Selected Contract Record Address
- Requested Player ID
- Selected Roster Slot
- Selected Player ID Occurrences

Use these before adding temporary intrusive hooks.

For development builds, temporary diagnostics are fine, but remove experimental capture counters/hooks from the public stable CT when they are no longer needed.

---

## 26. Common Failure Modes

### Wrong player edited

Usually caused by bypassing the shared roster/slot resolver and trusting a live resolver pointer or Player ID directly.

### Preset works until match end, then crashes

Usually indicates stale pointers or writes into temporary post-match/cutscene records.

### Preset disappears after a match

Usually means PES6 rebuilt ability copies. Use the existing mass-reload repair architecture rather than a permanent high-frequency rewrite loop.

### Add/Remove looks correct but later crashes

Treat every affected native structure as incomplete until score, player counters, raw history, metadata and any hidden statistics are all verified.

### CPU rises while table is idle

Look for:

- full-process scans in timers;
- repeated symbol resolution;
- unconditional writes;
- per-tick loops over all players;
- repair code without cooldown/generation checks.

---

## 27. Compatibility Strategy for a New Patch

When adding another PES6 patch:

1. record executable size and SHA-256;
2. test main resolver AOB;
3. test My Team name-build AOB;
4. test ML ability-sync AOB;
5. test optional Infinite Stamina AOB;
6. verify every fixed match/module offset used by the feature you want;
7. verify native function signatures;
8. test roster detection with at least 11 players;
9. test a transferred/unusual player;
10. test Player Selector and Player Editor;
11. test squad presets across a match;
12. test championship/celebration if possible;
13. test score Add/Remove/Reset through half-time/full-time;
14. test Native and Custom Match Time;
15. check idle CPU usage.

Do not mark a patch as supported because `[ACTIVATE]` alone succeeds.

---

## 28. Contribution / Fork Rules Recommended for This Project

For maintainable forks:

- keep CheatEntry IDs unique;
- keep visible descriptions user-facing and concise;
- keep experimental diagnostics clearly marked;
- document every new fixed address or native function;
- document whether a feature is temporary or save-persistent;
- add `[DISABLE]` cleanup for every resource created on enable;
- do not silently remove safety checks to support another EXE;
- test on both original supported builds after changing shared core code;
- keep README behavior descriptions synchronized with the CT;
- update the stable CT SHA-256 when releasing a modified build.

The project is distributed under **GPL-3.0**. Forks and redistributed modified versions should follow the license terms and preserve the relevant notices/source availability required by that license.

---

## 29. Useful Internal Extension Points

| Function / symbol | Intended use |
|---|---|
| `PES6MLSetSingleTeamSlot(slot, enabled)` | select one verified squad slot |
| `PES6MLLockTeamSlot(slot)` | resolve/lock a slot into Player Editor |
| `PES6MLApplyEditorAction(kind)` | selected-player quick actions |
| `PES6MLApplyPlayerPreset(key)` | selected-player profiles |
| `PES6MLApplySelectedRecovery(mode)` | selected-player fitness recovery |
| `PES6MLApplyTeamRecovery(mode)` | full-squad recovery |
| `PES6MLRequestTeamPreset(mode)` | public squad preset request |
| `PES6MLSetTeamPreset(mode)` | validated squad preset transaction |
| `PES6MLRecoverOverviewTeamPointers()` | recover verified squad pointers |
| `PES6MLUpdateFitnessOverview()` | build/update Overview |
| `PES6MLUpdatePostMatchSafety()` | unsafe transition guard |
| `PES6MLResetMatchClockControls(resetRaw)` | release/reset Raw controller state |
| `ml3_team_ids` | stable 32-slot roster ID map |
| `ml3_selected_id` / `ml3_selected_ptr` | current Player Editor identity/record |
| `ml3_contract_record_ptr` | selected ML contract/fitness record |
| `ml3_ability_values` | editor-side 26-ability buffer |
| `ml3_position_flags` | editor-side 12-position flags |
| `ml3_special_flags` | editor-side 23-special flags |

Functions exist only while the main editor is activated.

---

## 30. Final Rule

When adding a feature, prefer:

> **verified roster → validated target → minimal transaction → native synchronization when required → verification → rollback on failure**

over:

> **found an address once → write it continuously**

That principle is the core of the stable v1.0 architecture.
