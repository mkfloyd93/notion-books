# Notion Formulas

This document describes the formulas used throughout the Notion Books system.

Formulas support reading activity tracking, audiobook listening time,
series progress, and reading challenge completion. They reduce manual
data entry by calculating values from existing properties and related records.

## Books

### Time (total)

**Description:** Converts an audiobook's duration from hours and minutes
into total minutes. This provides a consistent value for calculating
listening time from reading progress.

```javascript
(prop("Time (hours)")*60) + prop("Time(minutes)")
```

### Minutes Read label

**Description:** Converts the total minutes logged into a readable
hours-and-minutes label.

```javascript
let(
  h, floor(Minutes Read / 60),
  m, Minutes Read % 60,
  ifs(h > 0, h + " hour" + if(h > 1, "s", ""), "") +
  if(h > 0 and m > 0, " and ", "") +
  ifs(
    m > 0, m + " minute" + if(m > 1, "s", "") + " listened",
    h > 0, " listened",
    "0 minutes listened"
  )
).style("blue")
```

### Started Formula

**Description:** Retrieves the date of the most recent `Started` event
from the Book's related Reading Logs. This supports rereading without
creating duplicate Book records.

```javascript
prop("Reading Log").filter(current.prop("Name") == "Started").last().prop("Date")
```

### Completed Formula

**Description:** Retrieves the date of the most recent completion or
DNF event, depending on the Book's current reading status. This date
is also used when evaluating reading challenge requirements.

```javascript
if(
  prop("Status") == "Read",
  prop("Reading Log").filter(current.prop("Name") == "Finished").last().prop("Date"),
  if(
    prop("Status") == "Did Not Finish",
    prop("Reading Log").filter(current.prop("Name") == "Did Not Finish").last().prop("Date"),
    fromTimestamp(toNumber(""))
  )
)
```

## Series

### Books Read

**Description:** Counts books marked as Read in the current Series,
then adds the number of books read in its related Sub Series.

```javascript
length(
  replaceAll(
    replaceAll(
      replaceAll(prop("Book Statuses"), "To Read", ""),
      "Read\b",
      "*"
    ),
    "[^*]",
    ""
  )
) + prop("Sub Series Book Statuses")
```

### Total Books

**Description:** Calculates the total number of books in a Series,
including books belonging to related Sub Series.

```javascript
prop("Current Series Book Count Formula") + prop("Sub Series Book Count")
```

### Status Text

**Description:** Displays the Series' reading progress as a visual
progress bar and percentage. Fully completed, published Series receive
a checkmark, while Series with all currently available books read
display an "Up to date!" message.

```javascript
if(
  (prop("Fully Published") == true) and (prop("Books Read") == prop("Total Books")),
  "✅",
  if(
    prop("Books Read") == prop("Total Books"),
    "Up to date!",
    (
      (
        substring("■■■■■■■■■■", 0, floor((prop("Books Read") / prop("Total Books")) * 10))
        + substring("□□□□□□□□□□", 0, ceil(10 - ((prop("Books Read") / prop("Total Books")) * 10)))
      )
      + " "
      + format(round((prop("Books Read") / prop("Total Books")) * 100))
    ) + "%"
  )
)
```

## Reading Challenges

### Progress

**Description:** Calculates the percentage of completed Prompts
within a Reading Challenge. The resulting number can be displayed
using Notion's native progress bar visualization.

```javascript
let(
  /* Get the list of statuses from the related prompts */
  statuses, prop("Prompts").map(current.prop("Status")),

  /* Count how many are "Completed" */
  completedCount, statuses.filter(current == "Completed").length(),

  /* Calculate the percentage */
  totalPrompts, statuses.length(),
  percent, if(totalPrompts > 0, completedCount / totalPrompts, 0),

  /* Display as a Progress Bar */
  percent * 100
)
```

## Prompts

### Status

**Description:** Automatically determines whether a reading challenge
Prompt is Completed, Not Started, In Progress, or Not Completed.

The calculation evaluates completion dates from related Books against
the challenge's date range. It also checks whether any related Book
is currently being read.

```javascript
let(
  /* 1. Extract the challenge dates */
  challenge, prop("Challenge Dates").first(),
  startDate, challenge.dateStart(),
  endDate, challenge.dateEnd(),

  /* 2. Get completion dates from related Books */
  allDates, prop("Books").map(current.prop("Completed Formula")).filter(!empty(current)),

  /* 3. Get reading statuses from related Books */
  allStatuses, prop("Books").map(current.prop("Status")),

  /* 4. Check for valid completions within the range */
  validCompletions, allDates.filter(current >= startDate && current <= endDate),

  /* 5. Check whether any related Book is being read */
  isReading, allStatuses.includes("Reading"),

  /* 6. Determine Prompt status */
  ifs(
    /* Completed: Found a valid completion date */
    validCompletions.length() > 0,
    style("Completed", "b", "green", "green_background"),

    /* Not Started: Before start date or no active reading */
    now() < startDate || (now() >= startDate && now() <= endDate && !isReading),
    style("Not Started", "b", "grey", "grey_background"),

    /* In Progress: Within date range and currently reading */
    now() >= startDate && now() <= endDate && isReading,
    style("In Progress", "b", "blue", "blue_background"),

    /* Not Completed: Deadline passed without completion */
    now() > endDate && validCompletions.length() == 0,
    style("Not Completed", "b", "red", "red_background"),

    "Unknown"
  )
)
```