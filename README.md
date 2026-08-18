<a id="top"></a>

# PES6 Master League Editor

### A comprehensive Cheat Engine editor for Pro Evolution Soccer 6 Master League

**Version:** 1.0.1
**Author:** jackcohle  
**Platform:** PC  
**Requirements:** Cheat Engine + Pro Evolution Soccer 6  
**Important:** Run Cheat Engine **as Administrator** before attaching to PES6.

**English** | [Türkçe](README_TR.md)

[![Latest Release](https://img.shields.io/github/v/release/jackcohle/PES6-Master-League-Editor?label=release)](https://github.com/jackcohle/PES6-Master-League-Editor/releases/latest)
[![License: GPL-3.0](https://img.shields.io/github/license/jackcohle/PES6-Master-League-Editor)](https://github.com/jackcohle/PES6-Master-League-Editor/blob/main/LICENSE)
![Platform](https://img.shields.io/badge/platform-Windows-lightgrey)
![Game](https://img.shields.io/badge/game-PES%206-lightgrey)

### ⬇ Download

**[Download the latest stable release](https://github.com/jackcohle/PES6-Master-League-Editor/releases/latest)**

**Video / Demo:** `https://www.youtube.com/watch?v=as4FTyxqqKM`

**SHA-256 — v1.0.1 stable CT**  
`488e5ab12777a24b8276aa1f712f0911f311a2a6660a96d5177fc1319d0efeb9`

> Full documentation is provided below. Back up your Master League save before making permanent edits.

![PES6 Master League Editor](assets/main.png)

---
## Table of Contents
- [About the Project](#about)
- [Compatibility & Tested Builds](#compatibility)
- [Installation](#installation)
- [Important Usage Notes](#usage-notes)
- [Main Features](#main-features)
  - [1. Automatic Master League Squad Detection](#1-squad-detection)
  - [2. Player Selector](#2-player-selector)
  - [3. Quick Player Actions](#3-quick-player-actions)
  - [4. Player Profiles](#4-player-profiles)
  - [5. Player Abilities](#5-player-abilities)
  - [6. Playable Positions](#6-playable-positions)
  - [7. Special Abilities](#7-special-abilities)
  - [8. Performance & Player Settings](#8-performance-settings)
  - [9. Identity & Physical Profile](#9-identity)
  - [10. Fitness, Condition & Recovery](#10-fitness)
  - [11. Contract & Salary](#11-contract)
  - [12. Player Development](#12-player-development)
  - [13. Squad Tools](#13-squad-tools)
  - [14. Squad Ability Presets](#14-squad-presets)
  - [15. Squad Fitness Overview](#15-fitness-overview)
  - [16. Master League Settings](#16-ml-settings)
  - [17. Club Finances](#17-finances)
  - [18. Match Controls](#18-match-controls)
  - [19. Score Controls](#19-score-controls)
  - [20. Match Clock Controls](#20-match-clock)
  - [21. Advanced Settings](#21-advanced)
  - [22. Diagnostics — Read Only](#22-diagnostics)
- [Permanent and Temporary Changes](#permanent-temporary)
- [Safety](#safety)
- [v1.0.1 — Player Development](#v101-release)
- [v1.0 — First Public Release](#v10-release)
- [Feature Summary](#feature-summary)
- [License](#license)
- [Author](#author)

> On GitHub desktop, you can also use the heading-based **Outline** navigation.

---

<a id="about"></a>
## About the Project
**PES6 Master League Editor** is a comprehensive Cheat Engine table built specifically for the Master League mode in Pro Evolution Soccer 6.

The goal of the project is not to provide a few isolated cheats, but to act as a complete **Master League editor** that can automatically detect the current squad, safely resolve the correct player, and manage player and team data from a single interface.

The following major sections all use the same verified Master League roster:

- Player Selector
- Player Editor
- Player Development
- Squad Recovery
- Squad Ability Presets
- Squad Fitness Overview

This prevents different parts of the table from targeting unrelated player lists or the wrong player records.

The table also handles Master League players whose live ability data does not follow the standard player-record layout. When such a player is edited for the first time, a very short one-time scan may occur. After the correct live record is found, the address is stored in a validated cache and later edits are applied immediately. The system is generic and does not hard-code a specific player name or Player ID.


[↑ Back to top](#top)

---
<a id="compatibility"></a>
## Compatibility & Tested Builds
v1.0.1 targets the same two PES6 setups used for development and runtime testing of v1.0:

| Environment | Tested executable | Status |
|---|---|---|
| **Standard / unpatched PES6** | `PES6.exe` | ✅ Tested |
| **PES 6 Original Season** patch | `pes6.exe` | ✅ Tested |

Both tested executables are **32-bit PE (x86)** files and are **21,880,832 bytes** in size, but their contents are not identical.

**SHA-256 — Standard PES6**  
`53263fb86b10b1bd2a9a962816c55ba23954e8f0596da80e8adebb4fead3295e`

**SHA-256 — PES 6 Original Season**  
`cd30427917be6a903ea4624147ca8506c7db5462a4a4e9f50ee8dd6c9d494628`

> The SHA-256 values are provided only to identify the exact executables used during editor testing. Your executable does not need to have the same filename or hash for the editor to work.

### Patch compatibility

The editor was developed with the goal of avoiding dependence on a single fixed PES6 setup.

Testing and validation are currently limited to **Standard / unpatched PES6** and **PES 6 Original Season**.

Compatibility with other patches is **not guaranteed**.

The core Player/Squad systems use runtime validation and recovery mechanisms where appropriate, but executable changes made by third-party patches can still affect individual features.

[↑ Back to top](#top)

---
<a id="installation"></a>
## Installation
1. Start PES6.
2. Run Cheat Engine **as Administrator**.
3. Attach Cheat Engine to `pes6.exe`.
4. Open `PES6-Master-League-Editor-v1.0.1-by-jackcohle-FINAL.CT`.
5. Enable `[ACTIVATE] PES6 Master League Editor v1.0.1`.
6. Load your Master League save.
7. Wait for the squad to be detected automatically.
8. Use `Player Selector` before editing a single player.
9. Use Player Editor, Player Development, or Squad Tools as needed.

[↑ Back to top](#top)

---
<a id="usage-notes"></a>
## Important Usage Notes
### Run Cheat Engine as Administrator

Cheat Engine should be started with **Run as administrator** before attaching to PES6; this avoids permission-related attach/write problems, especially when the game itself is elevated.

### Use Player Selector

Before editing an individual player, select that player through **Player Selector**. Player Editor, fitness/contract editing and the squad tools are designed around the verified Master League roster.

### A short pause on the first unusual-player edit can be normal

Some Master League players may use a different live-record layout. The first time one of these players is resolved, a short validation scan can occur; once found, the validated address is cached and later edits are immediate.

### Squad Ability Presets are session-based

Squad Ability Presets remain active across normal match transitions during the current Master League session. If PES6 rebuilds the squad ability records after a match, the preset is restored only to verified squad records.

Use **Restore Original Squad Ability Values** to return to the squad values captured before the active preset was applied.

### Championship / post-match celebration safety

Preset persistence no longer follows every newly surfaced resolver record with the same Player ID. This prevents squad-preset writes from being applied to temporary cutscene/celebration records while preserving normal between-match preset restoration.

Live Player Editor writes are also suspended during unsafe post-match transitions.

### Leaving and re-entering Master League

After leaving Master League, **manually untick `[ACTIVATE]` and tick it again before entering Master League again**. This clears temporary editor/session state and starts the next Master League entry from a clean runtime state.

### Squad changes after transfers

If the displayed roster is still the old one after a transfer, leave and re-enter the Master League screen so the current squad can be detected again. If you leave Master League completely, also perform the `[ACTIVATE]` reset described above.

### Using Automatic Player Development

**Automatic Player Development** can be enabled once for the current Master League session and left active. It adds age-based development EXP only to players who actually appear in the match. Starting XI players and substitutes who enter the match are included; unused substitutes are left untouched.

When `[ACTIVATE]` is disabled, the development timer, breakpoint, and temporary session state are cleared together with the main editor reset. Player development already processed by PES6 after previous matches is not reverted.

### Pause before using match controls

Pausing is recommended before changing score or remaining time. **Add Home Goal / Add Away Goal** also work from the in-match ESC statistics screens. Do not use **Add / Remove / Reset** while **Remaining Match Time (Raw) = 0**; this includes stoppage-time and period-transition windows, and the Actions remain blocked until Raw time becomes positive again.

### Avoid extreme synthetic goal histories

PES6's goal-history presentation was designed for normal match totals rather than dozens of synthetic entries. Use **Add Home Goal / Add Away Goal** for normal score ranges. Add Goal creates a real scorer entry but intentionally creates **no assist**. Remove Goal and Reset rebuild the native goal history and synchronize the affected goal/assist match statistics.

[↑ Back to top](#top)

---

<a id="main-features"></a>
## Main Features

<a id="1-squad-detection"></a>
### 1. Automatic Master League Squad Detection

After the editor is activated and Master League is loaded, the current squad is detected automatically.

Once detected, the same player list is shared by:

**Player Selector**, **Squad Ability Presets**, and **Squad Fitness Overview**.

This is especially important for transferred players, goalkeepers, youth players, and Master League records that may behave differently from normal PES6 player database entries.

Unused squad-capacity slots are not presented as real players; Player Selector focuses on the actual detected squad.

[↑ Back to top](#top)

---
<a id="2-player-selector"></a>
### 2. Player Selector

Before using Player Editor, choose the player you want to edit from **Player Selector**.

![PES6 Master League Editor](assets/player_selector.png)

When a player is selected:

- Player Editor locks onto that player.
- The real Master League Player ID is used.
- The player name is resolved from verified roster/player records.
- Ability data is loaded.
- Fitness and contract records are resolved.
- The player remains selected until another squad player is chosen.

Only **one player at a time** is the Player Editor target.

The purpose of this system is to reduce the risk of editing the wrong player.

#### Special Live Record Handling

Some players do not use the normal:

`Full Player Record → Current Ability Record`

path.

When one of these players is edited for the first time, the table may perform a short validation scan to locate the correct live ability record.

Once resolved, the result is stored in a validated cache so later edits are immediate. If that cached target is no longer valid after a reload or memory rebuild, the editor discards it and resolves the player again safely.

[↑ Back to top](#top)

---
<a id="3-quick-player-actions"></a>
### 3. Quick Player Actions

Four ready-to-use Actions are available for the currently selected player.

#### Set All 26 Abilities to 99

Sets all 26 core ability values to **99**.

Positions, Special Abilities, and physical information are not changed.

#### Enable All Special Abilities

Enables all **23 Special Abilities** for the selected player.

Core ability values are left untouched.

#### Max Performance Settings

Sets the player's additional performance-related values to their maximum levels:

- Consistency → 8
- Condition → 8
- Weak Foot Accuracy → 8
- Weak Foot Frequency → 8
- Injury Tolerance → A

This Action does not change the 26 core ability values.

#### Create Complete Superstar

Combines the three Actions above:

- 26 Abilities → 99
- All Special Abilities → enabled
- Consistency → 8
- Condition → 8
- Weak Foot Accuracy → 8
- Weak Foot Frequency → 8
- Injury Tolerance → A

This is the one-click option for turning the selected player into a complete superstar.

[↑ Back to top](#top)

---
<a id="4-player-profiles"></a>
### 4. Player Profiles

Player Profiles are designed to build role-specific elite players instead of simply setting everything to 99.

Each profile:

- Uses custom values for all 26 abilities.
- Sets appropriate playable positions.
- Sets Registered Position.
- Enables role-specific Special Abilities.
- Adjusts Consistency / Condition / Weak Foot / Injury values.

**Elite Wonderkid** is the exception: it keeps the player's current positions.

#### Elite Centre Forward

Main position: **CF**

Playable positions:

- CF
- SS

Important enabled Special Abilities:

- Positioning
- Reaction
- Scoring
- 1-on-1 Scoring
- Post Player
- Line Position
- Middle Shooting
- Centre
- Penalties
- 1-Touch Pass
- Outside

Uses Consistency 8, Condition 8, Weak Foot Accuracy 7, Weak Foot Frequency 7, and Injury Tolerance A.

#### Elite Playmaker

Main position: **AMF**

Positions:

- CMF
- AMF
- SS

Special Abilities:

- Dribbling
- Tactical Dribbling
- Positioning
- Playmaking
- Passing
- Middle Shooting
- Side
- Centre
- 1-Touch Pass
- Outside

Weak Foot Accuracy is raised to 8.

#### Explosive Winger

Main position: **WF**

Positions:

- SMF
- WF
- SS

Focused on pace, acceleration, and dribbling.

Special Abilities:

- Dribbling
- Tactical Dribbling
- Positioning
- Reaction
- Scoring
- 1-on-1 Scoring
- Side
- 1-Touch Pass
- Outside

#### Complete Midfielder

Main position: **CMF**

Positions:

- DMF
- CMF
- AMF

Focused on passing, stamina, teamwork, and two-way midfield play.

Special Abilities:

- Tactical Dribbling
- Positioning
- Playmaking
- Passing
- Middle Shooting
- 1-Touch Pass
- Marking
- Sliding
- Covering

#### Defensive Midfielder

Main position: **DMF**

Positions:

- CB
- DMF
- CMF

Focused on defense, response, stamina, mentality, and teamwork.

Special Abilities:

- Positioning
- Reaction
- Passing
- Centre
- Marking
- Sliding
- Covering
- D-Line Control

#### World-Class Centre Back

Main position: **CB**

Positions:

- CWP
- CB
- SB

Focused on Defense, Body Balance, Response, Heading, Jump, and Mentality.

Special Abilities:

- Positioning
- Reaction
- Centre
- Marking
- Sliding
- Covering
- D-Line Control
- Long Throw

#### Elite Goalkeeper

Main position: **GK**

Designed specifically around Goal Keeping and goalkeeper performance.

Special Abilities:

- Positioning
- Reaction
- Penalty Stopper
- 1-on-1 Stopper
- Long Throw

#### Elite Wonderkid — Keep Current Position

Designed as a high-potential young star profile.

Unlike the other profiles:

**the player's existing playable positions and Registered Position are preserved.**

Enabled Special Abilities:

- Dribbling
- Tactical Dribbling
- Playmaking
- Passing
- 1-Touch Pass

Condition 8, Consistency 7, Weak Foot Accuracy 7, Weak Foot Frequency 7, and Injury Tolerance A are applied.

---

<details>
<summary><strong>Show full 26-Ability values used by all Player Profiles</strong></summary>

Ability order:

`Attack / Defense / Body Balance / Stamina / Top Speed / Acceleration / Response / Agility / Dribble Accuracy / Dribble Speed / Short Pass Accuracy / Short Pass Speed / Long Pass Accuracy / Long Pass Speed / Shot Accuracy / Shot Power / Shot Technique / Free Kick Accuracy / Curling / Heading / Jump / Technique / Aggression / Mentality / Goal Keeping / Team Work`

#### Elite Centre Forward
`94, 45, 88, 84, 88, 91, 93, 87, 88, 87, 78, 79, 73, 76, 94, 91, 94, 75, 82, 90, 87, 90, 90, 86, 50, 82`

#### Elite Playmaker
`90, 55, 78, 86, 84, 88, 86, 92, 95, 91, 96, 90, 95, 90, 84, 86, 88, 94, 96, 65, 70, 96, 75, 88, 50, 96`

#### Explosive Winger
`89, 50, 74, 88, 97, 98, 86, 94, 93, 98, 86, 88, 91, 92, 84, 85, 86, 78, 91, 70, 75, 90, 84, 82, 50, 86`

#### Complete Midfielder
`86, 82, 84, 94, 85, 85, 89, 86, 88, 85, 93, 91, 91, 90, 84, 88, 85, 86, 88, 78, 82, 90, 85, 92, 50, 96`

#### Defensive Midfielder
`72, 94, 90, 94, 78, 76, 94, 76, 78, 74, 88, 86, 87, 85, 70, 84, 72, 68, 72, 84, 91, 82, 86, 95, 50, 94`

#### World-Class Centre Back
`60, 97, 96, 88, 79, 75, 96, 70, 70, 68, 79, 82, 84, 86, 60, 88, 62, 55, 60, 94, 97, 75, 88, 96, 50, 90`

#### Elite Goalkeeper
`45, 95, 90, 78, 65, 68, 97, 75, 55, 50, 72, 78, 78, 82, 45, 86, 50, 55, 60, 70, 88, 70, 55, 96, 99, 88`

#### Elite Wonderkid
`84, 70, 78, 86, 90, 92, 84, 90, 88, 91, 86, 84, 84, 85, 82, 84, 83, 78, 83, 76, 80, 88, 82, 84, 55, 86`

---

</details>

<a id="5-player-abilities"></a>
### 5. Player Abilities

All **26 core PES6 ability values** can be edited individually.

Each field controls a different part of the selected player's in-game performance:

- **Attack** — Affects the player's overall attacking effectiveness and tendency to occupy useful attacking areas.
- **Defense** — Affects the player's overall defensive effectiveness and positioning.
- **Body Balance** — Affects strength, stability and resistance to physical challenges.
- **Stamina** — Determines how well the player maintains physical performance as the match progresses.
- **Top Speed** — Determines the player's maximum running speed.
- **Acceleration** — Determines how quickly the player reaches higher running speed.
- **Response** — Affects reactions to loose balls, rebounds and rapidly changing situations.
- **Agility** — Affects turning, direction changes and general movement responsiveness.
- **Dribble Accuracy** — Affects close control and reliability while carrying the ball.
- **Dribble Speed** — Determines how much running speed can be maintained while dribbling.
- **Short Pass Accuracy** — Determines the precision of short and medium-range passes.
- **Short Pass Speed** — Determines the pace of short and medium-range passes.
- **Long Pass Accuracy** — Determines the precision of long passes, switches and crosses.
- **Long Pass Speed** — Determines the pace of long passes, switches and crosses.
- **Shot Accuracy** — Determines how accurately the player places shots on goal.
- **Shot Power** — Determines the force and speed of shots.
- **Shot Technique** — Affects shooting quality from awkward body positions, volleys and difficult situations.
- **Free Kick Accuracy** — Determines the accuracy of direct free kicks.
- **Curling** — Affects the amount of bend applied to shots, free kicks and crosses.
- **Heading** — Determines accuracy and effectiveness when playing the ball with the head.
- **Jump** — Determines the player's ability to rise for aerial balls.
- **Technique** — Affects first touch, ball control and technical execution in difficult situations.
- **Aggression** — In PES6, mainly affects the player's tendency to push forward and join attacking situations.
- **Mentality** — Affects the player's ability to keep responding and performing under difficult match conditions.
- **Goal Keeping** — Determines basic goalkeeping effectiveness.
- **Team Work** — Affects off-ball support, team movement and coordination with teammates.

This allows complete manual player creation without using a profile or a 99 Action.

[↑ Back to top](#top)

---

<a id="6-playable-positions"></a>
### 6. Playable Positions

Playable positions can be enabled or disabled individually with a `Yes / No` selector.

- **GK — Goalkeeper** — Allows the player to be used as a goalkeeper.
- **CWP — Sweeper** — Allows the player to be used as a sweeper behind the defensive line.
- **CB — Centre Back** — Allows the player to be used as a central defender.
- **SB — Side Back** — Allows the player to be used as a full-back on either side.
- **DMF — Defensive Midfielder** — Allows the player to be used as a defensive midfielder in front of the back line.
- **WB — Wing Back** — Allows the player to be used as a wing-back with defensive and wide attacking duties.
- **CMF — Centre Midfielder** — Allows the player to be used as a central midfielder.
- **SMF — Side Midfielder** — Allows the player to be used as a wide midfielder.
- **AMF — Attacking Midfielder** — Allows the player to be used as an attacking midfielder behind the forwards.
- **WF — Wing Forward** — Allows the player to be used as an advanced wide forward.
- **SS — Second Striker** — Allows the player to be used as a supporting striker.
- **CF — Centre Forward** — Allows the player to be used as the main central striker.

The player's main **Registered Position** is edited separately in Performance & Player Settings.

[↑ Back to top](#top)

---

<a id="7-special-abilities"></a>
### 7. Special Abilities

All **23 PES6 Special Abilities** can be enabled or disabled independently with `Yes / No`.

- **Dribbling** — Improves the player's effectiveness when taking opponents on with the ball.
- **Tactical Dribbling** — Improves controlled dribbling and ball retention in tighter situations.
- **Positioning** — Improves the player's ability to find useful positions during active play.
- **Reaction** — Improves quick attacking reactions to opportunities and loose balls.
- **Playmaking** — Improves the player's influence when organizing attacks and creating passing options.
- **Passing** — Improves special passing behavior beyond the base passing attributes.
- **Scoring** — Improves effectiveness in goalscoring situations.
- **1-on-1 Scoring** — Improves finishing effectiveness when facing the goalkeeper directly.
- **Post Player** — Improves receiving, shielding and linking play with a defender close behind.
- **Line Position** — Improves attacking movement along the defensive line when searching for space.
- **Middle Shooting** — Improves effectiveness and tendency when shooting from medium or long range.
- **Side** — Improves suitability and movement in wide attacking areas.
- **Centre** — Improves suitability and movement through central attacking areas.
- **Penalties** — Improves penalty-kick effectiveness.
- **1-Touch Pass** — Improves the ability to play clean passes with a single touch.
- **Outside** — Improves use of the outside of the foot for passes and shots when appropriate.
- **Marking** — Improves the player's ability to track and stay with an opponent.
- **Sliding** — Improves effectiveness when making sliding tackles.
- **Covering** — Improves defensive covering when teammates move out of position.
- **D-Line Control** — Improves influence on defensive-line organization.
- **Penalty Stopper** — Improves goalkeeper effectiveness against penalty kicks.
- **1-on-1 Stopper** — Improves goalkeeper effectiveness in one-on-one situations.
- **Long Throw** — Enables stronger and longer throw-ins.

[↑ Back to top](#top)

---

<a id="8-performance-settings"></a>
### 8. Performance & Player Settings

Additional player settings outside the 26 core abilities are also editable.

#### Preferred Foot
- Right / Left

Sets which foot the player primarily uses.

#### Free Kick Style
- Raw 0–15

Changes the player's free-kick animation/style index without directly changing Free Kick Accuracy.

#### Penalty Kick Style
- Style 1–8

Changes the player's penalty-kick animation/style.

#### Dribbling Style
- Style 1–4

Changes the player's dribbling animation/style.

#### Drop Kick Style
- Style 1–4

Changes the goalkeeper drop-kick animation/style.

#### Registered Position
Choose one of the 12 PES6 positions.

Sets the player's primary position used by PES6 for role and squad information.

#### Consistency
- 1–8

Controls how consistently the player performs from match to match.

#### Condition
- 1–8

Controls the player's form/condition tendency used by PES6.

#### Weak Foot Accuracy
- 1–8

Controls how accurately the player can use the weaker foot.

#### Weak Foot Frequency
- 1–8

Controls how often the player is willing to use the weaker foot.

#### Injury Tolerance
- C
- B
- A

Controls the player's resistance to injuries using PES6's injury-tolerance grade.

#### Favoured Side
- Raw 0–3

Edits PES6's raw preferred-side value used internally for side preference.

[↑ Back to top](#top)

---

<a id="9-identity"></a>
### 9. Identity & Physical Profile

The selected player's physical and identity values can be edited from Player Editor.

#### Height
148–211 cm

Changes the player's stored height value.

#### Weight
Raw 0–127

Changes PES6's encoded weight value rather than displaying kilograms directly.

#### Skin Colour
Raw 0–3

Changes the raw skin-colour category used by PES6.

#### Age
15–46

Changes the player's stored Master League age within the supported range.

#### Nationality
Selects the player's nationality from the original PES6 nationality list.
The editor displays country names directly instead of raw numeric nationality codes.

#### Shirt Number
1–99

Changes the selected player's current Master League squad shirt number.

Avoid assigning the same shirt number to multiple squad members.

[↑ Back to top](#top)

---

<a id="10-fitness"></a>
### 10. Fitness, Condition & Recovery

The selected player's Master League fitness values can be edited directly.

#### Match Condition

Changes the selected player's current match-condition state.

Available values:

- Excellent
- Good
- Normal
- Poor
- Terrible

#### Pre-Match Stamina
0–100

Changes the player's current pre-match stamina level.

#### Accumulated Fatigue
0–100

Changes the player's accumulated Master League fatigue.

Whenever one of these values is changed, **Squad Fitness Overview refreshes automatically**.

Four quick Actions are also available:

#### Fully Recover Selected Player

Sets:

- Condition → Excellent
- Stamina → 100
- Fatigue → 0

This is the one-click full recovery option for the selected player.

#### Restore Selected Player Stamina

Only `Stamina → 100`, leaving fatigue and condition unchanged.

#### Clear Selected Player Fatigue

Only `Fatigue → 0`, leaving stamina and condition unchanged.

#### Set Selected Player to Excellent Condition

Only `Condition → Excellent`, leaving stamina and fatigue unchanged.

These fitness values can persist after saving Master League in-game.

[↑ Back to top](#top)

---

<a id="11-contract"></a>
### 11. Contract & Salary

Contract information for the selected player in the current Master League squad can be edited.

#### Yearly Salary

Changes the yearly salary value stored in the player's Master League contract record.

The ability-dependent value shown beside a player on some in-game screens is not the same as the stored yearly salary.

#### Contract Years Remaining

Changes the number of contract years remaining for the selected player.

Available values:

- 0 Years
- 1 Year
- 2 Years
- 3 Years
- 4 Years
- 5 Years

Save Master League in-game to keep salary and contract changes.

[↑ Back to top](#top)

---

<a id="12-player-development"></a>
### 12. Player Development

**Player Development** uses PES6's native post-match development system instead of directly forcing permanent ability values.

#### Automatic Player Development

Enable it once during the current Master League session and leave it active if desired.

Only players who actually appear in the match receive the custom development EXP bonus:

- Starting XI players
- Substitutes who enter the match

Unused substitutes are left untouched.

| Age | Development EXP Bonus |
|---|---:|
| **17–21** | **+70 EXP** |
| **22–25** | **+40 EXP** |
| **26–30** | **+25 EXP** |
| **31+** | **No custom bonus** |

The bonus is **added on top of the player's existing development EXP**. Existing EXP is not reset, replaced with a fixed target, or reduced.

PES6 performs the actual ability growth through its normal post-match development process. Because the EXP is cumulative, the same bonus does not guarantee the same direct ability increase for every player or every match; the result depends on the player's existing development EXP before the match.

Automatic Player Development uses verified played-player information from the game, so only players who actually enter the match are targeted.

When `[ACTIVATE]` is disabled:

- The Automatic Development watcher is stopped.
- The development breakpoint is removed.
- The development timer is destroyed.
- Temporary development session state is cleared.
- Development status rows return to their initial state.

Player growth already processed by PES6 after previous matches is not reverted by this reset.

#### Manual Development Presets

Optional manual development Actions are also available.

**Selected Player**
- High Development — Next Match
- Peak Development — Next Match

**Entire Squad**
- High Development — Next Match
- Peak Development — Next Match

These presets do not change ability values immediately. They prepare development EXP before the match; a match must then be played so PES6 can process the resulting development through its normal post-match system.

[↑ Back to top](#top)

---

<a id="13-squad-tools"></a>
### 13. Squad Tools

While Player Editor targets one player, **Squad Tools** applies actions to the entire detected Master League squad.

![PES6 Master League Editor](assets/squad_tools.png)

#### Squad Recovery

Fitness Actions can be applied to every player in the detected squad.

#### Fully Recover Entire Squad

Sets every player to:

- Condition → Excellent
- Stamina → 100
- Fatigue → 0

#### Restore Entire Squad Stamina

All players:

`Stamina → 100`

#### Clear Entire Squad Fatigue

All players:

`Fatigue → 0`

#### Set Entire Squad to Excellent Condition

All players:

`Condition → Excellent`

Squad Recovery uses the same detected roster map as Player Selector.

[↑ Back to top](#top)

---
<a id="14-squad-presets"></a>
### 14. Squad Ability Presets

Ability presets can be applied to the entire detected squad without selecting players one by one.

The **Active Squad Preset** status displays the selected preset by name, and the detected-player counter shows how many verified squad members are available.

#### Complete Squad Boost

Applies to the entire squad:

- 26 Abilities → 99
- All 23 Special Abilities → enabled
- Max Performance Settings → enabled

In other words:

`All Abilities 99 + All Special Abilities + Max Performance`

#### Ultimate Squad by Position

Creates an extremely strong squad while preserving meaningful role differences instead of making every player identical.

Players are handled by Registered Position group:

- **GK**
- **DEF**
- **MID**
- **FWD**

The preset applies role-appropriate ability values, strong performance settings and role-appropriate Special Abilities while keeping the player's **Registered Position** and **Playable Positions** unchanged.

#### Set All Squad Abilities to 99

Sets only the 26 core ability values to 99 for every detected squad member.

#### Enable All Special Abilities — Entire Squad

Enables all 23 Special Abilities for every detected squad player without changing the 26 core ability values.

#### Max Performance Settings — Entire Squad

Applies maximum Consistency, Condition, Weak Foot and Injury Tolerance settings across the squad.

#### +5 / +10 / +15 / +20 Ability Boosts

Adds the selected amount to each player's current 26 ability values, capped at 99. The stored preset layer is restored from the expected squad values rather than repeatedly stacking the same boost after every match reload.

#### Restore Original Squad Ability Values

Restores the squad ability values captured before the active squad preset was applied.

#### Session persistence and celebration safety

Squad Ability Presets are **temporary / session-based**, but the selected preset remains active across normal matches in the same Master League session. When PES6 mass-reloads the verified squad records after a match, the preset is repaired on those squad records automatically.

The persistence system deliberately **does not reapply presets to newly surfaced resolver records**. This prevents same-ID cutscene/championship-celebration records from being mistaken for normal player records and avoids the celebration crash that can occur with aggressive resolver-following writes.

Squad Ability Presets use the same verified Master League roster as Player Selector and Squad Fitness Overview.

[↑ Back to top](#top)

---

<a id="15-fitness-overview"></a>
### 15. Squad Fitness Overview

A live squad view for monitoring fitness status in one place.

For every player it displays:

- Shirt Number
- Player Name
- Condition
- Stamina
- Fatigue

#### Color System

A player row is **green only when all three conditions are true:**

**Condition = Excellent**  
**Stamina = 100**  
**Fatigue = 0**

If any of these conditions is not met, the player is shown in **red**.

When fitness values are changed in Player Editor, the Overview refreshes automatically.

This makes it easy to see which players are fully ready at a glance.

[↑ Back to top](#top)

---
<a id="16-ml-settings"></a>
### 16. Master League Settings

Game, Master League and pre-match duration settings can be changed from the table.

#### Game Difficulty

Changes the gameplay difficulty used during matches.

- Beginner
- Amateur
- Regular
- Professional
- Top Player
- Superstar — Hidden / 6-Star

This also provides access to PES6's hidden Superstar / 6-Star difficulty.

#### Master League Difficulty

Changes the Master League management/economic difficulty.

- Very Easy
- Easy
- Normal
- Hard
- Very Hard

#### Transfer Frequency

Changes how frequently transfer activity occurs in Master League.

- Low
- Moderate
- High

#### Match Time (PES6 Native)

Edits PES6's native match-duration setting directly:

- 5 Minutes
- 10 Minutes
- 15 Minutes
- 20 Minutes
- 25 Minutes
- 30 Minutes

#### Custom Match Time

Provides the linked custom pre-match choices:

- PES6 Native
- 3 Minutes
- 7 Minutes
- 12 Minutes

Selecting **3 / 7 / 12 Minutes** automatically selects the required PES6 Native base. If **Match Time (PES6 Native)** is edited directly afterward, Custom Match Time automatically returns to **PES6 Native** so the two controls cannot silently conflict.

These two match-time settings should only be used from the **Master League menu before the match begins**. Once the match has started, the duration settings are locked and the remaining match time can only be adjusted through **Remaining Match Time (Raw)**.

[↑ Back to top](#top)

---

<a id="17-finances"></a>
### 17. Club Finances

Master League club funds can be edited.

#### Current Funds

Edit the current funds value directly.

Preset Actions:

#### Add 10,000 Funds
Adds 10,000.

#### Add 50,000 Funds
Adds 50,000.

#### Add 100,000 Funds
Adds 100,000.

#### Set Funds to 999,999
Sets funds directly to 999,999.

To keep the funds change, save Master League normally in-game. If you exit without saving, the change is lost.

[↑ Back to top](#top)

---
<a id="18-match-controls"></a>
### 18. Match Controls

A separate section provides controls that can be used during a running match.

Pausing the match is recommended before changing score or remaining time. **Add Goal** also works from the in-match ESC statistics screens; all score and clock actions are blocked during half-time, full-time and celebration transitions.

#### Infinite Stamina — My Team

Prevents in-match stamina depletion for your Master League team.

The table detects whether the Master League team is currently Home or Away at runtime, so the feature follows your team rather than a fixed scoreboard side.

[↑ Back to top](#top)

---

<a id="19-score-controls"></a>
### 19. Score Controls

Score Controls keep the scoreboard, native goal history and player match statistics synchronized.

#### Home Score / Away Score

Display-only status rows show the current Home and Away scores. They use detached mirror values: manually typing a different Value in Cheat Engine does **not** change the game score and the displayed status is restored to the real score automatically.

#### [ACTION] Add Home Goal

Adds one Home goal using PES6's native goal/stat path. One real **played outfield Home player** is selected as the scorer, the Home score is increased, the scorer's match and Master League goal statistics are updated, and PES6's native goal-history record is appended.

The generated goal intentionally uses **no assistant** (`FF = no assistant`). Goalkeepers are excluded from the random scorer selection.

#### [ACTION] Add Away Goal

Uses the same scorer-only native goal logic for the Away side: one real played outfield scorer, no assistant, synchronized score/stat updates and a native goal-history record.

#### [ACTION] Remove Home Goal / Remove Away Goal

Removes the **latest matching goal** for the selected scoreboard side. The score is reduced by 1, the matching native goal-history record is removed, and the scorer statistics are reduced accordingly. If the removed goal was a natural PES6 goal with an assistant, that assist statistic is removed as well. The score cannot go below 0.

#### [ACTION] Reset Score to 0–0

Sets the match to 0–0 by removing all native goal-history records and clearing the corresponding goal/assist match statistics. Non-goal Game History events are preserved.

Home/Away refers to scoreboard sides, not team names. **Add Home/Away Goal** can also be used from the in-match ESC statistics screens.

> **Important:** Do not use **Add / Remove / Reset** while **Remaining Match Time (Raw) = 0**. Raw=0 includes stoppage time as well as half-time, full-time and celebration/period transitions, so Score Controls are intentionally blocked until Raw time becomes positive again.

> PES6's goal-history presentation is designed for normal match totals. Avoid generating dozens of synthetic goal entries in a single match.

[↑ Back to top](#top)

---

<a id="20-match-clock"></a>
### 20. Match Clock Controls

The three match-time controls work as one linked system:

`Match Time (PES6 Native) ↔ Custom Match Time ↔ Remaining Match Time (Raw)`

#### Match Time (PES6 Native)

This is PES6's normal 5 / 10 / 15 / 20 / 25 / 30 minute pre-match setting.

#### Custom Match Time

Select **PES6 Native**, **3 Minutes**, **7 Minutes** or **12 Minutes** before kickoff.

Choosing 3 / 7 / 12 automatically selects the required Native base. Editing the Native value directly cancels Custom Match Time back to **PES6 Native**. Changing either pre-match duration control also releases any old Raw freeze/controller state so a previous live-clock edit cannot leak into the next setup.

After kickoff, **Native and Custom are locked**. They do not rescale or rewrite a match that is already running.

#### Remaining Match Time (Raw) — Edit / Freeze

This is the only live clock control after kickoff. The raw remaining value can be edited directly and the Cheat Engine row can be ticked to freeze it; untick the row to resume the clock.

When a 3 / 7 / 12 minute custom duration is active, Raw edits outside the allowed range for that active duration are rejected. Merely changing the pre-match selector does not trigger the warning.

#### End Current Period

Sets the remaining raw time to 0 after confirmation and may immediately end the current half/period.

Leaving/resetting the editor session releases Raw freeze state. After leaving Master League completely, manually untick and re-tick `[ACTIVATE]` before entering Master League again.

[↑ Back to top](#top)

---

<a id="21-advanced"></a>
### 21. Advanced Settings

This section is not required for normal use and should normally be left at its default values.

#### Editor Runtime
Default: **Enabled**

Turns the main editor runtime on or off; disabling it stops the normal detection and live editing update cycle.

#### Auto-Follow Player Resolver
Default: **Enabled**

Controls whether Player Editor may automatically follow the player currently resolved by PES6 when no squad player is manually locked.

#### Instant Live Write-Back
Default: **Enabled**

Controls whether Player Editor changes are written back to the selected player's verified live record immediately.

#### Selection Stability Checks
Default: **2 hits**

Configurable from 1–10 and defines how many consecutive resolver matches are required before an automatically detected candidate is accepted as stable.

#### Legacy Master League Mode Check
Default: **Disabled**

Enables the older Master League-only mode validation for compatibility/troubleshooting and is not normally required.

[↑ Back to top](#top)

---

<a id="22-diagnostics"></a>
### 22. Diagnostics — Read Only

Read-only technical information is available for troubleshooting and does not need to be edited during normal use.

- **Last Resolver Player ID** — Shows the most recent Player ID reported by the live PES6 player resolver.
- **Selected Player Record Address** — Shows the record currently used by Player Editor for the selected player.
- **Live Write Counter** — Shows how many live Player Editor write-back operations have occurred.
- **Selected Contract Record Address** — Shows the current Master League contract-record address for the selected player.
- **Requested Player ID** — Shows the Player ID expected for the requested Player Selector slot.
- **Selected Roster Slot** — Shows the roster slot currently locked by Player Selector.
- **Selected Player ID Occurrences** — Shows how many times the selected Player ID occurs in the detected roster data.

These values are useful when testing different PES6 executables, patches or unexpected player-record behavior.

[↑ Back to top](#top)

---

<a id="permanent-temporary"></a>
## Permanent and Temporary Changes
This distinction is important.

## Changes that can persist after saving Master League in-game

- Club Funds
- Yearly Salary
- Contract Years Remaining
- Condition
- Pre-Match Stamina
- Accumulated Fatigue
- Shirt Number
- Player Development progression processed by PES6 after matches

## Designed as temporary changes

- 26 Ability edits
- Player Profiles
- Playable Positions
- Special Abilities
- Performance boosts
- Squad Ability Presets
- Match Controls

[↑ Back to top](#top)

---
<a id="safety"></a>
## Safety
**Backing up your Master League save is strongly recommended.**

Run Cheat Engine **as Administrator** before attaching to PES6.

### Master League session reset

After leaving Master League, manually **untick `[ACTIVATE]` and tick it again before entering Master League again**. The manual reset is more predictable across the tested executables and patches. In v1.0.1 this reset also stops Automatic Player Development, removes its breakpoint, destroys its timer, and clears temporary development session state.

### Post-match and celebration protection

Live Player Editor writes are suspended during unsafe post-match transitions. Squad Ability Preset persistence also avoids following newly surfaced same-ID resolver records, preventing temporary championship/cutscene records from receiving normal squad ability writes.

### Match controls

**Add / Remove / Reset** Score Controls require a positive **Remaining Match Time (Raw)** value and are blocked while Raw=0, including stoppage-time and unsafe period transitions. Add Goal creates a scorer-only native goal record with no assistant; Remove/Reset synchronize the scoreboard, native goal history and the affected goal/assist match statistics.

### Values that can persist in the save

The following sections can change values that may be written into the Master League save:

- Finances
- Contract
- Salary
- Fitness
- Condition
- Shirt Number
- Player Development progression processed by PES6 after matches

Saving the game after entering an incorrect value may make that change persistent.

[↑ Back to top](#top)

---

<a id="v10-release"></a>
## v1.0 — First Public Release
This is the **first public stable release of PES6 Master League Editor**.

v1.0 combines Player Editor, squad recovery, squad presets, fitness overview, finances and match controls in one Cheat Engine table built around a shared verified Master League roster.

**Stable CT SHA-256**  
`ce9c35802e9a5aca2a29e2e273aa889eca41fcd30d75230796562fb3bfc44b1d`

[↑ Back to top](#top)

---

<a id="v101-release"></a>
## v1.0.1 — Player Development

v1.0.1 adds a new **Player Development** section built around PES6's native post-match development system.

### New — Automatic Player Development

Players who actually appear in a match can receive an age-based development EXP bonus:

- **Age 17–21 → +70 EXP**
- **Age 22–25 → +40 EXP**
- **Age 26–30 → +25 EXP**
- **Age 31+ → unchanged**

The bonus is added to the player's existing development EXP rather than replacing it. Starting XI players and substitutes who enter the match are eligible; unused substitutes remain untouched.

PES6 itself processes the resulting ability growth through its normal post-match development logic.

### Manual Development Presets

Optional development Actions are also available for the selected player or the entire squad:

- High Development — Next Match
- Peak Development — Next Match

### Session Integration

Disabling `[ACTIVATE]` now also integrates Player Development into the main editor reset:

- Stops the Automatic Development watcher
- Removes the development breakpoint
- Destroys the development timer
- Clears temporary development session state
- Resets development status rows

Player development already processed by PES6 is not reverted.

**Stable CT SHA-256**  
`488e5ab12777a24b8276aa1f712f0911f311a2a6660a96d5177fc1319d0efeb9`

[↑ Back to top](#top)

---

<a id="feature-summary"></a>
## Feature Summary
| Section | Includes |
|---|---|
| Player Selector | Single-player selection from the real Master League squad |
| Quick Player Actions | 99 Abilities, All Specials, Max Performance, Superstar |
| Player Profiles | 8 ready-made player profiles |
| Player Abilities | 26 editable abilities |
| Playable Positions | 12 positions |
| Special Abilities | 23 special abilities |
| Performance Settings | Foot, styles, position, condition, consistency, weak foot, etc. |
| Identity & Physical | Height, weight, skin, age, nationality, shirt number |
| Fitness & Recovery | Condition, stamina, fatigue + recovery Actions |
| Contract & Salary | Salary and contract years |
| Player Development | Age-based additive EXP for played players (+70/+40/+25) plus optional manual development presets |
| ML Settings | Difficulty, transfer frequency, Native match time, Custom 3/7/12 minute time |
| Squad Recovery | Full-squad fitness Actions |
| Squad Ability Presets | Complete Boost, Ultimate by Position, 99, specials, performance, +5/+10/+15/+20, session persistence |
| Fitness Overview | Player-by-player condition/stamina/fatigue overview |
| Infinite Stamina | Automatic side detection for your Master League team |
| Score Controls | Scorer-only native Home/Away Add Goal, display-only score status, crash-safe Remove/Reset with synchronized native goal history and goal/assist statistics |
| Match Clock | Linked Native/Custom duration controls + live Raw edit/freeze + End Current Period |
| Club Finances | Funds editing |
| Advanced Settings | Runtime and resolver options |
| Diagnostics | Technical read-only data |

[↑ Back to top](#top)

---

<a id="license"></a>
## License

This project is licensed under the **GNU General Public License v3.0 (GPL-3.0)**.

See the [LICENSE](https://github.com/jackcohle/PES6-Master-League-Editor/blob/main/LICENSE) file for details.

Copyright © 2026 **jackcohle**

[↑ Back to top](#top)

---

<a id="author"></a>
## Author
**jackcohle**
