# 🎮 Game Glitch Investigator: The Impossible Guesser

## 🚨 The Situation

You asked an AI to build a simple "Number Guessing Game" using Streamlit.
It wrote the code, ran away, and now the game is unplayable. 

- You can't win.
- The hints lie to you.
- The secret number seems to have commitment issues.

## 🛠️ Setup

1. Install dependencies: `pip install -r requirements.txt`
2. Run the broken app: `python -m streamlit run app.py`

## 🕵️‍♂️ Your Mission

1. **Play the game.** Open the "Developer Debug Info" tab in the app to see the secret number. Try to win.
2. **Find the State Bug.** Why does the secret number change every time you click "Submit"? Ask ChatGPT: *"How do I keep a variable from resetting in Streamlit when I click a button?"*
3. **Fix the Logic.** The hints ("Higher/Lower") are wrong. Fix them.
4. **Refactor & Test.** - Move the logic into `logic_utils.py`.
   - Run `pytest` in your terminal.
   - Keep fixing until all tests pass!

## 📝 Document Your Experience

**Game Purpose:** A number-guessing game where the player tries to identify a secret number within a limited number of attempts, with directional hints after each guess. The twist: the original code was intentionally broken, and the goal is to find and fix the bugs.

**Bugs Found:**

| # | Bug | Location |
|---|-----|----------|
| 1 | Secret number regenerated on every Streamlit rerun — impossible to win | `app.py` (top-level `random.randint` call) |
| 2 | Directional hints were backwards ("Go Higher!" when guess was too high) | `app.py` game logic / `logic_utils.py` |
| 3 | `check_guess()` was an unimplemented stub returning `"Not Implemented"` | `logic_utils.py` |
| 4 | Hard difficulty range (1–50) was smaller/easier than Normal (1–100) | `logic_utils.py: get_range_for_difficulty` |
| 5 | Attempts counter initialized to 1 instead of 0 (first guess shown as attempt 2) | `app.py` session state init |
| 6 | Info prompt hardcoded "1 and 100" regardless of difficulty setting | `app.py` |
| 7 | "New Game" only reset attempts and secret — score, status, and history carried over | `app.py` |
| 8 | Invalid guesses (letters, empty input) counted as used attempts | `app.py` submit logic |
| 9 | Invalid guesses were appended to guess history | `app.py` submit logic |
| 10 | Secret was randomly cast to a string every even attempt, causing type-comparison failures | `app.py` |
| 11 | "Too High" outcome awarded +5 bonus points every other attempt (unintentional) | `logic_utils.py: update_score` |
| 12 | No validation that guess was within the selected difficulty range | `app.py` |

**Fixes Applied:**

- Moved secret number initialization into `st.session_state` with an `if "secret" not in st.session_state` guard so it only generates once per session.
- Corrected hint direction in `check_guess`: "Go LOWER!" when guess > secret, "Go HIGHER!" when guess < secret.
- Implemented `check_guess`, `parse_guess`, `update_score`, and `get_range_for_difficulty` in `logic_utils.py`.
- Fixed Hard difficulty range to `(1, 500)` to make it meaningfully harder.
- Initialized `attempts` to `0` and moved the increment inside the valid-guess branch.
- Updated the info prompt to dynamically display `{low}` and `{high}`.
- Reset all session state fields (`secret`, `attempts`, `score`, `status`, `history`) on New Game.
- Removed the type-casting glitch — comparisons now always use the integer secret.
- Fixed `update_score` to consistently deduct 5 on wrong guesses without bonus points.
- Added range validation to reject guesses outside the difficulty bounds.

## 📸 Demo

- [ ] [Insert a screenshot of your fixed, winning game here]

## 🚀 Stretch Features

- [ ] [If you choose to complete Challenge 4, insert a screenshot of your Enhanced Game UI here]
