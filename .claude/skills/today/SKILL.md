---
name: today
description: Everyday logging and day questions. Use whenever the person reports or asks to log a meal, drink or snack, a weight or other reading, steps, sleep, a workout or its sets, a new allergy, medication, injury or condition, a correction or an entry to remove, or sends a screenshot or photo of any of these; and whenever they tell you something to keep: a trip or event, a constraint on when or where they train, a profile fact, their values or an injury's story, a precommitment, a standing instruction, how they want you to talk to them, a goal, a trigger or what they track, or a follow-up; and when they ask what's left today, how a measure is trending, what they did recently, or what the findings say.
---

# Today: logging and day questions

The rules in CLAUDE.md hold throughout: every write takes callKey, units go as they were said, a day goes as `{ "date": "YYYY-MM-DD" }` when they name it or `{ "relative": "today" | "yesterday" }`, and you say back what the result shows was saved. Ids and versions (`{ id, editCount }`) always come from a read; never make one up.

## A meal or drink

1. **One `get_repeats` call for the whole meal,** with `terms`: one term per item, a few words each from what they said (up to 8). Not one call per item.
2. **`log_meal`** with `day` and `items`:
   - **A match:** `{ "saved": { kind, id } }` with `portions` (how many of its stored portion) or `grams` (for a food with a gram weight per unit). Software logs it from its stored values, scaled. A saved name is their earlier words, often with its portion: the portions are your reading of it against what they said. A value they state goes in `stated`, in place of the stored one. A saved dish's or meal's items can be scaled one by one with `parts` (`[{ position, scale }]`, scale 0 leaves it out; a meal item's position as `get_day` shows it).
   - **No match:** a food item with `description` (the food's name only: never the meal or the size in it), `quantity` and `quantityUnit` (the size, as they said it), every macro estimated (`energy` as `{ value, unit: "kcal" }`, `proteinG`, `fatG`, `carbG`, `fibreG`; zero is a value), `basis` (`coach_estimate`, `label`, `photo_estimate` or `stated`), `confidence` and `portionAssumption`.
   - `eatenAt` (HH:MM) or `dayPart` when they gave it, and `name` for which meal it was (Breakfast, Lunch, Dinner, Snack, or their own word) when they said.
3. **Say back** what was saved, and which items you estimated. Quote the totals the result gives.

- **The same as a past meal** ("same lunch as yesterday"): read it with `get_day` (or `get_history` with `type: "meals"`), then `repeat_meal` with `repeatOf` (the meal's kind, id and editCount), `existing` (its items' versions as read), `scale` (1 for the same amount) and `overrides` (`[]` unless one item changed).
- **A name to reuse** ("save this as my usual breakfast"): `save_repeat` with `name`, `target: null` and `reference` (`{ kind, id }` of the food, dish or meal). To rename one, `target` is its `{ id, editCount }`.
- **A photo or a label:** read it, then the same steps, with `basis` `photo_estimate` or `label`.

## A reading

**`log_body`** for a weight, a waist, a blood test value or any catalogue measurement: `measurement` (the catalogue's key, such as `bodyweight` or `waist`), `value` as they said it (`{ value, unit }`, or `{ stone, pounds }`, or `{ feet, inches }`), and `day`, the day it was measured. With no unit given, use the one they've logged it in before; ask only if their history doesn't settle it. A count or a rating goes bare. The first unit sent for a measurement with none becomes its unit: say it back. A named measurement (a rating of something) goes to `log_metric` with its `name`.

## Steps and sleep

**`log_metric`** with `measurement` (the key the summary shows for it), `day`, `value` (`{ value, unit }` as said, such as `{ value: 7.5, unit: "h" }`; a step count bare), `basis` (`stated` for what they said, `measured` for a device's reading or screenshot) and `components: []`. If it refuses because they have no selection of that measurement, log it with `log_body`, which adds one.

## A session

1. **Exercise ids,** when the session has sets: `get_chart_section` with `section: "plan"`; its `in_force` list is the program line by line, then each exercise with the id `log_session` takes. A set of an exercise the plan prescribes goes under the plan's own exercise, never a catalogue exercise of a like name. One outside the plan: `section: "exercises"`.
2. **`log_session`** with `day`, `status` (`completed`, `skipped` or `rest`; never `planned`), `basis`, `sets` (each `exerciseId`, `setIndex`, `basis`, and `reps`, `load`, `rir`, `duration`, `distance`, `side` as they gave them, with units), and `duration` as said.
   - **Completed,** on a day with workouts planned: `plannedWorkout` is the one it was, as `get_summary` or `get_day` shows it (a written session `{ id, editCount }`, or a scheduled slot `{ slotId, editCount }`), or `null` if it was none of them.
   - **Rest or skip,** on a day with workouts planned: `replaces` lists the ones it takes off (`[]` if none). "I'm resting today" names every one planned that day; a skip is one workout, so save one skip per workout. Say which came off. Moving a workout to another day is `change_planned_day` (see the `plan-change` skill), not a save of this session.
   - Light, moderate and hard minutes, when they matter: complete them from their words as your guess, and say it back.
3. **More sets for a session already saved:** `get_day` for the session's `{ id, editCount }` and every set's, then `log_sets` with `session`, `existingSets` (the complete list as read) and the new `sets`.

## A correction, or an entry to remove

1. **Read it first:** `get_day` for that day (or `get_history`), for the row's `{ id, editCount }`, and its parent's for a child row (a meal's item, a session's set).
2. **A wrong value:** `correct_capture` with `resource` (such as `meals`, `meal_items`, `training`, `sets`, `measurements`), `target`, `parent` for a child row, and `patch` with only the fields to change; a measurement in it as `{ value, unit }` as they said it. Say back the old value and the new.
3. **An entry that shouldn't be there:** `delete_capture` with `resource`, `target`, `parent` for a child row, and, for a row with children, `children` (its complete list as read). Say what was removed.

## A new allergy, medication, injury or condition

**`capture_safety`** at once, with `list` (`allergies`, `medications`, `injuries` or `health_conditions`) and `entries`: each `name`, `kind` (for allergies: `allergy`, `intolerance` or `coeliac_disease`; for medications: `medication` or `supplement`), `detail` in their own words, and `side`, `status` and `startedOn` only when they said. "None" is `none: true`. To change or remove a saved entry: `changes`, each with its `target` from the summary's safety part, and `remove: true` or the new `detail`, `status` or `stoppedOn`. Say back what was saved, then check anything you've suggested in this conversation against it.

## Questions

- **What's left today, or how today is going:** `report_today`: the figures the dashboard's Today page prints. Quote what is left as it gives it (`left`, or `over` when they're past the target), with the target and what they've had; never subtract one yourself. `get_day` with `{ "relative": "today" }` gives the same figures (`leftKcal`, `leftProteinG`, `leftFatG`, `leftFibreG`) with the day's meals and sets.
- **How they're doing against their goals, or what's coming up:** `report_progress`: the figures the dashboard's Goals & progress page prints. Each goal in their words with its measures: the latest reading, the current level, the change per week and each line with its gap (level less line). Quote those figures as given and say what they show; software gives the numbers and never says a goal is reached or a line crossed, so leave that judgement to them. It also holds the finished weeks, this week day by day and the dated events ahead. The readings behind each chart are left out: `get_history` with `type: "readings"` has them.
- **What the week ahead looks like:** `report_week_ahead`: the figures the dashboard's Week ahead page prints, for today and the next six days. The calories to eat (the plan's targets added), the estimated burn (their own observed daily average, the same on every day) and the gap with what it comes to in body weight, then each day's target, burn, deficit, planned sessions and estimated movement cost. Also the sessions counted against their weekly floor. Quote those figures as given; never work one out, and never add the movement costs to the burn. If the burn is empty, it says why. Each session's prescribed exercises and the daily block's list are left out: `get_day` with that date has them.
- **How recent weeks and days have gone:** `report_history`: the figures the dashboard's History page prints. Every week from the first with data (newest first, the week in progress marked): days counted, average weight and steps, sessions against the floor, eaten against the plan over the same days, estimated burn (a floor where a part is missing, with the days and parts), the deficit, and the balance the week would have had eating exactly to target. Then the last 14 days' totals. Quote those figures as given; never work one out. The chart series and each day's meals, sessions and sets are left out: `get_history` and `get_day` have them. If older weeks are named as left out, `get_history` with `type: "daily_totals"` and the range it names has them.
- **A trend:** `get_history` with `type: "readings"`, the `measurement` key and a `range` (`{ "lastDays": 28 }`, or `{ from, to }`). Quote the summary and the trend it returns; never count or add up the rows yourself.
- **Totals over a stretch:** `get_history` with `type: "daily_totals"` and a `range`.
- **What they did recently:** `get_recent_training` (`days` 1 to 7; 3 by default).
- **The findings:** `get_findings`, most severe first. Quote their captions.
- **The program, the nutrition plan, their values or a precommitment:** `get_document`. Any other part of the chart: `get_chart_section`.

## Saving the rest of what they tell you

Save each fact **at once, in their words, and say back what was saved** (the rules in CLAUDE.md). Which verb:

- **A trip, a social event or a race:** `capture_event` with its dates.
- **A practical constraint** (when they can train, where and with what, how they eat, a preference): `capture_constraint`, their words as the record; add only what they gave as a measured fact. To fix one: `correct_capture`.
- **A profile fact** (birth date, height, a measured resting metabolic rate, name, pronouns, when the current diet began): `update_profile`.
- **Their values or the story of an injury:** `update_document`, with their words and your short restatement.
- **A commitment they made to themselves:** `capture_precommitment`. **A rule they gave you to follow** ("no weigh-ins on Sundays until June"): `set_standing_instruction`, naming the goal it serves.
- **How they want you to talk to them** (length, tone, detail, headline first, whether to ask about meal times or steps): `set_communication_preferences`.
- **A note for them to act on:** `write_coach_note`. **Their own cue for an exercise:** `set_exercise_note`.
- **A goal:** `set_goal`. **The number that tells a goal is reached or has stopped working:** `set_trigger`. **What they track for it:** `select_measurements`. **A weekly-pattern setting** (a repeated miss, adherence below a percent, a weekly rate): `set_weekly_pattern`.
- **Something you promised or left open:** `set_follow_up`, and close it when it's answered.
- **A measurement, session type or exercise the catalogue doesn't hold:** `add_custom_entry`.

Rules for these:
- **Read the current version first.** Before changing or closing a stored item, read it with `get_chart_section` (or `get_document`) for its `{ id, editCount }`; never make one up.
- **Removing a goal or a trigger asks the person first.** Only on their yes, with their words quoted in `agreed`.
- **A threshold or a window they haven't stated stays blank.** Never invent one.
- **A plan change isn't a save:** changing the plan, the program, targets or the weekly schedule goes through the `plan-change` skill. Never keep any of this in a file.
