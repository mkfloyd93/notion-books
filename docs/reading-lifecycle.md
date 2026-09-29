## Reading Lifecycle

The reading workflow uses four Notion automations that respond to changes I make on the Book record. Rather than manually creating Reading Sessions or Reading Logs, I update the book's **Status** and **Progress**, and those changes trigger the appropriate automation.

### Starting a Reading Session

When I change a Book's Status to `Reading`, the start automation initializes everything needed to track that read.

It creates a new Reading Session and connects it to the Book. That session is also stored as the Book's `Active Reading Session`, which gives later automations a way to identify where new activity should be recorded.

The automation then creates the first Reading Log with a `Started` event at 0% progress. That log becomes the Book's `Last Log`, and the Book's Progress is set to 0%.

At this point, the system has established both pieces of temporary state it needs while I read:

- `Active Reading Session` identifies the current read-through.
- `Last Log` identifies the most recently recorded progress point.

### Recording Progress

While I am reading, I manually update the Book's Progress percentage.

Each change triggers the progress automation. The automation uses `Last Log` to retrieve the previous percentage and compares it with the new Progress value to determine how much of the book I have read since the previous update.

For an audiobook, that percentage difference is multiplied by the Book's total runtime:

```text
(Current Progress - Previous Progress) × Total Runtime
                      =
             Estimated Minutes Listened
```

The automation creates a new Reading Log containing the current percentage, calculated minutes, date, Book, and `Active Reading Session`.

After the log is created, it replaces the previous `Last Log`. This means the next Progress update can use the newly recorded percentage as its starting point.

The process repeats each time I update Progress.

### Completing a Reading Session

When I finish a book, I change its Status to `Read`.

Because I may not have manually updated Progress to exactly 100% before finishing, the completion automation uses the last recorded percentage to calculate the remaining portion of the audiobook.

It creates a final `Finished` Reading Log at 100% and records the remaining listening time. The Book's Progress is then set to 100%.

Finally, `Active Reading Session` and `Last Log` are cleared. These properties are no longer needed because there is no active read, but the Reading Session and all of its Reading Logs remain as permanent historical records.

### Ending a Book Without Finishing

Changing Status to `Did Not Finish` triggers a separate automation.

A `Did Not Finish` Reading Log is created and connected to the Book's current Reading Session. The automation then clears `Active Reading Session` and `Last Log`, ending the active workflow while preserving the Reading Session and all activity recorded before the book was abandoned.

### The Complete Flow

The four automations work together as a lifecycle:

```text
MANUAL                          AUTOMATED

Status → Reading ────────────► Create Reading Session
                               Create Started Log
                               Set Active Reading Session
                               Set Last Log
                               Set Progress → 0%

Progress → New % ────────────► Retrieve previous %
                               Calculate progress delta
                               Calculate minutes listened
                               Create Reading Log
                               Replace Last Log

       [repeat progress updates while reading]

Status → Read ───────────────► Calculate remaining minutes
                               Create Finished Log
                               Set Progress → 100%
                               Clear active state

             OR

Status → Did Not Finish ─────► Create DNF Log
                               Clear active state

                                      ↓
                              Reading Session + Logs
                               remain as history
```

This design keeps the interaction with the system simple: I control **what I'm reading and how far I've progressed**, while the automations handle the activity records, calculations, relationships, and workflow state required behind the scenes.