# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

When I first ran the game, the secret number changed every single time I clicked "Submit Guess" — meaning it was nearly impossible to ever win because the target kept moving. The directional hints were also backwards: when my guess was too high, the game told me to "Go Higher!", and when my guess was too low, it said "Go Lower!" — the exact opposite of what was true. On top of that, the `check_guess` function in `logic_utils.py` was a stub that always returned `"Not Implemented"`, so the core game logic did nothing at all. After the initial refactor, several subtler bugs remained: the Hard difficulty had a smaller range (1–50) than Normal (1–100), invalid guesses still counted as attempts, and starting a new game didn't fully reset the score, status, or history.

**Concrete bugs noticed at the start:**
- The secret number regenerated on every Streamlit rerun because it was assigned outside `st.session_state`.
- The hints were backwards — "Go HIGHER!" when the guess was too high, "Go LOWER!" when too low.

---

## 2. How did you use AI as a teammate?

I used Claude (Claude Code) as my primary AI tool throughout this project. I described the bugs I was seeing and asked Claude to help me find the root causes and suggest fixes, then I reviewed each suggestion before applying it.

**Correct AI suggestion:** Claude correctly identified that the secret number kept changing because Streamlit re-executes the entire script on every interaction, so `random.randint(1, 100)` ran fresh each time. It suggested wrapping the initialization in `if "secret" not in st.session_state`, which I verified by running the game and confirming the number in the debug panel stayed constant across multiple guesses.

**Incorrect or misleading suggestion:** In the refactored code, Claude left in logic that alternated the type of the secret number every other attempt (`secret = str(st.session_state.secret)` on even attempts). This was meant to simulate a "glitch" but caused broken comparisons and unpredictable behavior. I caught it by reading the diff carefully and removed the type-switching logic so comparisons always used the integer value.

---

## 3. Debugging and testing your fixes

I decided a bug was fixed when I could reproduce the original broken behavior, apply the fix, and then confirm the broken behavior no longer occurred. For the hint direction bug, I deliberately guessed a number I knew was too high, confirmed the wrong hint appeared before my fix, then verified the correct "Go LOWER!" hint appeared after.

I ran `pytest` after implementing `check_guess` and `parse_guess` in `logic_utils.py`. The tests confirmed that the function correctly returned `"Win"`, `"Too High"`, and `"Too Low"` for the right inputs, and that `parse_guess` rejected non-numeric strings and empty input gracefully.

Claude helped me understand one test case I hadn't considered: what happens when comparing an integer guess to a string secret (the type-mismatch glitch). It explained that Python raises a `TypeError` on `>` comparisons between `int` and `str` in Python 3, which helped me understand why the `try/except TypeError` block existed — and why removing the type-switching logic was the right fix.

---

## 4. What did you learn about Streamlit and state?

The secret number kept changing in the original app because Streamlit works by re-running the entire Python script from top to bottom every time a user interacts with the page (clicks a button, types in a field, etc.). Since `secret_number = random.randint(1, 100)` was at the top level of the script, it executed fresh on every rerun and produced a new random number each time.

Streamlit "reruns" are like refreshing a web page — everything resets unless you explicitly save it. `st.session_state` is like a small backpack the app carries between reruns: anything you put in it survives the next rerun. You can think of it like a global dictionary that Streamlit keeps alive for the duration of a user's session.

The fix that gave the game a stable secret number was wrapping the initialization in a guard: `if "secret" not in st.session_state: st.session_state.secret = random.randint(low, high)`. This means the random number is only generated once — the very first time the app loads — and every subsequent rerun just reads the stored value.

---

## 5. Looking ahead: your developer habits

One habit I want to reuse is reading diffs carefully before accepting AI-generated changes. On this project, Claude sometimes left in "intentional glitch" code from the starter that wasn't actually meant to be kept, and only reading the diff line-by-line helped me catch it. This taught me to treat AI output as a starting point for review, not a final answer.

Next time I work with AI on a coding task, I would verify each fix in isolation rather than applying several suggestions at once — it became hard to tell which change fixed which bug when I applied multiple edits in one go. I would also ask the AI to explain *why* a fix works, not just *what* to change, so I can catch cases where the explanation doesn't match the actual code behavior.

This project changed the way I think about AI-generated code: it can look completely correct at a glance while hiding logic bugs that only show up when you run the app or read the diff carefully. AI is a powerful first draft, but human review is what makes it reliable.
