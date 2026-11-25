# StudyHub: Personalisation vs Predictability in an Adaptive Home Screen

**Module:** Intelligent Interactive Systems (School of Computer Science, University of Birmingham)  
**Student:** Shravani Jagtap | **Student ID:** [3090270]  
**Project:** Assignment 2 — *Personalisation vs Predictability: A Test Bed for Adaptive Interfaces* (builds on Assignment 1 literature review)  

---

## 1. Project overview

Adaptive interfaces can reduce interaction effort, yet silent layout changes often make them unpredictable and hard to control. Assignment 1 reviewed eight 2025–2026 studies on personalised systems (Adel et al. [1]; Algamdi [2]; Chai [3]; Du and Chen [4]; Eckl et al. [5]; Feng et al. [6]; Hofstede et al. [7]; Geddam et al. [8]). While those studies reported gains in efficiency, accessibility, learning, or accuracy, each left trust, understanding, predictability, usability, or user control untested. Figure 1 of Assignment 1 therefore proposed comparing three interface conditions.

**StudyHub** implements that comparison in `index.html` as a student home screen containing eight app tiles (`Timetable`, `Library`, `Notes`, `Messages`, `Grades`, `Maps`, `Cafe menu`, `Settings`) displayed across three interactive phone frames side by side:

* **Version A — Stable:** The tile layout never changes.
* **Version B — Automatic:** Once a tile is used twice, the app moves it to the top position automatically with no prompt or explanation.
* **Version C — Suggested:** Once a tile is used twice, the app displays a suggestion card explaining why and lets the student **Accept**, **Reject**, or later **Undo** the move.

---

## 2. Design rationale

Every mechanism in StudyHub maps directly to the Assignment 1 research gaps and HCI/AI design principles documented in `docs/report.md`:

* **Three-condition test bed (Assignment 1 Figure 1; Shneiderman, 2022):** Versions A, B, and C share identical tasks and rendering code, isolating visibility and human control (aiming for high automation with high human control in Version C).
* **Mixed-initiative interaction (Horvitz, 1999; Amershi et al., 2019 G9, G17):** In Version C, the system takes initiative after two uses of a tile (`uses[id] >= 2`), while the student retains final authority via **Accept**, **Reject**, and **Undo**, with control actions counted.
* **Bridging the gulf of evaluation (Norman, 2013; Amershi G11; Kulesza et al., 2013; Cowan, 2001):** Version B hides the cause of layout shifts, risking wrong taps and weakened agency (Haggard, 2017). Version C surfaces a single focused change and one plain-language reason (`Why: you have opened Notes 2 times`), respecting working-memory limits.
* **Inspectable, editable, forgettable control (Lecture 10; Amershi G14; Lecture 04):** Version C tags adapted tiles with a `Moved` badge, lists accepted changes under **Changes to your layout** with an **Undo** button, and remembers both **Reject** and **Undo** (`rej[id] = true`) so the system never re-suggests that tile.
* **Expectation alignment (Luger and Sellen, 2016; Amershi G1):** Version C explicitly states *"Only the order of the tiles changes. You can undo it later"*, preventing users from expecting hidden removals or content edits.
* **Predictability, wrong-tap, and tipping-point logging (Lecture 05, Lecture 09):** The prototype counts wrong taps (`misses`), marks whether a wrong tap occurred after a layout change, logs the first control action as a tipping-point marker, and collects 1–5 ratings on predictability, control, and trust.
* **Fixed rule instead of a generative model (Lecture 02, Lecture 08, Lecture 12):** Because identical LLM calls vary in wording and behaviour, StudyHub uses a deterministic rule to simulate user-preference inference so A, B, and C can be compared fairly.

---

## 3. How the prototype works

Each phone runs the same `makeApp(cond, root)` state machine inside `index.html`:

1. **Welcome & Start:** The tester reads a 3-step intro screen and presses **Start**.
2. **Six tap tasks:** The phone presents the fixed task sequence `Notes` → `Timetable` → `Notes` → `Library` → `Notes` → `Timetable` (`TASKS`), with a 6-segment progress bar.
3. **Adaptation trigger (after Task 3, the second `Notes` tap):**
   * **Version A (Stable):** `Notes` remains in its original slot; nothing shifts.
   * **Version B (Automatic):** `Notes` jumps silently to the first slot, shifting `Timetable` and `Library`. On Task 4 (`Open Library`), tapping the old `Library` position triggers a wrong-tap alert (`"Not that one. You tapped Timetable. Look for Library."`).
   * **Version C (Suggested):** An amber suggestion card asks *"Move Notes to the top of your screen"* with the reason and **Accept** / **Reject** buttons. Accepting moves `Notes` to slot 1, adds a `Moved` badge, and creates an **Undo** entry under **Changes to your layout**.
4. **End-of-session ratings:** After all six tasks, the user rates three statements from `1` (Disagree) to `5` (Agree):
   * *"I could predict where things would be"*
   * *"I felt in control of changes to the screen"*
   * *"I trusted that the changes were right for me"*
5. **Observer panels & session record:** Beneath each phone, **What to watch for**, **What just happened**, **Results** (wrong taps, layout changes, accepts, rejects, undos, ratings), and an expandable **Raw session record** update after every click. A **Restart all three** button resets all three phones together.

---

## 4. Implementation status

| Component / Feature | Status | Notes |
| :--- | :--- | :--- |
| Three-version comparison UI (A, B, C) | Implemented | Shared `makeApp` engine and layout renderer in `index.html`; comparison table at top; responsive A/B/C tab bar on narrow screens (`<= 1099px`). |
| Preference inference rule | **Simulated** | Deterministic rule (`uses[id] >= 2 && !moved(id) && !rej[id]`) representing Lecture 02 sense–interpret–act inference with no external AI API call. |
| Version C suggestion & controls | Implemented | Suggestion card with plain-language reason, **Accept**, **Reject**, `Moved` tile badge, and **Changes to your layout** **Undo** list. |
| Task flow, wrong-tap tracking & ratings | Implemented | 6 fixed tap tasks, wrong-tap feedback and post-change flagging, tipping-point marker, and 1–5 Likert ratings for predictability, control, and trust. |
| Observer panels & raw session log | Implemented | Live **What just happened** feed, **Results** summary, and timestamped **Raw session record**. |
| Content-adaptive quiz | Planned | Deferred from an earlier prototype in favour of the clearer tile-order layout task (planned later stage, cf. Chai [3]). |
| LLM-generated reason text | Planned | Future language model would only phrase the `"why"` sentence in constrained format; application code still controls layout changes. |
| Saved preferences with expiry | Planned | Planned to address the stale-note problem (Lecture 10), where moved tiles currently stay at the top indefinitely unless undone. |
| Empirical participant study | Planned | Prototype and instrumentation are built; no participant data has been collected yet. |

---

## 5. Technology stack

* **Prototype (`index.html`):** Single self-contained HTML5 file (236 lines) with embedded CSS3 and vanilla JavaScript plus inline SVG icons. Zero external libraries, zero build step, works fully offline, and requires no API keys.
* **System diagram (`docs/system_diagram.svg`, `.png`):** Vector flowchart and PNG export matching the implemented `index.html` states and planned extensions.
* **Documentation & report (`docs/`):** Written in Markdown (`docs/report.md`) and formatted into `docs/Assignment2_Project_Report_FINAL.docx` and `.pdf` (10 pages; 1,291 words total: 228-word abstract + 1,063-word main report).
* **Screenshots (`screenshots/`):** Rendered directly from `index.html` for the report figures and storyboards.
* **Planned / undecided:** Specific language model provider for future reason-text generation (none integrated yet).

---

## 6. Running or inspecting the work

No installation, server, or build step is required. Open `index.html` directly in any modern web browser:

```bash
open index.html
```

* **Desktop view:** All three phone versions (**A. Stable**, **B. Automatic**, **C. Suggested**) appear side by side below the comparison table, with observer panels underneath each phone.
* **Narrow-screen view (`<= 1099px`):** Use the sticky **A. Stable / B. Automatic / C. Suggested** tabs at the top to switch between phones.
* **Resetting:** Click **Restart all three** in the header banner to clear all session logs, ratings, and tile positions.

---

## 7. Development record

*(Note: this folder is a standalone repository snapshot without local `.git` commit history; stages below trace the design evolution documented in `docs/report.md`.)*

* **Stage 1 — Research framing:** Defined the three-way comparison (Stable, Automatic, Suggested) from the Assignment 1 literature review gaps and Figure 1.
* **Stage 2 — Early quiz prototype pivoted to home screen:** Built an initial content-adaptive quiz, then set it aside because users found it harder to follow and it did not cleanly match Assignment 1's layout conditions; simplified to the 8-tile StudyHub home screen.
* **Stage 3 — Version C control mechanisms:** Added the suggestion card with plain-language explanation, **Accept** / **Reject** actions, `Moved` badge, and **Changes to your layout** **Undo** control.
* **Stage 4 — Measurement & logging:** Added the 6-task sequence, wrong-tap detection, tipping-point marker, 1–5 predictability/control/trust ratings, and timestamped session record.
* **Stage 5 — Clarity revisions from classmate feedback:** Because classmates did not understand the initial home-screen version, added the 3-step welcome intro, task progress bar, plainer suggestion wording, top comparison table, side-by-side layout, and observer explanation boxes.

---

## 8. AI assistance

Claude (Anthropic) was used as a drafting assistant for the prototype code, system diagram, storyboards, and report text. All research framing, interface decisions, classmate checks, and final verification were directed by Shravani Jagtap, and no API keys are used or stored in the project.

| Focus area | What AI was used for | Human direction and verification |
| :--- | :--- | :--- |
| **Concept & literature mapping** | Structuring how Assignment 1 gaps map to prototype features | Verified each Assignment 1 gap (`[1]`–`[8]`, Figure 1) maps accurately to an implemented or planned StudyHub component. |
| **Interface prototype** | Drafting the initial quiz and the 3-phone `index.html` layout | Rejected the early adaptive quiz as confusing; directed the shift to three side-by-side home screens with app-style visual design. |
| **Clarity & usability** | Revising copy, welcome intro, and observer panels | Responded to informal classmate confusion on the early home screen by requiring plainer wording, a comparison table, and live *"What just happened"* explanations. |
| **System diagram** | Drafting the end-to-end flowchart | Requested that all three conditions (A, B, C) and simulated vs. planned components be shown together after an earlier Version-C-only draft proved confusing. |
| **Project report** | Drafting prose and checking word counts | Edited wording, checked all 8 Assignment 1 and HCI citations, and ensured the 1,291-word report (`docs/report.md`) accurately reflects implemented vs. planned scope. |

---

## 9. Additional files

* `index.html` — Complete interactive StudyHub prototype (Versions A, B, and C).
* `README.html` — Pandoc HTML export of an earlier draft of `README.md`.
* `docs/report.md` — Markdown source of the Assignment 2 project report (1,291 words).
* `docs/Assignment2_Project_Report_FINAL.docx` & `docs/Assignment2_Project_Report_FINAL.pdf` — Final formatted 10-page submission report with embedded system diagram, storyboards, and references.
* `docs/system_diagram.svg` & `docs/system_diagram.png` — End-to-end system architecture and flow diagram.
* `screenshots/versions_at_a_glance.png` — Comparison table of Versions A, B, and C.
* `screenshots/three_versions_overview.png` — Capture of Version A with its header, phone frame, and observer panels.
* `screenshots/observer_view_B.png` — Supplemental background capture in the screenshots folder.
* `screenshots/storyboard_1_stable.png` — Storyboard 1: Version A welcome screen, Task 1, and Task 4 unchanged layout.
* `screenshots/storyboard_2_auto.png` — Storyboard 2: Version B Task 1, silent move of `Notes` to top before Task 4, and resulting wrong tap on `Timetable`.
* `screenshots/storyboard_3_suggested.png` — Storyboard 3: Version C suggestion card, post-Accept `Moved` state with **Undo**, post-Undo restored layout, and end-of-session 1–5 ratings.
