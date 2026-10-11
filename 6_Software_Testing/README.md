# 6. Software Testing

This folder contains the software-testing and bug-tracking evidence for the **Remote Patient Vitals Alert & Monitoring App** (Problem Statement #16).

## Deliverables

| File | Description |
|---|---|
| [`BugReport.pdf`](BugReport.pdf) | Bug-tracking deliverable showing mock bugs organized into two categories (**Expedite** and **Everything else**) and a mock bug workflow. |
| [Lab 4 Repository](https://github.com/KadakWNL/16_hangman) | Completed individual Vibe Coding repository for Scenario 16 - Hangman Challenge. |

## Lab 4: Vibe Coding - Defect Resolution & Feature Engineering

As prescribed in the instructor submission guidelines (Practicing on Software Testing Tools using Vibe Coding to fix bugs, add features, and retest), the assignment was completed for the assigned individual game application repository.

- **Assigned Project:** Scenario 16 - Hangman Challenge
- **Repository Link:** [https://github.com/KadakWNL/16_hangman](https://github.com/KadakWNL/16_hangman)
- **Tooling Used:** Antigravity CLI (`agy`) with Gemini 3.8 Flash

### Tasks Implemented:
1. **Task 1 (Defect Fix):** Corrected guess-state handling in `guess()` so repeated incorrect guesses do not consume extra lives, and repeated correct guesses are not double-counted.
2. **Task 2 (Session Model):** Integrated `SessionStats` into `HangmanGame` to track rounds played, rounds won, win percentage, and best streak across the entire session.
3. **Task 3 (Difficulty & Scoring):** Implemented difficulty selection (Easy: 8 lives, Medium: 6 lives, Hard: 4 lives) with corresponding base points, dynamic round adaptation, and consistent hint penalties.
4. **Task 4 (Robust Input & Feedback):** Added clean input handling for whitespace, invalid letters, malformed commands (`/quit`, `/hint`, unknown commands), and user replay confirmations (`y`/`yes`/`n`/`no`).

### Submission Deliverables in Repo:
- `before_fix.mp4` - 8-second video demonstrating the defect prior to modification
- `after_fix.mp4` - 35-second video demonstrating all fixed functionality and new features
- `chat_history.pdf` - Complete LLM prompt and response transcript export without em dashes
