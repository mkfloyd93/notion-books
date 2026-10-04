## Reading Lifecycle

The reading workflow uses four Notion automations that respond to changes I make on the Book record. Rather than manually creating Reading Sessions or Reading Logs, I update the book's **Status** and **Progress**, and those changes trigger the appropriate automation.

<table>
<tr>
<td width="70%" valign="top">

<h3>Starting a Reading Session</h3>

<p>When I change a Book's Status to <code>Reading</code>, the start automation initializes everything needed to track that read.</p>

<p>It creates a new Reading Session and connects it to the Book. That session is also stored as the Book's <code>Active Reading Session</code>, which gives later automations a way to identify where new activity should be recorded.</p>

<p>The automation then creates the first Reading Log with a <code>Started</code> event at 0% progress. That log becomes the Book's <code>Last Log</code>, and the Book's Progress is set to 0%.</p>

<p>At this point, the system has established both pieces of temporary state it needs while I read:</p>

<ul>
<li><code>Active Reading Session</code> identifies the current read-through.</li>
<li><code>Last Log</code> identifies the most recently recorded progress point.</li>
</ul>

</td>
<td width="30%" valign="top" align="center">
<a href="images/automations-started.png">
<img src="images/automations-started.png" width="250" alt="Notion automation for starting a reading session">
</a>
</td>
</tr>
</table>

<table>
<tr>
<td width="70%" valign="top">

<h3>Recording Progress</h3>

<p>While I am reading, I manually update the Book's Progress percentage.</p>

<p>Each change triggers the progress automation. The automation uses <code>Last Log</code> to retrieve the previous percentage and compares it with the new Progress value to determine how much of the book I have read since the previous update.</p>

<p>For an audiobook, that percentage difference is multiplied by the Book's total runtime:</p>

<pre>(Current Progress - Previous Progress)
            × Total Runtime
                   =
     Estimated Minutes Listened</pre>

<p>The automation creates a new Reading Log containing the current percentage, calculated minutes, date, Book, and <code>Active Reading Session</code>.</p>

<p>After the log is created, it replaces the previous <code>Last Log</code>. This means the next Progress update can use the newly recorded percentage as its starting point.</p>

<p>The process repeats each time I update Progress.</p>

</td>
<td width="30%" valign="top" align="center">
<a href="images/automation-update-percentage.png">
<img src="images/automation-update-percentage.png" width="250" alt="Notion automation for recording reading progress">
</a>
</td>
</tr>
</table>

### Workflow in Action

These recordings demonstrate the two manual interactions that drive the automated reading workflow: changing a book's status and updating its reading progress.

<table>
<tr>
<td width="50%" align="center" valign="top">

**Starting a Reading Session**

<a href="images/reading-status-workflow.gif">
<img src="images/reading-status-workflow.gif" width="240" alt="Demonstration of changing a book's status to Reading in Notion">
</a>

Changing the book's status to `Reading` starts the automated tracking workflow.

</td>
<td width="50%" align="center" valign="top">

**Updating Reading Progress**

<a href="images/listening-progress-tracking.gif">
<img src="images/listening-progress-tracking.gif" width="240" alt="Demonstration of updating audiobook reading progress in Notion">
</a>

Updating progress