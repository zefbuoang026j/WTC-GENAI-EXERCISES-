PYTHON AUTOMATION LOGGING



You’re on the right track—automation + logging is a powerful combo. I’ll break it down in a practical way so you can actually use it, not just understand it.


---

1. What is logging in automation (simple explanation)

When you automate something, your script runs without you watching it.

👉 Logging = your script talking back to you about what it’s doing.

Instead of print(), you use Python’s built-in logging module to:

Track events

Record errors

Monitor behavior over time


Basic Example

import logging

# Configure logging
logging.basicConfig(
    filename="app.log",
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s"
)

logging.info("Program started")

def divide(a, b):
    try:
        result = a / b
        logging.info(f"Divided {a} by {b}")
        return result
    except Exception as e:
        logging.error(f"Error dividing numbers: {e}")

divide(10, 2)
divide(5, 0)

logging.info("Program finished")

What happens:

Logs go into app.log

You get timestamps + errors

You can debug without watching the script live



---

2. 3 Practical Use Cases (real-world)

🔹 1. Website Monitoring Bot

Check if a website is up every 5 minutes.

import requests
import logging
import time

logging.basicConfig(filename="monitor.log", level=logging.INFO)

while True:
    try:
        response = requests.get("https://example.com")
        if response.status_code == 200:
            logging.info("Website is UP")
        else:
            logging.warning("Website issue detected")
    except Exception as e:
        logging.error(f"Connection failed: {e}")

    time.sleep(300)  # 5 minutes

👉 Useful for:

Freelance monitoring tools

SaaS uptime trackers



---

🔹 2. File Processing Automation

Process files and log what happened.

import logging
import os

logging.basicConfig(filename="files.log", level=logging.INFO)

for file in os.listdir("data"):
    if file.endswith(".txt"):
        logging.info(f"Processing {file}")
    else:
        logging.warning(f"Skipped {file}")

👉 Useful for:

Data pipelines

Bulk file automation



---

🔹 3. Scheduled Task Automation (your idea 👇)

You mentioned:

> “monitoring automation that runs at a certain time”



Yes — combine:

logging

scheduling


import logging
import time

logging.basicConfig(filename="task.log", level=logging.INFO)

def job():
    logging.info("Task executed")

while True:
    current_time = time.strftime("%H:%M")

    if current_time == "14:00":
        job()
        time.sleep(60)  # prevent duplicate runs

    time.sleep(10)

👉 This is basic scheduling (you can upgrade with schedule or cron later)


---

3. Small Project Idea (VERY practical)

🚀 Build: “System Monitoring Automation Tool”

Features:

Runs every X minutes

Checks:

Website status

CPU usage (optional)

File changes


Logs everything


Example structure:

monitor_project/
│── monitor.py
│── logger.py
│── checks/
│     ├── website.py
│     ├── system.py
│── logs/
      └── monitor.log

Core idea:

import logging
import time

logging.basicConfig(filename="logs/monitor.log", level=logging.INFO)

def check_system():
    logging.info("Checking system...")

def check_website():
    logging.info("Checking website...")

while True:
    check_system()
    check_website()
    time.sleep(60)

👉 This is real SaaS foundation You can later turn it into:

Dashboard

Alerts (email/SMS)

API



---

4. Common Mistakes (avoid these)

❌ 1. Using print() instead of logging

Prints disappear

Logs persist


✔ Always use logging for automation


---

❌ 2. Not setting log levels properly

Levels:

DEBUG → detailed info

INFO → normal actions

WARNING → something odd

ERROR → something broke


👉 Beginners log everything as INFO — bad for debugging


---

❌ 3. No timestamps

Bad:

logging.info("Task done")

Good:

format="%(asctime)s - %(message)s"


---

❌ 4. Infinite loops without control

while True:

👉 Can:

spam logs

crash system


✔ Add delays:

time.sleep(60)


---

❌ 5. Logs getting too big

If your automation runs for days:

👉 Your log file will explode

✔ Solution:

from logging.handlers import RotatingFileHandler


---

❌ 6. Not logging errors

Always log exceptions:

try:
    risky()
except Exception as e:
    logging.error(e)


---

🔥 Key Insight (important)

What you described:

> “automation that runs at a certain time and executes functions”



That is basically:

👉 Automation = Scheduler + Logic + Logging

Scheduler → when it runs

Logic → what it does

Logging → what happened



---

If you want next step

I can help you turn this into:

✅ A real **freelance product**

Monitoring dashboard

Alerts (email/WhatsApp)

Web interface (React + API)


Just say: 👉 “build production version”

And we’ll level this into something you can actually sell.



CODE REVIEW Python



You’ve got a solid beginner script here—it works, it’s readable, and it handles basic validation. But if you’re aiming to grow into a professional Python developer, there are a few important improvements to make.


---

1. Code smells / quality issues

a) Everything is in global scope
All logic runs at the top level. This makes the code harder to reuse, test, or extend.

b) Weak input validation
isdigit() only accepts positive integers (no negatives, no decimals). Also, marks like 150 would still pass.

c) Magic numbers
The number 5 (subjects) is hardcoded with no explanation.

d) Repetitive / tightly coupled logic
Input, calculation, and output are all mixed together instead of separated into functions.

e) No error handling
If something unexpected happens, the program could crash or behave incorrectly.

f) Naming could be clearer
Variables like m, count, and total are okay but not very descriptive in a larger system.


---

2. Suggested improvements (clean version)

Here’s a cleaner, more structured version:

def get_marks(num_subjects):
    marks = []
    
    for i in range(num_subjects):
        while True:
            user_input = input(f"Enter mark {i + 1}: ")
            
            try:
                mark = int(user_input)
                
                if 0 <= mark <= 100:
                    marks.append(mark)
                    break
                else:
                    print("Mark must be between 0 and 100.")
            
            except ValueError:
                print("Invalid input. Please enter a number.")
    
    return marks


def calculate_average(marks):
    return sum(marks) / len(marks)


def get_grade(avg):
    if avg >= 75:
        return "Distinction"
    elif avg >= 60:
        return "Merit"
    elif avg >= 50:
        return "Pass"
    return "Fail"


def main():
    print("Student Grade Calculator")
    
    name = input("Enter student name: ")
    num_subjects = 5

    marks = get_marks(num_subjects)
    average = calculate_average(marks)
    grade = get_grade(average)

    print(f"\nName: {name}")
    print(f"Marks: {marks}")
    print(f"Average: {average:.2f}")
    print(f"Grade: {grade}")
    print("Done")


if __name__ == "__main__":
    main()


---

3. Why these improvements matter (in simple terms)

Functions = modular code
Breaking code into functions makes it easier to reuse and test. For example, you can reuse calculate_average() anywhere.

Validation with try/except
Real users will break your app. This prevents crashes and makes your program more professional.

Separation of concerns
Input, logic, and output are separated. This is how real-world systems are built (especially in APIs and SaaS apps).

Readable naming
Code is read more than it’s written. Clear names reduce confusion when your codebase grows.

Scalability
Today it's 5 subjects. Tomorrow it could be dynamic, from a database or API. This structure supports that.



---

4. Ratings

Readability: 6/10

Easy to follow for small script

But lacks structure and clear separation


Performance: 8/10

Totally fine for this use case

No major inefficiencies


Maintainability: 4/10

Hard to extend (e.g., adding more features)

Everything being in one block is the main issue



---

Final takeaway

Your code is good for a beginner, but not yet production-ready.

The key upgrade you need is: 👉 thinking in structure (functions, separation, validation), not just “making it work”


---

If you want, I can help you turn this into a mini real-world feature (like saving grades to a file or building a simple CLI app).



CODE PYTHON REFINEMENT


You’re actually pretty close already—this is clean, readable, and structured well. The improvements here are more about Python style and small idioms rather than big rewrites.

Let’s tighten it up.


---

🔧 Key Improvements (Idiomatic Python)

1. Prefer early validation + fewer nested branches

Instead of checking .isdigit() and continuing, you can lean on try/except—this is the Pythonic way.

Why?
Python follows EAFP (“Easier to Ask Forgiveness than Permission”) instead of LBYL (“Look Before You Leap”).


---

2. Use constants for magic numbers

MAX_ATTEMPTS = 7
LOW, HIGH = 1, 100

Why?
Improves readability and makes changes easier later.


---

3. Use for instead of manual counter

You’re manually managing attempts, but Python gives you this for free.


---

4. Avoid repeated calculations

You calculate attempts left manually—can simplify with loop index.


---

5. Slightly cleaner comparisons

You don’t need elif guess > number—just use else.


---

6. Small UX improvements

Strip input

Handle uppercase/lowercase consistently



---

📊 Side-by-Side Comparison

🧱 Your Version

import random

def play():
    number = random.randint(1, 100)
    attempts = 0
    max_attempts = 7

    print("Guess a number between 1 and 100")

    while attempts < max_attempts:
        user_input = input("Enter guess: ")

        if not user_input.isdigit():
            print("Invalid input. Try again.\n")
            continue

        guess = int(user_input)
        attempts += 1

        if guess < number:
            print("Too low!")
        elif guess > number:
            print("Too high!")
        else:
            print(f"Correct! Attempts: {attempts}")
            return

        print(f"Attempts left: {max_attempts - attempts}\n")

    print(f"Game over! The number was {number}")


---

🚀 Improved Idiomatic Version

import random

LOW, HIGH = 1, 100
MAX_ATTEMPTS = 7


def play():
    number = random.randint(LOW, HIGH)
    print(f"Guess a number between {LOW} and {HIGH}")

    for attempt in range(1, MAX_ATTEMPTS + 1):
        try:
            guess = int(input("Enter guess: ").strip())
        except ValueError:
            print("Invalid input. Enter a number.\n")
            continue

        if guess < number:
            print("Too low!")
        elif guess > number:
            print("Too high!")
        else:
            print(f"Correct! Attempts: {attempt}")
            return

        remaining = MAX_ATTEMPTS - attempt
        print(f"Attempts left: {remaining}\n")

    print(f"Game over! The number was {number}")


def main():
    while True:
        play()
        if input("Play again? (y/n): ").strip().lower() != "y":
            print("Goodbye!")
            break
        print()


if __name__ == "__main__":
    main()


---

🧠 Python Features You’re Now Using Better

✅ for loops over manual counters

More Pythonic than managing attempts += 1.


---

✅ Exception handling (try/except)

This is core Python philosophy, not just error handling.


---

✅ Constants (UPPER_CASE)

Standard Python convention.


---

✅ f-strings properly used

You already did this right 👍


---

⚡ Extra (Optional Upgrade Ideas)

If you want to push this toward “real-world / professional”:

1. Add input bounds checking

if not LOW <= guess <= HIGH:
    print(f"Enter a number between {LOW} and {HIGH}")
    continue


---

2. Return result instead of printing

Makes it testable:

return True  # win
return False # loss


---

3. Separate logic from I/O

This is how real apps are structured:

game logic

input/output layer



---

🧩 Bottom Line

Your code is already:

readable ✅

structured ✅

beginner-to-intermediate solid ✅


The changes above make it:

more Pythonic

easier to maintain

closer to production-style thinking



---

If you want, next step I can show you how to turn this into: 👉 a CLI app with arguments
👉 or a small web app (Flask)
👉 or even part of a SaaS-style project (login + scores leaderboard)

Just tell me 👍



CODE REVIEW AND FEEDBACK


Automation Logging Exercise Submission

1. Small Code Sample Using Logging

import logging
import time

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
    filename="automation.log"
)


def monitor_system():
    logging.info("Monitoring automation started")

    for attempt in range(1, 4):
        logging.info(f"Checking system status... Attempt {attempt}")
        time.sleep(1)

        # Simulated condition
        if attempt == 3:
            logging.warning("High CPU usage detected")
            break

    logging.info("Monitoring automation finished")


monitor_system()


---

2. AI Feedback Reflection

Feedback Received

After reviewing the implementation, the following improvements were identified:

1. Add exception handling to prevent crashes.


2. Use different log levels correctly (INFO, WARNING, ERROR).


3. Avoid hardcoding values and make configurations reusable.


4. Improve readability with smaller functions.


5. Add timestamps and structured log formatting.




---

3. Checklist of Issues Identified

Code Review Checklist

[ ] Are logs added for important actions?

[ ] Are correct log levels used?

[ ] Is exception handling implemented?

[ ] Are messages clear and descriptive?

[ ] Are repeated values configurable?

[ ] Is sensitive information avoided in logs?

[ ] Is the code modular and readable?

[ ] Are unnecessary print statements removed?

[ ] Are logs stored properly in a file?

[ ] Is the automation easy to debug using logs?


Future Usage

This checklist will be used during future code reviews to improve debugging, readability, maintainability, and automation reliability.


---

4. Improved Version of the Code

import logging
import time

# Logging configuration
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
    filename="automation.log"
)

MAX_ATTEMPTS = 3


def check_system(attempt):
    logging.info(f"Running system check attempt {attempt}")

    if attempt == MAX_ATTEMPTS:
        logging.warning("Potential issue detected: High CPU usage")
        return False

    return True


def monitor_system():
    logging.info("Automation started")

    try:
        for attempt in range(1, MAX_ATTEMPTS + 1):
            status = check_system(attempt)
            time.sleep(1)

            if not status:
                break

        logging.info("Automation completed successfully")

    except Exception as error:
        logging.error(f"Automation failed: {error}")


monitor_system()


---

5. Three Key Learnings

Learning 1: Logging Improves Debugging

I learned that logging helps track program execution and makes it easier to identify where problems occur in automation systems.

Learning 2: Log Levels Matter

I learned that different log levels such as INFO, WARNING, and ERROR help categorize system events and make debugging more organized.

Learning 3: Code Reviews Improve Quality

I learned that reviewing code using a checklist helps identify issues like poor structure, missing exception handling, and unclear logging messages before deployment.


---

6. Reflection

Through this exercise, I improved my understanding of automation logging in Python. Initially, I viewed logging as similar to print statements, but I learned that logging provides structured tracking of system events and errors. I also learned the importance of using proper log levels, exception handling, and reusable configurations.

The feedback process helped me recognize weaknesses in my first implementation and improve the code structure. The checklist created from the review process will help me during future projects to write cleaner and more maintainable automation systems.

