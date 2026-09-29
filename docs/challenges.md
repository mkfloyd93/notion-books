# Reading Challenges

The reading challenge system allows me to manage multiple reading challenges without maintaining separate book lists or manually checking off completed prompts.

Challenges are connected directly to the Books already stored in my library. Each challenge is divided into individual Prompts, which can contain multiple potential book choices while still identifying the book I am currently most likely to use.

Prompt and challenge progress are then calculated from the underlying reading data.

```text
Reading Challenge
       │
       ├── Prompt
       │     ├── Candidate Book
       │     ├── Candidate Book
       │     └── Top Book Pick
       │
       ├── Prompt
       │     ├── Candidate Book
       │     └── Top Book Pick
       │
       └── Prompt
             └── Candidate Book
```

This keeps challenge planning flexible while allowing completion to be determined from actual reading activity.

---

## Challenge Data Model

The challenge system uses two connected databases: **Reading Challenges** and **Prompts**.

### Reading Challenges

Each Reading Challenge represents a complete challenge and stores information including:

- challenge name and description
- start and end dates
- source URL
- tags
- related Prompts
- overall completion

The challenge itself does not need to track individual books. Instead, its Prompts handle the relationship between challenge requirements and Books.

### Prompts

Each Prompt represents one requirement within a Reading Challenge.

For example:

```text
Reading Challenge: Example Challenge

Prompt: Read a murder mystery
Prompt: Read a book involving food
Prompt: Read a book involving the realm of the dead
```

Prompts are related back to their Reading Challenge and can be connected to Books from the main Books database.

This means the same Book records used throughout the rest of the system can also be used for challenge planning and completion tracking.

<!-- SCREENSHOT: Reading Challenge with related Prompts -->

---

## Planning Books for a Prompt

A Prompt can be connected to multiple Books.

This is intentional. I often want several possible options for a challenge requirement rather than committing to a single book far in advance.

```text
Prompt
  │
  ├── Possible Book A
  ├── Possible Book B
  ├── Possible Book C
  │
  └── Top Book Pick
```

The `Books` relation stores all of the titles I am considering for the Prompt.

A separate `Top Book Pick` relation identifies the book I currently prefer without removing the other possibilities.

This lets me change my reading plans without having to restructure the challenge or lose the alternatives I had already identified.

<!-- SCREENSHOT: Prompt with several candidate Books + Top Book Pick -->

---

## Date-Aware Prompt Tracking

Prompt completion is not based only on whether one of its related Books has ever been read.

Each Prompt also receives the start and end dates of its Reading Challenge. The system compares those dates against the completion information from its related Books.

This allows it to distinguish between a Book that satisfies the Prompt **during the challenge** and a Book that happens to have been read at another time.

```text
Related Book
      │
      ▼
Was it completed?
      │
      ▼
Was completion within
the Challenge dates?
      │
      ├── Yes → Valid completion
      │
      └── No  → Does not complete Prompt
```

Because Book completion dates are already derived from Reading Logs, challenge tracking can reuse the reading activity captured elsewhere in the system rather than requiring a second completion record.

---

## Prompt Status

Prompt Status is calculated automatically using the challenge dates and the state of its related Books.

The formula can produce several states depending on the current situation.

### Completed

A Prompt is `Completed` when at least one related Book has been successfully completed within the Challenge's date range.

### In Progress

During an active Challenge, a Prompt can become `In Progress` when one of its related Books is currently being read.

### Not Started

If the Challenge is active but none of the related Books are currently being read or have completed the Prompt, it remains `Not Started`.

### Not Completed

Once the Challenge deadline has passed, a Prompt without a valid completion is marked `Not Completed`.

The resulting lifecycle is:

```text
                 Challenge Begins
                        │
                        ▼
                   Not Started
                        │
                 Book begins?
                    /       \
                  Yes        No
                   │          │
                   ▼          │
              In Progress     │
                   │          │
             Book completed?  │
                /      \      │
              Yes       No    │
               │         │    │
               ▼         └────┤
            Completed         │
                              ▼
                      Challenge Ends
                              │
                              ▼
                       Not Completed
```

This makes Prompt status a reflection of the underlying reading activity rather than another value I need to maintain manually.

---

## Calculating Challenge Progress

The Reading Challenge uses the statuses of its related Prompts to calculate overall completion.

Conceptually:

```text
Completed Prompts
───────────────── × 100
  Total Prompts
```

For example:

```text
12 Completed Prompts
──────────────────── × 100 = 60%
   20 Total Prompts
```

As individual Prompt statuses change, the Challenge's completion percentage updates with them.

The hierarchy therefore works in both directions:

```text
Reading Activity
       ↓
     Books
       ↓
    Prompts
       ↓
Reading Challenge
       ↓
Overall Completion
```

A reading event at the Book level can ultimately update the progress of a Reading Challenge without requiring me to manually mark either the Prompt or the Challenge complete.

<!-- SCREENSHOT: Challenge showing calculated completion -->

---

## Reusing the Core Book Library

One of the main design decisions behind the challenge system was **not creating a separate collection of Books for each challenge**.

Instead:

```text
                  Books
                /   |   \
               /    |    \
              ▼     ▼     ▼
         Prompt A Prompt B Prompt C
              │      │      │
              ▼      ▼      ▼
         Challenge Challenge Challenge
```

A Book can therefore be considered for multiple Prompts or multiple Challenges while remaining a single canonical record.

The same Book's Status and completion data can support:

- normal reading tracking
- planning
- analytics
- series tracking
- challenge completion

This avoids duplicating reading data across different parts of the system.

---

## Manual vs. Derived Data

The challenge workflow intentionally separates planning decisions from information the system can determine itself.

### I manage

I choose:

- which Reading Challenges I want to participate in
- the Prompts associated with each Challenge
- which Books could satisfy each Prompt
- which candidate is currently my `Top Book Pick`

### The system derives

Notion determines:

- whether a related Book is currently being read
- whether a Book was completed
- whether that completion occurred during the Challenge
- the resulting Prompt Status
- overall Challenge completion

This allows the subjective part of reading challenges — deciding what I might want to read — to remain flexible while objective progress tracking happens automatically.

---

## Design Decisions

### Keep planning flexible

Prompts can contain multiple candidate Books rather than forcing an early commitment to one title.

### Separate candidates from preference

`Top Book Pick` identifies my current preferred choice without removing other Books from consideration.

### Derive completion from reading activity

Completing a Book can satisfy a Prompt without requiring me to manually update the Prompt afterward.

### Respect challenge dates

A Book only counts toward completion when its reading activity meets the timing requirements of the Challenge.

### Reuse existing data

Challenges operate on the same Books and reading history used throughout the rest of the system rather than maintaining a separate tracking system.