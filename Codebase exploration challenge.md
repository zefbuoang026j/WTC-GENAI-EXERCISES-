TASK PRIORITY DEEP DIVE

Good instinct — but that answer is a bit too quick. Let’s pressure-test it like a senior dev would.

You said TaskManager 👍
Now let’s make sure that choice actually holds up.


---

Let’s examine your decision

Instead of confirming it, I want you to justify it.

1. Responsibility boundaries

What is calculate_task_score currently responsible for?

Does it change task data, or just evaluate it?


👉 If you added “mark as abandoned” there, what side-effect would you be introducing?


---

2. Think about the Task model

The rule says: “tasks should be marked as abandoned”


👉 Where is task.status actually defined and stored?

In which file/class?

Who is currently allowed to change it?



---

3. Why TaskManager might make sense (or not)

You chose TaskManager — now challenge yourself:

Does TaskManager already contain logic that updates task status?

Or does it mostly coordinate operations (create, update, fetch)?


👉 Can you find a method in TaskManager that modifies a task and share it?


---

4. When should this rule run?

This is the subtle part most people miss.

Should the rule run:

when a task is viewed?

when tasks are loaded from storage?

when a task is updated?

or on some scheduled process?



👉 Your answer here might change where the code belongs.


---

5. Hidden coupling check

If you implement this in TaskManager:

Will every place that uses tasks go through TaskManager?

Or could some parts of the code bypass it and break your rule?



---

Your next step

Don’t change your answer yet — refine it.

👉 Answer this:

> If a task becomes 8 days overdue while the app is closed, when exactly should it become “abandoned” the next time the system runs?



That answer will tell you whether TaskManager is truly the right place — or just the most obvious one.

Good — that “no” tells me you’re starting to question your first instinct, which is exactly what we want.

Let’s tighten the thinking instead of jumping to a new answer.


---

Re-evaluate your assumption

If TaskManager is not the right place, then ask yourself:

Is this rule about:

calculating a value (like score)?

or changing the state of a task (like status)?



👉 Which one does “mark as abandoned” fall under?


---

Now connect it to your code

Look back at what you saw:

calculate_task_score(...) → returns a number

sort_tasks_by_importance(...) → orders tasks

get_top_priority_tasks(...) → slices a list


👉 Do any of these functions modify the task itself?


---

Push deeper

Think about the Task model you saw:

It has:

status

priority

due_date

timestamps



👉 If a rule directly changes status, which layer should own that rule:

the place that stores behavior of a task

or the place that orchestrates tasks



---

Critical scenario (this is the key insight)

Imagine this:

1. App is closed for 10 days


2. A task becomes overdue during that time


3. You open the app again



👉 Question:

What piece of code runs first that even knows the task is overdue?


Is it:

CLI?

TaskManager?

or something triggered when tasks are loaded or accessed?



---

One focused task for you

Go back to your code and find:

Where tasks are loaded from storage

Where they are first processed after loading


👉 Paste that snippet.


---

Final guiding question 💡

If you wanted this rule to be impossible to bypass anywhere in the system, would you:

attach it to a function that must always run (like scoring or loading),

or place it in a layer that might be skipped?


Your answer here will lead you to the right architectural choice.


Nice — this is exactly the piece we needed. Now let’s reason through it together instead of jumping to conclusions.


---

What you’ve spotted correctly ✅

This is clearly the storage layer (TaskStorage)

Tasks are being loaded from a file (tasks.json) into memory

The load() method is called immediately in __init__, so this runs as soon as storage is created


That last point is very important — you’ve just found one of the earliest execution points in the system.


---

Now let’s dig deeper with targeted questions

1. Where is the “moment of truth”?

Look at this line:

self.load()

This runs when storage is initialized


👉 Question:

After load() finishes, where do these tasks go next?

Which part of the system uses self.tasks after this?


Can you trace:

TaskStorage → ? → ? → user sees tasks


---

2. Does this layer change task meaning?

Inside load():

self.tasks[task.id] = task

👉 Question:

Is this layer just reconstructing objects, or changing their meaning/state?


If you added:

> “if overdue > 7 days → mark abandoned”



👉 Would that be:

restoring data?

or applying business rules?



---

3. Timing of your rule (this is the key insight)

You now know:

load() runs once when the app starts


👉 Question:

What if a task becomes overdue while the app is already running?


Would your rule:

still trigger?

or be missed?



---

4. Compare with scoring function

Think back to:

calculate_task_score(task)

👉 Question:

When is this function executed?

once?

or every time tasks are sorted/viewed?



👉 Which is more “always up-to-date”:

load-time logic

or evaluation-time logic?



---

5. Separation of concerns (subtle but important)

Right now:

TaskStorage → handles persistence

task_priority.py → handles evaluation (scoring & sorting)


👉 Question:

If you mix “status mutation” into storage, what happens if later you switch from JSON → database?


Would your business rule:

stay reusable?

or get tied to storage implementation?



---

One focused task for you 🔍

Find where TaskStorage is used — likely inside TaskManager.

👉 Paste:

The code where TaskManager initializes or uses TaskStorage

And where it returns tasks to the CLI



---

Final practical question 💡

If you want this rule to:

always be correct

never be skipped

and not depend on when the app starts


👉 Would you:

store the “abandoned” status permanently,

or derive it dynamically whenever tasks are evaluated?


Your answer to that will determine whether you:

mutate task.status

or extend the scoring / filtering logic instead


Take your time — this is exactly the kind of thinking that levels you up.

REFLECTION DOCUMENTATION


Codebase Exploration Challenge – Reflection Documentation

Introduction

This reflection documents my learning journey during the Codebase Exploration Challenge. Throughout the exercises, I focused on understanding unfamiliar codebases, tracing feature implementation, analyzing data flow, understanding domain models, and improving my debugging and reasoning skills.

Instead of only looking for direct answers, I used questioning and hypothesis-driven thinking to build my understanding step by step. This process helped me improve my ability to read code, reason about application behavior, and identify how different parts of a system interact.


---

Part 1: Understanding Project Structure

My Initial Understanding

At the beginning of the challenge, I focused on understanding where different responsibilities lived inside the project. I explored how the application was structured and how features were grouped.

I learned that understanding a codebase is not about memorizing every file, but about identifying:

Entry points

Feature boundaries

Data flow

Domain models

Relationships between components


I also learned that asking the right questions is more important than immediately searching for answers.

Key Learnings

A project structure often reflects the responsibilities of the application.

Features are usually separated into layers such as UI, business logic, state management, and persistence.

Understanding naming conventions helps trace functionality faster.

Looking for keywords related to a feature is an effective starting point.


Reflection

Initially, large codebases felt overwhelming because I tried to understand everything at once. Through this exercise, I learned to narrow my focus to a single feature or workflow and follow it step by step.

I also learned that building mental models while exploring code is extremely important. Instead of passively reading files, I started forming hypotheses about what the code was doing before confirming them.


---

Part 2: Understanding Feature Implementation

Task Priority Feature

One of the features I explored was the task priority system.

My Understanding

My understanding was that the feature calculated task weight using priority weight values.

I discovered that:

Tasks are assigned different priority levels.

Priority values influence ordering and behavior.

Overdue tasks dominate sorting because their urgency is higher.

Descending order was used during prioritization.


What I Learned

While exploring this feature, I learned:

Sorting logic is often tied to business rules.

Priority systems are not only visual but affect application behavior.

Business logic can sometimes be hidden inside utility or helper functions.

Following data transformations step by step helps reveal the real behavior of the system.


Reflection

Before this exercise, I mostly looked at code line by line. During this challenge, I started tracing logic through the entire workflow instead.

I learned that understanding “why” a calculation exists is more important than only understanding “how” it is implemented.


---

Part 3: Understanding Data Flow and State Changes

Task Completion Flow

I explored how task completion changes application state.

My Investigation Included

Identifying components involved when marking a task as complete.

Finding code responsible for state changes.

Understanding how updates are persisted.

Identifying possible failure points.


My Understanding

I learned that completing a task affects multiple parts of the system:

UI updates

State changes

Data persistence

Synchronization logic


I also identified that failures can happen when:

State updates are incomplete

Persistence fails

Synchronization conflicts occur

UI does not reflect updated state correctly


Reflection

This exercise improved my understanding of how applications manage state.

Previously, I mostly focused on visible behavior in the UI. After tracing task completion logic, I learned that many hidden operations happen behind the scenes whenever a user interacts with the application.

I also realized how important it is to track data movement through the system instead of only reading isolated functions.


---

Part 4: Understanding Complex Logic and Merge Behavior

Merge Logic Exploration

I explored complex logic involving remote tasks and local tasks.

My Understanding

My understanding was that the system:

Creates separate lists for remote tasks and local tasks.

Merges them together.

Resolves conflicts between local and remote versions.

Updates task statuses after merging.


The most important part of this logic was conflict resolution.

What I Learned

Through this exploration, I learned:

Complex conditionals often represent business rules.

Merge systems must carefully handle conflicting updates.

State synchronization is difficult because multiple sources of truth exist.

Understanding control flow requires tracing conditions one step at a time.


Reflection

At first, the nested logic felt confusing and difficult to follow.

Using questioning techniques helped me simplify the logic into smaller mental steps:

1. Identify inputs.


2. Track transformations.


3. Observe conditions.


4. Understand output.



This approach helped me reduce confusion and understand the intent behind the code.


---

Part 5: Algorithm Analysis and Debugging

Merge Sort Bug Investigation

I analyzed a buggy merge sort implementation.

My Learning Process

I learned that my mental model for merge sort should be:

1. Split arrays recursively.


2. Sort each half.


3. Merge sorted halves.



I also learned structured debugging steps:

Step 1: Define invariants.

Step 2: Compare behavior with a trusted baseline.

Step 3: Use property testing ideas.


Reflection

This exercise changed how I think about debugging.

Previously, I focused mostly on fixing visible errors. Through this challenge, I learned that debugging should involve understanding:

Expected behavior

Invariants

Data transformations

Edge cases


I also learned that strong mental models make debugging easier.


---

Part 6: Performance Optimization Learning

Performance Analysis

I analyzed inefficient code used for product pair suggestions.

Problems I Identified

Double loops

Duplicate checks

Recomputing the same pairs

Expensive sorting operations


Improvements I Learned

I learned that performance can improve by:

Removing duplicate work

Avoiding unnecessary checks

Reducing time complexity

Using sorting strategically

Using pointer-based approaches instead of repeated nested loops


Reflection

This exercise improved my understanding of algorithmic thinking.

I learned that performance problems are often caused by repeated work and inefficient traversal patterns.

I also learned to think critically about time complexity instead of only focusing on whether code works.


---

Part 7: Testing and TDD Learning

What I Learned About Testing

During the testing exercises, I learned:

The importance of having a structured test plan.

How unit tests verify isolated behavior.

How integration tests verify system interaction.

Why TDD encourages clearer thinking before implementation.


Reflection

Before these exercises, I viewed testing mostly as validation after coding.

Now I understand that testing is also:

A design tool

A thinking tool

A debugging aid

Documentation for expected behavior


I learned that good tests improve confidence and reduce fear when modifying code.


---

Overall Reflection

This challenge significantly improved my ability to explore and understand unfamiliar codebases.

The biggest lessons I learned were:

Understanding code requires curiosity and structured thinking.

Asking questions improves learning more than immediately receiving answers.

Building mental models helps simplify complex systems.

Following data flow is essential for understanding application behavior.

Debugging is easier when invariants and expectations are clearly defined.

Performance optimization requires identifying repeated work.

Testing strengthens both implementation and understanding.


I also learned that confusion is a normal part of code exploration. Instead of feeling blocked by complexity, I learned to break problems into smaller pieces and reason about them step by step.

This challenge improved not only my technical understanding but also my confidence when approaching unfamiliar systems.


---

Conclusion

The Codebase Exploration Challenge helped me develop stronger analytical thinking, debugging skills, and system-level understanding.

I now approach unfamiliar codebases more strategically by:

Exploring structure first

Tracing data flow

Building hypotheses

Asking questions

Breaking down complexity into manageable parts


This experience helped me grow from simply reading code into actively reasoning about how software systems work.