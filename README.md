# 📚 Notion Books

A custom reading management and analytics system built with **Notion and Python** to manage my TBR, plan what to read next, automatically track reading activity, and analyze my reading over time.

> This project combines a relational Notion workspace, native Notion automations, Python-based metadata enrichment, and custom dashboards into a single reading system.

## Overview

I originally built this system because I wanted more control over my reading data than a traditional book-tracking app could provide.

Over time, it evolved into a connected system for:

- Managing a large TBR with structured book metadata
- Planning what to read using flexible, customizable views
- Tracking daily audiobook listening time
- Automating reading activity and progress calculations
- Supporting rereads without duplicating book records
- Managing series and reading challenges
- Analyzing reading habits through personalized dashboards

The system is designed around a simple idea: **automate repetitive data work while keeping subjective decisions flexible and manual.**

### How It Works

The overall book lifecycle looks like this:

```text
Goodreads
    ↓
Capture
    ↓
Enrich
    ↓
Plan
    ↓
Read
    ↓
Track
    ↓
Analyze
```

Books begin on my Goodreads Want to Read shelf, move through an enrichment workflow using Notion and Python, and eventually feed into planning, reading activity, and analytics.

---

## ✨ Key Features

### Daily Listening Time & Automated Reading Logs

I wanted a way to track how many minutes I spend listening to audiobooks each day, rather than only tracking completed books. To accomplish this, I introduced a **Reading Log** database to record individual reading events, progress percentages, and estimated listening time.

To avoid manually maintaining these records, I built four native Notion automations that respond to changes in a Book's **Status** and **Progress**.

When I update my progress, the system calculates the percentage completed since the previous update and uses the audiobook's total runtime to estimate minutes listened. It then creates a dated Reading Log, allowing me to track daily listening time and analyze my reading habits over time.

Starting, finishing, or abandoning a book also triggers automations that manage Reading Sessions and preserve reading history.

[Explore the reading lifecycle and automations](docs/reading-lifecycle.md)

### Reread Support Without Duplicate Records

A key design decision was separating **Books**, **Reading Sessions**, and **Reading Logs**.

Each Book has one canonical record containing its metadata. Every time I start reading that book, the system creates a new Reading Session, and individual reading events are recorded as Reading Logs associated with that session.

```text
Book
│
├── Reading Session #1
│   ├── Started
│   ├── Reached 25%
│   └── Finished
│
└── Reading Session #2
    ├── Started
    ├── Reached 40%
    └── Finished
```

This allows the system to preserve separate histories for multiple read-throughs without duplicating the book's metadata.

### Automated Goodreads Metadata Enrichment

Adding books to my library starts with a Goodreads URL rather than manually entering every property.

A Python notebook running in Google Colab identifies books needing enrichment, retrieves their metadata from Goodreads using Selenium and BeautifulSoup, and normalizes the results for my Notion database.

The workflow also checks the Authors database for existing records, creates missing authors when necessary, and updates the original Book through the Notion API.

I still manually manage subjective information such as reading priority, preferred season, and cover presentation.

[Explore the Goodreads integration](docs/goodreads-integration.md)

### Date-Aware Reading Challenges

Reading challenges use connected **Reading Challenges**, **Prompts**, and **Books** databases.

Each Prompt can contain multiple candidate books and a preferred `Top Book Pick`. Rather than manually marking prompts complete, Notion formulas evaluate whether a related book was completed within the challenge's date range.

The resulting Prompt statuses automatically contribute to the challenge's overall completion percentage.

[Explore the reading challenge logic](docs/challenges.md)

---

## 🖥️ The Notion Workspace

The Notion interface brings these workflows together through several pages designed for different aspects of reading management.

### Reading Dashboard

The main Reading page serves as the entry point to the system, bringing together the information and views I use to manage my reading.

<p align="center">
  <img src="docs/images/reading-dashboard.png" width="800" alt="Notion Reading Dashboard">
</p>

### Library

The Library serves as the administrative side of the system.

Newly captured books appear in a **Needs Info** view until their metadata has been enriched and I've completed any remaining manual setup. Once ready, books move into the main TBR workflow.

<p align="center">
  <img src="docs/images/library.png" width="800" alt="Notion Library">
</p>

### TBR

The TBR provides ways to browse unread books by genre, priority, season, series, and other metadata.

<p align="center">
  <img src="docs/images/tbr.png" width="800" alt="Notion TBR">
</p>

### Reading Planning

The Planning page supports a more flexible approach to deciding what to read next.

My current setup uses reading rotations, allowing me to organize potential books into reading cycles without changing the underlying database structure.

<p align="center">
  <img src="docs/images/planning.png" width="800" alt="Notion Reading Planning">
</p>

### Series Management

The Series database tracks reading progress, publication status, upcoming books, and related authors.

It also supports parent and sub-series relationships, allowing nested series to be represented while calculating progress across connected books.

The Series area includes a separate planning interface for my manhwa and webtoon backlog.

<p align="center">
  <img src="docs/images/series.png" width="800" alt="Notion Series Management">
</p>

### Reading Analytics

The Stats dashboard uses the same structured data collected throughout the system to display reading trends, including:

- Books read and books not finished
- Minutes listened
- Books completed over time
- Fiction vs. nonfiction
- Audience and genre breakdowns
- Authors

Because these statistics are derived from the same databases used for everyday reading management, I don't need to maintain a separate analytics dataset.

<p align="center">
  <img src="docs/images/stats.png" width="800" alt="Notion Reading Statistics">
</p>

---

## 🏗️ Technology & Architecture

### Tech Stack

| Technology | Purpose |
|---|---|
| **Notion** | Relational databases, dashboards, formulas, views, and native automations |
| **Python** | Metadata processing, normalization, and integration logic |
| **Notion API** | Querying, creating, and updating database records |
| **Selenium** | Browser automation for dynamically loaded Goodreads pages |
| **BeautifulSoup** | HTML parsing and metadata extraction |
| **Google Colab** | Cloud-based Python execution environment |
| **Goodreads** | Source for my TBR and book metadata |

### Data Architecture

The system is built around seven interconnected Notion databases:

```text
Books
│
├── Authors
├── Series
├── Reading Sessions
│   └── Reading Log
└── Prompts
    └── Reading Challenges
```

**Books** acts as the central record for each title, connecting book metadata to reading history, series information, and challenge planning.

The supporting databases have distinct responsibilities:

- **Authors:** Reusable author records shared across Books.
- **Series:** Book collections, publication tracking, and parent/sub-series relationships.
- **Reading Sessions:** Individual read-throughs of a Book.
- **Reading Log:** Historical reading events and estimated listening time.
- **Reading Challenges:** Challenge information, date ranges, and overall completion.
- **Prompts:** Individual challenge requirements connected to candidate Books.

The separation between permanent metadata, current reading state, and historical activity allows the system to support different workflows without duplicating records.

For a detailed breakdown of the database relationships and schema, see the [System Architecture documentation](docs/architecture.md).

---

## 📖 Technical Documentation

The `docs` directory contains more detailed explanations of the system's design and implementation.

| Document | Description |
|---|---|
| [System Architecture](docs/architecture.md) | Database structure, relationships, design decisions, and ERD |
| [Reading Lifecycle](docs/reading-lifecycle.md) | Reading Sessions, Reading Logs, progress calculations, rereads, and native Notion automations |
| [Goodreads Integration](docs/goodreads-integration.md) | Python scraping, metadata normalization, author matching, and Notion API integration |
| [Reading Challenges](docs/challenges.md) | Date-aware prompt completion and challenge progress |
| [Notion Formulas](docs/formulas.md) | Formulas used for reading tracking, series progress, and challenges |
| [Database Schema (DBML)](docs/notion-schema.dbml) | Technical representation of the seven connected databases |

### Source Code

The Python implementation for Goodreads metadata enrichment is available in:

[`automation/populate_new_books.ipynb`](automation/populate_new_books.ipynb)

The notebook is designed to run in Google Colab, with API credentials managed through Google Colab Secrets rather than stored in the repository.