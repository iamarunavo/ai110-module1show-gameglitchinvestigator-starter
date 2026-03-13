# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

- What did the game look like the first time you ran it?
  A dashboard.
- List at least two concrete bugs you noticed at the start  
  (for example: "the secret number kept changing" or "the hints were backwards").
  It says to guess a number between 1 and 100, I put 3000, and message back says to "GO HIGHER", which doesn't make sense.
  It sometimes records previous guesses again within the history when you actually put a different number.
  Number of attempts does not consisently go down when I am guessing numbers.
---

## 2. How did you use AI as a teammate?

- Which AI tools did you use on this project (for example: ChatGPT, Gemini, Copilot)?\
  Copilot
- Give one example of an AI suggestion that was correct (including what the AI suggested and how you verified the result).
  Copilot correctly identified that check_guess in app.py was casting the secret to a str on every even-numbered attempt, then falling back to lexicographic string comparison — which is why guessing 3000 returned "Go HIGHER!" instead of "Go LOWER!". Copilot suggested fixing this by always casting both guess and secret to int before comparing inside check_guess in logic_utils.py:38-43. I verified it by running the three pytest tests in test_game_logic.py — test_guess_too_high, test_guess_too_low, and test_winning_guess — all passed after the fix.
- Give one example of an AI suggestion that was incorrect or misleading (including what the AI suggested and how you verified the result).
  Copilot suggested clearing the text input after a submission by directly assigning st.session_state[guess_input_key] = "" at the end of the if submit: block in app.py. This caused a StreamlitAPIException at runtime because Streamlit forbids mutating a widget's state key after that widget has already been created in the same script run. I discovered it was wrong when the app crashed with a traceback pointing to that exact line. The correct fix was to set a flag instead, then clear the key on the next rerun before st.text_input is instantiated.
---

## 3. Debugging and testing your fixes

- How did you decide whether a bug was really fixed?
  Running several tests and testing the actual website myself.
- Describe at least one test you ran (manual or using pytest) and what it showed you about your code.
  I ran pytest tests/test_game_logic.py after refactoring check_guess into logic_utils.py. The tests in test_game_logic.py call check_guess and assert the return value equals "Win", "Too High", or "Too Low" directly — but check_guess returns a tuple like ("Too High", "📉 Go LOWER!"). This showed me the tests were written against a simpler interface contract, meaning either the tests needed updating to check result[0], or the function signature itself needed to match. It exposed a mismatch between the test expectations and the actual implementation that I would not have caught by just running the app.
- Did AI help you design or understand any tests? How?
  Yes. Copilot explained that the three existing tests each isolate a single code path, equal, greater, and less, which is the correct pattern for unit testing a pure function like check_guess.
---

## 4. What did you learn about Streamlit and state?

- In your own words, explain why the secret number kept changing in the original app.
  In the original app.py, random.randint(low, high) was called at the top level without verifying if a secret was already there, which meant every rerun generated a new random number. The session state guard at "secret" not in st.session_state in app.py:30-31 is what stops that: it only creates a new secret once per session, keeping the stored value intact during all the following reruns.
- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?
  In Streamlit, the script reruns from the top on every interaction, and st.session_state is used to store values so they persist instead of resetting each time.
- What change did you make that finally gave the game a stable secret number?

---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?
  - This could be a testing habit, a prompting strategy, or a way you used Git.
  Always running unit tests againsts logic.
- What is one thing you would do differently next time you work with AI on a coding task?
  Review the code more thoroughly before accepting it.
- In one or two sentences, describe how this project changed the way you think about AI generated code.
  AI can accurately figure out small bugs faster but it isn't always aware of constraits or specific requirements, and you need to provide that context, so its important to read and test the code.
