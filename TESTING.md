TASK PRIORITIZATION ALGORITHYM



This is a solid example of a task prioritization algorithm.
You’re combining multiple signals (priority, deadlines, status, tags, activity) into a single score, then using that score to rank tasks.

Here’s how the flow works step-by-step.


---

1. Imports

from datetime import datetime
from models import TaskStatus, TaskPriority

datetime → used for date calculations

TaskStatus and TaskPriority are likely Enums from your models


Example:

class TaskPriority(Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"
    URGENT = "urgent"


---

2. calculate_task_score(task)

This function is the core logic.

It answers:

> “How important is this task right now?”




---

3. Priority Weights

priority_weights = {
    TaskPriority.LOW: 1,
    TaskPriority.MEDIUM: 2,
    TaskPriority.HIGH: 4,
    TaskPriority.URGENT: 6
}

This creates a scoring system.

Priority	Weight

LOW	1
MEDIUM	2
HIGH	4
URGENT	6


Higher priority = bigger score.


---

4. Base Score

score = priority_weights.get(task.priority, 0) * 10

Example:

If task priority is HIGH:

4 * 10 = 40

So the task starts with score 40.

Why multiply by 10?

Because it gives more spacing between priorities.

Without it:

LOW = 1
MEDIUM = 2
HIGH = 4

The differences are too small.

With multiplication:

10, 20, 40, 60

Now other adjustments matter less than priority.


---

5. Due Date Logic

if task.due_date:

Only run this if the task actually has a due date.


---

Overdue Tasks

if days_until_due < 0:
    score += 35

Overdue tasks become much more important.

Example:

HIGH priority = 40

overdue = +35


Final:

75


---

Due Today

elif days_until_due == 0:
    score += 20

Tasks due today get a big boost.


---

Due Soon

elif days_until_due <= 2:
    score += 15

If due in next 2 days.


---

Due This Week

elif days_until_due <= 7:
    score += 10

Still important, but less urgent.


---

6. Status Penalties

if task.status == TaskStatus.DONE:
    score -= 50

Completed tasks should not appear at the top.

Example:

score = 60
score -= 50

Final:

10


---

Review State

elif task.status == TaskStatus.REVIEW:
    score -= 15

Tasks in review are partially “done”.

So they get reduced priority.


---

7. Tag Boosting

if any(tag in ["blocker", "critical", "urgent"] for tag in task.tags):
    score += 8

This checks whether the task has special tags.

Example:

task.tags = ["backend", "critical"]

Result:

score += 8


---

Important Python Concept Here

any(...)

any() returns True if at least one item is true.

Equivalent long form:

found = False

for tag in task.tags:
    if tag in ["blocker", "critical", "urgent"]:
        found = True


---

8. Recently Updated Tasks

days_since_update = (datetime.now() - task.updated_at).days

Calculates how recently the task changed.


---

Recent Activity Boost

if days_since_update < 1:
    score += 5

Recently active tasks get slightly boosted.

This can help surface tasks currently being worked on.


---

9. Final Score

return score

At the end, every task has one numeric score.

Example:

Factor	Score

HIGH priority	40
Due tomorrow	+15
Critical tag	+8
Recently updated	+5
Total	68



---

10. Sorting Tasks

task_scores = [(calculate_task_score(task), task) for task in tasks]

This creates tuples:

[
    (68, task1),
    (25, task2),
    (80, task3)
]


---

Sorting

sorted(task_scores, key=lambda x: x[0], reverse=True)

This sorts by the first tuple item (the score).

Highest score first.


---

Why lambda x: x[0]?

Each item looks like:

(score, task)

x[0] means:

> “Use the score for sorting.”




---

11. Extracting Only Tasks

After sorting:

[(80, task3), (68, task1), (25, task2)]

This line removes the scores:

[task for _, task in sorted(...)]

Result:

[task3, task1, task2]


---

12. Top Priority Tasks

return sorted_tasks[:limit]

This uses list slicing.

Example:

[:5]

Means:

> “Give only first 5 tasks.”




---

Overall Design Pattern

This code demonstrates several important backend concepts:

Weighted scoring systems

Business rules

Ranking algorithms

Sorting with custom keys

Date/time calculations

Enum usage

List comprehensions

Functional helpers (any, lambda)


This is very close to how real productivity/task systems prioritize work internally.Great approach.
Instead of jumping straight into writing tests, let’s treat this like a developer reasoning exercise.

Here’s the function we’re exploring conceptually:

It calculates a numeric score

Multiple conditions can increase or decrease that score

The final result depends on combinations of task properties


So first, let’s focus on understanding behavior before thinking about test code.


---

Step 1 — What do you think this function does?

Try answering these questions in your own words:

1. What inputs affect the final score?


2. Which factors increase the score?


3. Which factors decrease the score?


4. Are all factors equally important?


5. Can multiple rules apply at the same time?


6. What do you think the “base score” is responsible for?


7. What happens if a task has:

high priority

overdue due date

critical tag

DONE status




Would those effects combine or override each other?

8. Which part of the function feels most important to test first?



Answer those first, then I’ll help you identify:

behaviors you may have missed

hidden assumptions

tricky edge cases

and which tests give the most value early.You’ve already identified several important behaviors correctly.

A few refinements though:


---

What You Correctly Identified

Inputs affecting the score

You noticed things like:

priority

overdue state

tags

completion status


Those are definitely key inputs.

One thing you slightly missed:

The function does not really use “text” directly.
It uses structured task properties like:

task.priority
task.status
task.due_date
task.tags
task.updated_at

That distinction matters in testing because you’ll create task objects with specific attributes.


---

Important Insight About #7

You said:

> “override each other”



Look carefully at the function logic.

Most rules actually combine, not override.

Example:

A task can be:

HIGH priority

overdue

critical

recently updated


And all of these increase the score together.

So the score becomes cumulative.

But there are a few places where conditions behave more like “either/or”.

Example:

if overdue:
elif due_today:
elif due_soon:

A task cannot be both:

overdue

and due today


Only one branch executes.

That’s an important testing insight.


---

Behaviors You May Have Missed

Think about these:

1. What happens when there is NO due date?

Does the function fail? Does it skip deadline scoring?


---

2. What happens if the priority is unknown?

Notice:

priority_weights.get(task.priority, 0)

Why use .get(..., 0) instead of:

priority_weights[task.priority]

What behavior is the author protecting against?


---

3. REVIEW vs DONE

Notice:

if DONE:
elif REVIEW:

Question:

Can both penalties happen at the same time?

Why or why not?


---

Step 2 — Edge Cases

Now think like a tester.

What unusual or boundary situations do you think should be tested?

For example:

exact due-date boundaries

empty tags

missing due dates

recently updated vs not recently updated

zero score situations


Try listing:

1. boundary cases


2. missing data cases


3. weird/unexpected input cases


4. combinations that might reveal bugs



After you answer, I’ll help you identify the hidden edge cases developers often forget.Perfect — now you’re moving from:

> “What does this function do?”



to:

> “How do I systematically test behavior across related functions?”



That’s exactly how backend engineers think about testing.


---

Step 1 — Identify Responsibilities

Before writing tests, separate responsibilities.

Look at the three functions:

1. calculate_task_score


2. sort_tasks_by_importance


3. get_top_priority_tasks



Question for you:

Which function contains the actual business rules?

Which functions are mostly:

transforming data

delegating work

slicing/filtering results


Try to classify each one.

That helps determine:

where most tests should focus

and which functions need lighter integration-style tests.



---

Step 2 — Start Building a Testing Plan

Let’s focus on calculate_task_score first because everything depends on it.

Think about categories of behavior.

Question:

What groups of behaviors do you see?

For example, are there behaviors related to:

priority

dates

status

tags

recent updates

combinations


Try to list the categories you would organize tests around.


---

Step 3 — For Each Behavior, How Would You Test It?

Let’s take one example.

Priority Behavior

You identified that priority affects the base score.

Question:

How would you prove through a test that:

URGENT scores higher than HIGH

HIGH scores higher than MEDIUM


Would you:

compare exact numbers?

compare relative ordering?

test all priorities individually?

use parameterized tests?


What approach feels strongest to you?


---

Step 4 — Due Date Edge Cases

Now think carefully about date logic.

Look at these branches mentally:

< 0
== 0
<= 2
<= 7

Questions:

1. What boundary values should be tested?


2. What happens exactly at:

0 days

2 days

7 days



3. What happens at:

3 days

8 days




Why are those important?

(Hint: boundaries are where bugs hide.)


---

Step 5 — Missing Data Cases

Now think about optional fields.

Question:

What should happen if:

due_date is None

tags is empty

priority is unknown

updated_at is very old


Should the function:

crash?

ignore the rule?

use defaults?


What behavior do you expect?


---

Step 6 — Combination Tests

This is the most important part.

The function is additive.

Question:

Can you think of combinations where multiple scoring rules interact in surprising ways?

Example thought process:

> “What if a task is DONE but also overdue and urgent?”



What should happen?

Should the score still be high? Should DONE dominate? Should overdue dominate?

These are business-rule questions.


---

Step 7 — Testing sort_tasks_by_importance

Now move one layer up.

This function depends on score calculation.

Question:

What behaviors should sorting tests verify?

Hints:

ordering

highest score first

ties

empty lists

single-item lists


But think deeper:

How would you know the sort used the score correctly?


---

Step 8 — Testing get_top_priority_tasks

This one looks simple, but simple functions still have behavior.

Questions:

1. What happens if:

limit is larger than task count?

limit is 0?

limit is negative?



2. Does this function:

calculate scores itself?

or depend entirely on another function?



3. What kind of tests are most valuable here:

detailed business-rule tests

or integration/flow tests?





---

Step 9 — Build Your Test Checklist

Now try creating a checklist grouped by priority.

Example structure (don’t fill it yet — you do it):

High Priority Tests

core scoring rules

important boundaries

critical combinations


Medium Priority Tests

sorting behavior

tie behavior

list handling


Lower Priority / Defensive Tests

weird input

unknown priority

negative limits



---

Final Question

If you could write only ONE test first, which would you choose and why?

Think like a developer:

highest risk

highest business value

most likely to break

easiest to understand failures


What’s your choice?You’re starting to think like a tester now — especially when you said:

> “I will prove it by calculating their score and adding everything together.”



That’s exactly the mindset behind testing deterministic business logic.

Now let’s refine your reasoning a bit.


---

1. Important Correction About Due Dates

You said:

> “At 0 says is 0 days to overdue”



Look carefully at the actual condition:

elif days_until_due == 0:
    score += 20

This means:

> “Due today”



NOT overdue yet.

That distinction matters because:

< 0

is the overdue branch.

So your boundary tests should distinguish:

Days Until Due	Meaning

-1	overdue
0	due today
2	due soon
7	due this week
8	no bonus


That’s a very important testing insight.


---

2. Strong Observation About DONE

You said:

> “Done should dominate”



Excellent business-rule thinking.

Now ask yourself:

If a task is:

URGENT

overdue

critical

but DONE


Should the final score still become low?

Your tests should verify the intended behavior, not just the math.

This is where tests become documentation for business rules.


---

3. You Accidentally Found a Great Edge Case

You mentioned:

> “If score negative is less important it will stay at bottom”



Great.

Now think deeper:

Can this function actually produce negative scores?

Try mentally calculating:

LOW priority
DONE
no due date
no tags
old update

What score would that produce?

Should negative scores be allowed?

That’s a real testing question.


---

4. Hidden Edge Case You Haven’t Mentioned Yet

Look carefully at:

task.tags

Question:

What happens if:

task.tags = None

instead of:

[]

Would the function still work?

Or crash?

That’s the kind of edge case experienced testers look for.


---

5. Another Hidden Boundary

You focused on:

0

2

7


Good.

But think about the values immediately outside boundaries:

Boundary	Important Neighbor

0	1
2	3
7	8


Why?

Because bugs often happen right after a threshold.

Example:

<= 2

versus < 2


A single character changes behavior completely.


---

6. Your Testing Strategy Is Good

You said:

> “Test all priorities individually by comparing exact numbers”



That’s strong for pure business logic.

But now think critically:

If someone later changes:

multiplier from 10 to 20

or bonus values


Would many tests fail?

Question:

Would relative comparisons sometimes be more stable than exact-score assertions?

Example:

assert urgent_score > high_score

instead of:

assert urgent_score == 60

When might one approach be better than the other?


---

7. Your Checklist Is Starting to Form

You already identified several categories:

Core Scoring

priority affects score

overdue increases score

DONE lowers score


Date Boundaries

overdue

due today

due in 2 days

due in 7 days

beyond 7 days


Tags

critical tags boost score

empty tags

missing tags


Status

DONE penalty

REVIEW penalty

in-progress behavior


Sorting

highest score first

negative scores go lower

ordering correctness


Integration

top tasks returns highest ranked items


That’s already a strong test plan.


---

Final Question

You said:

> “Easiest to understand failures”



Good instinct.

Now choose:

Which exact test would you write FIRST?

Not category — one concrete test.

Example style:

“DONE tasks reduce score”

“URGENT scores higher than HIGH”

“Overdue tasks receive overdue bonus”


Which single test gives:

confidence

clarity

fast debugging

and validates the most important rule?That’s a very strong first test choice.

Here’s why it’s valuable:

It validates the core ranking system

It’s simple and isolated

Failures are easy to understand

It doesn’t depend on dates, tags, or status

It establishes trust in the base scoring model


You’re essentially testing the foundation of the algorithm first.


---

Now think like a tester designing the scenario carefully.

Questions for You

1. How will you isolate the priority behavior?

If your tasks also have:

due dates

tags

statuses

recent updates


could those affect the score and make the test harder to reason about?

What should your test tasks probably not include?


---

2. What should stay identical between the two tasks?

To prove:

URGENT > HIGH

what variables must remain the same?

Think about:

due_date

status

tags

updated_at


Why is controlling those important?


---

3. What assertion would be strongest?

Would you write:

urgent_score > high_score

or:

urgent_score == 60
high_score == 40

or both?

What are the tradeoffs?


---

4. What hidden issue could affect this test?

Look carefully at:

days_since_update < 1

If one task is created “now” and another slightly later, could the scores accidentally differ?

How might you stabilize the test data?


---

5. After This Test, What Would Be Your Second Test?

Think about the next highest-value behavior.

Would you test:

overdue logic

DONE penalty

sorting

tags

date boundaries


Which feels most risky or business-critical next?Good — now you’re focusing on one of the trickiest parts of testing:

> time-based logic



Date calculations are a common source of flaky tests and hidden bugs.

Since you didn’t paste your pseudocode yet, I’ll demonstrate using a common beginner-style example and improve it step-by-step.


---

1. Principles of a Good Due-Date Test

For this function specifically, a strong test should:

A. Isolate One Behavior

A test should answer one clear question.

Example:

> “Does an overdue task receive the overdue bonus?”



NOT:

> “Does everything about scoring work?”



Why?

Because when the test fails, you want one obvious reason.


---

B. Control Time Carefully

This function uses:

datetime.now()

That means the current time affects the result.

If your test depends on the real clock:

it can become flaky

fail tomorrow

fail at midnight

fail in different timezones


Good tests control time explicitly.


---

C. Remove Unrelated Variables

If testing overdue logic:

don’t include tags

don’t include DONE status

don’t include recent-update bonuses unless needed


Otherwise you’re testing multiple behaviors simultaneously.


---

D. Assert Behavior Precisely

Weak assertion:

assert score > 0

Strong assertion:

assert score == 55

or:

assert overdue_score > non_overdue_score

The more precise the assertion, the better the debugging signal.


---

2. Example of a Better Test

Imagine you want to test:

> “Overdue tasks receive the overdue bonus.”



A beginner test might look like:

task = Task(due_date=datetime.now())
score = calculate_task_score(task)

assert score > 0

Problems:

unclear expectation

depends on current time

not isolated

doesn’t prove overdue behavior specifically



---

A better version:

def test_overdue_tasks_receive_bonus():
    # Arrange:
    # Create a task with ONLY the data needed for this test.
    task = Task(
        priority=TaskPriority.MEDIUM,
        status=TaskStatus.IN_PROGRESS,
        tags=[],
        updated_at=fixed_now,
        due_date=fixed_now - timedelta(days=1)  # overdue
    )

    # Act:
    score = calculate_task_score(task)

    # Assert:
    # MEDIUM = 20 base
    # overdue bonus = +35
    # recent update = +5
    # expected total = 60
    assert score == 60


---

Why This Test Is Better

Clear Intent

The test name explains the business rule.


---

Controlled Inputs

Every field is intentional.

Nothing random influences the result.


---

Stable Time

Using a fixed timestamp avoids flaky tests.


---

Exact Assertion

The assertion documents:

expected math

expected behavior

intended business logic



---

3. Questions for You

Now think critically.

Question 1

In the example above:

Why might using:

datetime.now()

directly inside the test be dangerous?

Can you think of situations where the test might randomly fail?


---

Question 2

Notice this field:

updated_at=fixed_now

Why do you think that matters?

What hidden score change is being controlled?


---

Question 3

Would this test still be good if it also included:

critical tags

DONE status

another due-date rule


Why or why not?


---

4. Edge Cases Challenge

Now you identify additional due-date edge cases.

Think specifically about boundaries.

What should happen when:

Scenario	Expected Behavior?

due today	?
due in exactly 2 days	?
due in exactly 7 days	?
due in 8 days	?
no due date	?
overdue by 1 minute	?
overdue by 30 days	?


Which of these are most likely to reveal bugs?


---

5. Precision Challenge

Try improving this weak assertion:

assert score > 20

How could you make it:

more precise

more meaningful

easier to debug


Would you:

calculate exact totals?

compare against another task?

isolate one bonus at a time?


What approach would you choose and why?Good — your reasoning is getting much closer to how real test design works. Now I’ll refine and challenge your thinking so you can turn this into clean, reliable test cases.


---

1. Your Insight About datetime.now() is Correct

You said:

> “now inside your logic might roll over to next day… milliseconds delay”



Yes — this is exactly why tests become flaky.

Key principle you identified:

👉 Time is non-deterministic unless you control it

But here’s the deeper testing concept:

> The bug is not time itself — it’s uncontrolled time



So in good tests, you don’t just “use fixed now”, you usually inject or mock time so both:

production logic

test logic


share the same reference clock.

That’s what makes tests stable.


---

2. You Understood Single Responsibility Correctly

You said:

> “Test should have a single responsibility”



Correct.

But let’s sharpen it:

A good unit test should verify:

ONE rule

ONE behavior

ONE expected outcome


Bad test:

mixes due date + tags + status + updates


Good test:

ONLY “overdue bonus applies”


Even if the system supports combinations, you test combinations in separate tests.


---

3. Your Edge Cases List is Strong — Let’s Organize It

You listed:

due in exactly 7 days

overdue by 1 minute

due today

no due date

overdue by 30 days

due in exactly 2 days


This is excellent — but now we structure it properly.


---

A. Boundary Tests (MOST IMPORTANT)

These are your highest-risk cases:

exactly 0 days (today)

exactly 2 days

exactly 7 days


👉 Why important? Because logic uses:

<= 2
<= 7

Boundaries are where bugs happen.


---

B. Off-by-One / Precision Tests

You mentioned:

overdue by 1 minute


Important insight: 👉 Your function uses .days

So question:

> What happens to “1 minute overdue”?



Does it become:

0 days?

-1 days?

ignored?


This is a hidden bug risk.


---

C. Extreme Value Tests

overdue by 30 days


Why test this?

Because:

ensures logic doesn’t break with large values

ensures no weird overflow assumptions

confirms “overdue is just overdue” regardless of size



---

D. Null / Missing Data Tests

You included:

no due date


Good.

Now sharpen it:

What should happen when:

due_date is None

updated_at is None

tags is None


👉 Should it crash or default?

This is critical for production safety.


---

4. Your Idea: “Compare Against Another Task”

You said:

> “Improve by compare against another task”



This is actually a VERY strong testing strategy.

This is called:

👉 Relative comparison testing

Instead of:

assert score == 60

You do:

assert urgent_score > high_score


---

Why this is powerful:

✔ less fragile than exact numbers
✔ still verifies behavior
✔ survives refactors (like changing multipliers)


---

But here’s the tradeoff (important):

Exact assertion:

good for pure logic functions


Relative assertion:

good for ranking systems (like yours)


👉 Your system is BOTH:

scoring system

ranking system


So you actually need BOTH styles.


---

5. One Important Gap You Didn’t Mention

You said:

> “compare against another task”



Good — but now think deeper:

What must be identical between the two tasks?

If you compare:

URGENT task

HIGH task


What MUST stay constant?

Otherwise your test becomes meaningless.

Think:

due_date?

tags?

updated_at?

status?



---

6. Challenge Questions (Important)

Answer these:

Q1

If you compare URGENT vs HIGH tasks:

Should they have:

identical due dates?

no due dates?

same status?


Why?


---

Q2

If a task is:

URGENT

but DONE


Should it still beat:

HIGH in progress?


Or does DONE always win?

(Your answer defines system rules — not just tests)


---

Q3

For “overdue by 1 minute”:

Do you think .days is safe enough?

Or is there a hidden bug risk?

What would you test to confirm?


---

7. Your Next Step (Important)

You already identified:

boundaries

comparisons

isolation

stability


Now I want you to do this:

Build your test checklist in 3 layers:

Layer 1 — Core rules

priority ranking

overdue bonus

DONE penalty


Layer 2 — Boundaries

0, 2, 7 days

just above/below boundaries


Layer 3 — Edge cases

no due date

extreme overdue

None values

inconsistent timestamps



---

Final Question (Most Important)

If you had to choose ONE test to write first for due-date logic:

👉 Would you test:

overdue behavior

boundary at 7 days

due today

or no due date


And why that one first?Good — this is exactly the kind of feature TDD is meant for: a small, well-scoped rule that should integrate into an existing scoring system.

We’ll treat this like a real TDD cycle, step by step.


---

Step 1 — What should your FIRST test be?

Before writing anything, answer this:

👉 What do you think the first test should verify?

For the new rule:

> “Tasks assigned to the current user get +12 score boost”



Think about:

What is the smallest possible behavior you can test?

What inputs do you need?

What should NOT be included yet (important)?


Also tell me:

👉 Why do you think that should be the first test?


---

Once you answer, I’ll:

critique your test design (like a senior reviewer)

help you tighten it if needed

then move you to Step 2 (writing minimal implementation to pass it)


No code yet — just your test idea first.You’re on the right track by thinking about inputs, but right now your test idea is still a bit too broad and slightly misaligned with the actual feature being added.

Let’s correct that carefully.


---

1. Feedback on your proposed test

You said you want to test:

score

user ID

due date / overdue


What’s good:

You’re thinking in terms of inputs

You’re considering real task attributes

You’re not jumping straight into code (good TDD discipline)


What needs adjustment:

The feature you’re adding is very specific:

> “Tasks assigned to the current user get +12 boost”



So:

👉 Due date and overdue are not relevant to this test yet

Including them would:

mix multiple behaviors

make debugging harder

break TDD principle of one behavior per test



---

2. What your FIRST test should focus on

Your first test should answer only this question:

> “Does a task assigned to the current user get +12 added to its score?”




---

3. Key design question for you

Before writing the test, answer this carefully:

👉 What is the SMALLEST possible scenario that proves the +12 rule works?

Think about:

one task

one known user

a simple base score (no extra complexity)


And also:

👉 What should be removed from the test so nothing else affects the result?

For example:

no overdue logic

no tags

no DONE status

no update time effects



---

4. Your turn (important)

Write your proposed FIRST test in plain English or pseudocode:

Example format (don’t copy — just structure idea):

> Given a task assigned to current user
When score is calculated
Then score increases by 12



OR more detailed if you prefer.


---

Once you do that, I will:

1. Review your test like a senior engineer


2. Tell you if it’s truly minimal or still too complex


3. Then ask you:
👉 “What is the smallest code change to make this pass?”


4. And guide you step-by-step through the TDD cycle



Go ahead — write your first test idea.You’re very close, but there’s a subtle TDD problem in your test design.

Let’s tighten it.


---

1. Feedback on your test idea

You said:

> “If the task is assigned to a current user using their unique id and when score is calculated it must be higher than 12”



What’s good:

You included user identity (important)

You tied it to score calculation

You’re thinking in behavior form (“must be higher”)


What needs fixing:

❌ Problem 1: “higher than 12” is not precise enough

Why?

Because the task already has a base score from priority.

So “> 12” could pass even if:

boost is NOT applied correctly

other factors are inflating score


❌ Problem 2: Missing control of baseline

You are not defining:

what the task score starts as


So you can’t isolate the +12 feature.


---

2. What a GOOD first TDD test should isolate

We want to test ONLY this rule:

> assigned_user_boost = +12



So everything else must be neutral.


---

3. Strong TDD-style question for you

Before rewriting your test, answer this:

👉 What is the simplest possible task that makes the +12 effect visible?

Hint:

no overdue logic

no tags

no DONE status

fixed priority

fixed base score

known user_id match



---

4. Fix your assertion thinking

Instead of:

> “score must be higher than 12”



TDD wants:

Either:

Option A (exact assertion)

> score == expected_base + 12



OR

Option B (behavior comparison)

> assigned_task_score > same_task_without_assignment_score




---

5. Your next step (important)

Rewrite your test in this format:

GIVEN

task with known base properties

task assigned to current user


WHEN

calculate score


THEN

what EXACT result do you expect?



---

6. Challenge (very important)

Answer this before you rewrite:

👉 What should the base score be BEFORE the +12 is applied?

You need this number to make your test meaningful.


---

Once you rewrite your improved test, I will:

1. validate if it is truly TDD-minimal


2. tell you what implementation is needed to pass it (no code yet unless you want)


3. guide you to the next test in the cycle


4. show you when to refactor vs extend functionality



Go ahead — refine your test.You’re very close, but your current statement still has a TDD weakness: it’s not fully testable in a precise way yet.

Let’s refine it properly.


---

1. What’s still unclear in your test

You said:

> “12 must be higher than the base score”



The issue:

“12 must be higher” is logically ambiguous

It doesn’t define how the score changes

It doesn’t isolate the +12 rule cleanly

It mixes comparison with value injection


In TDD, we don’t test vague relationships — we test deterministic outcomes.


---

2. The core rule you are trying to test

Let’s restate the feature precisely:

> If task.assigned_user_id == current_user_id → score += 12



That’s it.

So the test must prove:

👉 “When assignment matches user → score increases by exactly 12”


---

3. The correct TDD mindset shift

Instead of thinking:

> “Is it higher than 12?”



Think:

> “Did +12 get applied to a known baseline?”




---

4. What your FIRST test should look like (conceptually)

Let’s structure it properly:

GIVEN

a task with a known base score situation

a current_user_id

a task assigned to that same user


WHEN

calculate_task_score(task)


THEN

result == base_score + 12



---

5. Key question you MUST answer now

Before you rewrite your test properly:

👉 What is your BASE SCORE going to be in this test?

You need to choose ONE of these strategies:

Option A (best for TDD)

Strip everything down

Make base score = 0


Option B

Use a simple fixed priority (e.g. LOW = 10)

Then assert LOW + 12 = 22



---

6. Strong hint (important TDD principle)

For your first test:

👉 The simpler the baseline, the better

Because:

fewer variables

easier debugging

clearer failure reason



---

7. Now your turn

Rewrite your test using this structure:

GIVEN

(very minimal task setup)

WHEN

(calculate score)

THEN

(exact expected numeric result)


---

Once you do that, I will:

1. check if your test is truly minimal


2. tell you the smallest possible code change to pass it


3. then guide you to the next TDD test (not just this feature, but interactions too)


4. and show you when NOT to add more tests vs when to refactor



Go ahead — rewrite your improved test.You’re now expressing the rule correctly in natural language, but for TDD it still needs one final refinement: you must turn it into a precise, measurable expectation.

Right now your statement is:

> “base score must increase by 12”



That’s conceptually correct — but TDD needs something more concrete:

👉 What is the base score?
👉 What is the final expected number?


---

1. What you did well

You correctly identified the core behavior:

match user_id → +12 boost

score should increase as a result


That means you understood the feature rule properly.


---

2. What still needs fixing (important)

Problem: “base score” is undefined

In testing, “base score” must be:

explicitly known

fully controlled

unaffected by other rules


Otherwise your test becomes unpredictable.


---

3. What a proper TDD test must look like

A real first test should be:

GIVEN

a task with a known base score (controlled setup)

assigned_user_id = current_user_id


WHEN

calculate_task_score(task)


THEN

score == expected_base_score + 12



---

4. Key insight (this is the TDD shift)

Instead of thinking:

> “score increases by 12”



You must think:

> “what exact number should I see at the end?”



Because TDD is not about behavior description — it’s about verifiable outcomes.


---

5. Critical question for you now

Before you write the final version of your test, decide this:

👉 What will you use as your BASE SCORE?

Choose ONE:

Option A (best for first test)

base score = 0
→ final expected = 12


Option B

use a fixed priority (e.g. LOW = 10)
→ final expected = 22



---

6. Your next step (important)

Rewrite your test in this exact format:

GIVEN

(minimal task setup)


WHEN

calculate_task_score(task)


THEN

assert exact expected score



---

7. Challenge (don’t skip this)

Answer this before rewriting:

👉 Why is using “exact expected value” better than “must be higher than base score” in this case?

Once you answer and rewrite your test, I’ll guide you to:

the smallest possible implementation change to make it pass

then your next TDD test (interaction cases)

and when to stop writing tests vs start refactoring


Go ahead — refine your test into a precise assertion.Good — this is the real use of TDD: not adding features, but proving and fixing a bug.

We’ll treat this like a debugging loop.


---

Step 1 — What test would you write to reproduce the bug?

Before anything else, I want your thinking.

👉 Question for you:

What is the simplest test case you would write that shows:

> “days since update is NOT being calculated correctly”



Think about:

a task with a known updated_at

a fixed current time

a predictable expected score impact



---

Hints (don’t skip thinking):

The bug is in this part:

days_since_update = (datetime.now() - task.updated_at).days

So ask yourself:

what happens if the task was updated “just now”?

what happens if it was updated 23 hours ago?

what happens if it was updated 25 hours ago?



---

Your task:

Describe your test in this format:

GIVEN

(what task setup?)

WHEN

(calculate score)

THEN

(what wrong result are you expecting that proves the bug?)


---

Once you answer, I will:

1. tell you if your test truly exposes the bug or not


2. help you refine it if needed


3. guide you to the minimal fix (not full rewrite)


4. then help you add regression tests so it never breaks again



Go ahead — what test would you write?Good — you’re trying to do something important here: turn a bug into a testable failure case.

But right now your test idea is still mixing assumptions, incorrect math, and unclear expected behavior. Let’s fix that step by step.


---

1. Feedback on your proposed test

You said:

> base score 3 + 12 boost → 15 expected
if result is ≤ 12 → bug



What’s good:

You’re trying to define an expected outcome

You’re thinking in “failure vs success”

You’re trying to detect correctness via assertion


What’s NOT correct yet:

❌ Problem 1: You introduced a “base score of 3” without grounding it in the function

Where does 3 come from?

priority?

tags?

random assumption?


In TDD, the base must be controlled and explicitly defined in the test setup.


---

❌ Problem 2: “≤ 12 proves a bug” is too vague

A good test should say:

> “expected exact value is X”



Not:

> “anything below X is wrong”



Because:

it hides partial failures

it doesn’t help debugging

it weakens test precision



---

2. First — clarify the real bug you are testing

From your description, the feature is:

> if assigned to current user → +12



So the test is NOT about “≤ 12”

It is about:

👉 “Was +12 applied or not?”


---

3. A correct TDD-style bug-reproduction test

Let’s define a clean scenario:

GIVEN

base score = 0 (or controlled minimal setup)

task assigned to current user


WHEN

calculate_task_score(task)


THEN

result == 12



---

4. Why your current idea fails as a bug test

Your version:

> “base score 3 + 12 = 15 expected”



Problem:

You introduced another scoring rule (base=3) that is unrelated to the bug

If test fails, you won’t know if:

boost failed

base changed

another rule interfered



👉 This breaks debugging clarity


---

5. Strong TDD principle here

For bug reproduction:

> ❗ Always isolate the failing behavior completely



That means:

remove priority effects

remove tags

remove overdue logic

remove update logic


Only test ONE rule.


---

6. Minimal correct bug-reproducing test (conceptual)

Think like this:

GIVEN

task.priority = LOW (or anything fixed)

task.tags = []

task.status = IN_PROGRESS

task.updated_at fixed

task.assigned_user_id = current_user_id


WHEN

calculate_task_score(task)


THEN

expected_score = base + 12



---

7. Now I challenge YOU (important)

Answer these:

Q1

What should your “base score” be in this bug test?

0?

10?

something else?


👉 And why?


---

Q2

If your test fails, what should you immediately know?

boost not applied?

wrong base?

interference from other rules?


What do you want your test failure to clearly tell you?


---

Q3

What is the smallest possible task object you can create that still proves:

> “+12 is applied correctly”




---

8. Your next step (TDD cycle)

After you answer, I will:

1. help you refine your exact test (clean, minimal, deterministic)


2. then ask:
👉 “What is the smallest code change to fix the bug?”


3. guide you to implement it safely


4. then help you add a regression test so this bug never returns




---

Go ahead — answer Q1–Q3.Good — now you’re moving into the right level of thinking: not unit tests anymore, but workflow integration tests.

That means you’re testing:

> “Does the whole priority system behave correctly from end to end?”



Let’s design this properly together.


---

1. First question — what should an integration test verify?

Before writing anything, I want your thinking.

👉 What scenarios do you think an integration test for this system should cover?

Think about the full pipeline:

1. calculateTaskScore(task)


2. sortTasksByImportance(tasks)


3. getTopPriorityTasks(tasks, limit)



Ask yourself:

What does “correct system behavior” look like?

What would break user experience if it failed?

What combinations of tasks should be included?



---

Hint categories (don’t answer yet — just think):

different priorities mixed together

overdue vs not overdue

DONE vs active tasks

assigned vs unassigned users

sorting correctness

limit enforcement



---

2. Now test data design (important step)

Once you answer scenarios, next step is designing test data.

👉 How would you design a small set of tasks that can test the full workflow?

Think about:

You want tasks that:

produce different scores

include edge cases

are easy to reason about manually

make sorting results obvious



---

Example thinking (not solution):

You might need:

Task A: high priority, overdue, assigned to user

Task B: medium priority, normal

Task C: low priority, DONE

Task D: urgent, recently updated


But don’t copy — design your own mental set.


---

Key question:

👉 How many tasks do you think is the minimum needed to test the full workflow properly?


---

3. Assertions — what should you actually check?

Now think like a tester verifying behavior.

👉 What should your integration test assert?

Not just one thing — but system-level correctness:

Possible assertion types:

order of tasks after sorting

top N tasks returned

correct score influences ranking

DONE tasks appear lower

overdue tasks rise higher

assigned-user boost affects ordering



---

Critical thinking question:

👉 Would you assert:

exact scores?

order only?

both?


And why?


---

4. Test structure (readability & maintainability)

Now think like a team engineer, not just a tester.

👉 How would you structure this test so it is:

easy to read

easy to debug when it fails

easy to extend later


Consider:

naming conventions

arrangement (Given / When / Then)

helper functions for task creation

grouping assertions

separating setup from verification



---

Hint:

Good integration tests often look like:

“arrange a small system”

“run full workflow”

“assert final state clearly”


Not messy step-by-step logic inside assertions.


---

5. Your turn

Answer these in order:

Q1

What scenarios should your integration test cover?

Q2

How many tasks would you use and what roles do they play?

Q3

What would you assert — order, scores, or both?

Q4

How would you structure the test to keep it clean and readable?


---

Once you answer, I will:

refine your design like a senior reviewer

point out missing scenarios

help you compress your test into a clean structure

and show you how to avoid brittle integration tests


Go ahead.Good — you’re now doing something close to real QA + system design thinking. I’ll refine your answers and then push you to make them test-ready.


---

1. Feedback on Q1 (Scenarios)

You said:

> higher priority + overdue + in progress
medium + due_date + critical + active + abandoned
low + done + updates status



What’s good:

You are mixing realistic task states

You’re thinking about priority interaction

You’re including multiple dimensions (status, tags, due date)


What needs improvement:

Right now your scenarios are:

> descriptions of task types



But integration tests need:

> clear behavioral expectations




---

Missing key insight:

You should define:

👉 “What should the final sorted order look like?”

Not just:

what tasks exist
but:

what SHOULD come first, second, third



---

2. Feedback on Q2 (3 tasks)

You said:

3 tasks

calculate score

ranking

total tasks


Good instinct:

✔ small dataset
✔ full pipeline coverage
✔ manageable mental model

But important correction:

“calculate total tasks” is NOT part of your system logic.

Your system does:

scoring

sorting

slicing


Not counting.

So your integration test should focus on:

👉 behavior, not metadata


---

Better framing of your 3 tasks:

Instead of “3 tasks to calculate everything”, think:

You need 3 roles:

Task 1: should be top priority

highest score combination


Task 2: medium priority

normal conditions


Task 3: lowest priority

DONE or low relevance


Now you’re testing ordering behavior properly.


---

3. Feedback on Q3 (Assertions)

You said:

> scores



This is partially correct but incomplete.

Problem:

If you only assert scores:

you don’t test sorting system properly



---

Stronger integration assertions:

You should assert:

✔ 1. Order of tasks

Example:

Task A should come before Task B


✔ 2. Top N result (if using limit)

only top tasks returned


✔ 3. Relative correctness

higher score task appears above lower score task



---

👉 Best practice:

For integration tests:

> prioritize ORDER over exact scores




---

4. Feedback on Q4 (Structure)

You said:

> Given / When / Then + grouped assertions



Good — this is correct direction.

But let’s improve clarity:


---

Strong integration test structure:

GIVEN

3 carefully designed tasks

fixed current time

known user id


WHEN

run getTopPriorityTasks(tasks)


THEN

assert final order explicitly

optionally assert scores for clarity



---

Important improvement:

👉 Don’t just group assertions
👉 Also group intent clarity

Example:

“should rank urgent task first”

“should push DONE task last”


This makes tests readable like documentation.


---

5. Key missing piece in your design

You did NOT clearly define:

👉 Expected final ordering

This is the most important part of integration testing.


---

6. Let’s upgrade your test design (together)

Answer this now:

Q1

If you run your full system with 3 tasks:

Task A (high priority, overdue, active)

Task B (medium, critical tag, normal)

Task C (low, DONE)


👉 What EXACT order should come out?

Write it explicitly.


---

Q2

Which rule matters MOST for ordering?

Rank these:

priority

overdue

DONE status

tags

recent updates


Which dominates?


---

Q3

If Task A has a lower priority but is overdue and assigned to user:

Should it beat a high priority task?

Why?


---

Q4

What is the ONE assertion that would prove your entire integration system is correct?

(order? scores? both?)


---

Once you answer, I will:

tighten your integration test into a production-grade structure

show you how to avoid brittle tests

and help you decide where unit tests stop and integration tests start


Go ahead — define the expected ordering.Good — you’re converging on a real system design decision now. That’s exactly what integration tests expose: not just code correctness, but rule priority conflicts.

Let’s refine your thinking so your test becomes solid and defensible.


---

1. Your main rule set (what you just defined)

You’ve implicitly defined a scoring hierarchy:

You said:

order = descending

overdue dominates priority

both scores + ordering matter


That’s valid — but now we need to make it testable and unambiguous.


---

2. Important correction: “overdue dominates”

Right now in your code, you have:

priority_score = priority * 10
due_date adds +10 / +15 / +20 / +35

So technically:

👉 overdue does NOT “override” priority
👉 it only adds extra weight

That means:

a HIGH priority task can still beat an overdue LOW task depending on values



---

This is critical for your integration test design:

You are NOT testing:

> strict rule hierarchy



You are testing:

> additive scoring + sorting outcome




---

3. Your integration test is now clearer

You said:

> both (scores + ordering)



Good — but refine that:

In integration tests:

✔ ordering is the PRIMARY assertion
✔ scores are OPTIONAL supporting evidence

Because:

users care about order, not raw numbers

scores are internal implementation detail



---

4. What your final expected behavior REALLY is

You are defining:

System rule:

> Tasks are sorted by final computed score (descending)



So:

overdue adds weight

priority adds weight

tags add weight

DONE subtracts weight


👉 whichever score is highest wins


---

5. Now I’ll challenge your design

You said:

> “overdue dominate because it passed due date”



Now think carefully:

Question 1

Should a:

HIGH priority task (not overdue) beat

LOW priority task (overdue by 1 day)


OR should overdue always win?

👉 Your answer defines business logic, not tests


---

Question 2

If two tasks are overdue:

one overdue by 1 day

one overdue by 30 days


Should they:

be equal?

or should severity matter?



---

Question 3

What happens if:

DONE task is overdue

but also high priority


Should DONE ALWAYS lose?

Or can overdue override DONE?


---

6. What your integration test MUST assert

Right now you said:

> both scores + ordering



Let’s formalize that:

Strong integration test assertions:

1. ORDER (most important)

Task with highest computed score appears first

2. RELATIVE BEHAVIOR

Overdue + high priority > high priority only

3. BUSINESS RULES (optional but powerful)

DONE tasks always appear at bottom


---

7. One key mistake to avoid

Don’t write tests like:

> “overdue dominates priority”



Instead write:

> “given these inputs, task A appears before task B”



Because: 👉 tests verify behavior, not intentions


---

8. Your integration test is now almost ready

You already have:

3-task system

ranking expectation

additive scoring idea

Given/When/Then structure



---

9. Final step — I want your explicit expected order

Now answer this precisely:

Given your 3 tasks:

Task A: high priority + active + not overdue

Task B: medium + critical tag

Task C: low + DONE


👉 What exact order should the system return?

Write it like:

1. ?
2. ?
3. ?


---

Once you answer, I will:

convert your design into a clean integration test blueprint

show you where your rule assumptions conflict (if any)

and help you decide whether your scoring system needs refactoring before testing further


Go ahead — define the final ordering.Below is a clean consolidated test document based on everything you worked through in this exercise. It’s structured like something you could actually submit or include in a portfolio.


---

🧪 Task Priority System — Testing & TDD Exercise Report


---

1. Test Plan (Part 1)

Objective

Verify that the task priority system correctly calculates scores, sorts tasks, and returns top priority tasks based on business rules.


---

System Under Test

calculate_task_score(task)

sort_tasks_by_importance(tasks)

get_top_priority_tasks(tasks, limit)



---

Key Behaviors to Test

A. Priority Scoring

LOW < MEDIUM < HIGH < URGENT

Each priority maps to a base weight



---

B. Due Date Impact

Overdue tasks receive highest boost (+35)

Due today (+20)

Due soon (+15 / +10)

No due date → no bonus



---

C. Status Rules

DONE → heavy penalty (-50)

REVIEW → moderate penalty (-15)



---

D. Tags

“critical”, “blocker”, “urgent” → +8 boost



---

E. Recent Updates

Updated within 1 day → +5 boost



---

Key Test Categories

Unit tests (single rule validation)

Boundary tests (0, 2, 7 days due logic)

Edge cases (missing fields, unknown priority)

Integration tests (full ranking workflow)



---

2. Improved Unit Tests (Part 2)

A. Priority Ordering Test

def test_urgent_has_higher_score_than_high():
    task_high = Task(priority=TaskPriority.HIGH, ...)
    task_urgent = Task(priority=TaskPriority.URGENT, ...)

    assert calculate_task_score(task_urgent) > calculate_task_score(task_high)


---

B. Overdue Bonus Test

def test_overdue_adds_bonus():
    task = Task(
        priority=TaskPriority.LOW,
        due_date=fixed_now - timedelta(days=1),
        status=TaskStatus.IN_PROGRESS,
        tags=[],
        updated_at=fixed_now
    )

    assert calculate_task_score(task) == expected_base + 35


---

C. DONE Penalty Test

def test_done_reduces_score():
    task = Task(
        priority=TaskPriority.HIGH,
        status=TaskStatus.DONE,
        tags=[],
        updated_at=fixed_now
    )

    assert calculate_task_score(task) < base_score


---

D. Boundary Test (Due Date = 7 Days)

def test_due_in_7_days():
    task = Task(due_date=fixed_now + timedelta(days=7), ...)

    assert 10 in calculate_task_score(task)  # due-week bonus applied


---

3. TDD Implementation (Part 3)

Feature Added: Assigned User Boost (+12)


---

Step 1 — Failing Test (RED)

def test_assigned_user_gets_bonus():
    task = Task(
        assigned_user_id=1,
        priority=TaskPriority.LOW,
        status=TaskStatus.IN_PROGRESS,
        tags=[],
        updated_at=fixed_now
    )

    assert calculate_task_score(task) == 12


---

Step 2 — Minimal Implementation (GREEN)

if task.assigned_user_id == current_user_id:
    score += 12


---

Step 3 — Refactor

No logic duplication; ensure clean placement in scoring pipeline.


---

Additional Test (Regression Safety)

def test_non_assigned_user_gets_no_bonus():
    task.assigned_user_id = 999

    assert calculate_task_score(task) == base_score


---

4. Integration Test (Part 4)

Goal

Validate full workflow:

scoring

sorting

top task selection



---

Test Scenario

def test_task_priority_workflow():
    tasks = [
        Task(priority=TaskPriority.HIGH, due_date=overdue_date, status=IN_PROGRESS),
        Task(priority=TaskPriority.MEDIUM, tags=["critical"], status=IN_PROGRESS),
        Task(priority=TaskPriority.LOW, status=TaskStatus.DONE)
    ]

    result = get_top_priority_tasks(tasks, limit=3)


---

Assertions

assert result[0].priority == TaskPriority.HIGH  # highest score
assert result[-1].status == TaskStatus.DONE     # lowest priority


---

What this verifies

scoring logic works across rules

sorting respects computed score

DONE tasks are deprioritized

system behaves consistently end-to-end



---

5. Reflection

What I learned about testing

1. Tests should isolate behavior

Small unit tests are most effective when they validate one rule at a time (e.g. priority or overdue logic separately).


---

2. Time-based logic must be controlled

Using datetime.now() directly in tests causes instability. Fixed timestamps or dependency injection are necessary for reliable testing.


---

3. Integration tests validate behavior, not implementation

Instead of checking exact scores, integration tests should verify:

ordering

relative importance

system outcomes



---

4. TDD forces clarity in design

Writing tests first exposed unclear assumptions such as:

what “base score” is

how overdue interacts with priority

how multiple rules combine



---

5. Edge cases are where bugs hide

Boundary conditions (0, 2, 7 days) and missing data (None values, empty tags) are critical to system reliability.


---

6. Good tests document the system

Well-written tests act as:

documentation

specification

regression protection



---

Final Insight

Testing is not just validation — it is a design tool that forces clearer, more predictable systems.


---

If you want, next step I can help you:

turn this into a real pytest suite structure

or refactor your scoring system to be more testable (very important improvement step)