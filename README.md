# Mini Project 3: Smart Task Manager API – Activity Tracking & Insights

## Overview

In this assignment, you will continue improving your **Smart Task Manager API** from Mini Project 2.

Your application already supports task management, database persistence, filtering, searching, and pagination. Now, you will extend it by introducing **task activity tracking, activity history, and productivity insights**.

Imagine you want to understand what happened to a task over time. When was its status changed from `todo` to `in_progress`? How many times was its priority updated? When was it deleted?

Currently, your application stores the latest state of each task. In this project, you will extend it to record important actions and make those records accessible through new API endpoints.

You will also practice database aggregation, query performance, and turning functional requirements into working backend features.

You can implement this project using either a **relational database (SQLite)** or a **nonrelational database (MongoDB)**.

**Estimated completion time: 24 hours**

---

## Learning Goals

By completing this project, you will practice:

- Extending an existing FastAPI application without breaking previous functionality
- Working with relational or nonrelational databases
- Designing database tables or collections for new features
- Integrating database operations into API workflows
- Recording and retrieving task activity history
- Using database queries and aggregations to generate insights
- Measuring and improving database query performance
- Turning feature ideas into smaller implementation steps
- Using AI tools responsibly while writing and understanding your own code

---

## Project Requirements

### 1. Preserve Your Previous Implementation

You will continue working on your Mini Project 2 codebase.

Your existing task management features should remain functional, including:

- Creating, retrieving, updating, and deleting tasks
- Database persistence
- Filtering and searching
- Pagination
- Input validation and error handling

You should **not rebuild the application from scratch**. Extend your existing implementation with the new features below.

### 2. Choose Your Database

Choose **one** of the following approaches for storing tasks and activity records.

#### Option A: Relational Database — SQLite

You can use SQLite with SQLAlchemy or another suitable Python database library. Store activity records in a separate table.

```text
Database
├── tasks
└── task_activities
```

Each activity record should contain a `task_id` referencing the associated task's identifier. Design the relationship so that deleting a task **does not delete its activity history**. For example, be mindful of foreign-key constraints and cascade-delete rules.

#### Option B: Nonrelational Database — MongoDB

You can use MongoDB with PyMongo or another suitable Python database library. Store activity records in a separate collection.

```text
Database
├── tasks
└── task_activities
```

Each activity document should contain a `task_id` referencing its associated task's identifier.

#### Requirements for Both Options

- You may continue using the database from Mini Project 2 or switch to one of the options above.
- Use your selected database for **both tasks and activity records**.
- Store activity records permanently in the database.
- Do not use Python lists or dictionaries as permanent storage.
- Keep database credentials and configuration outside your source code.
- Ensure activity history remains available after its associated task has been deleted.

Both options have the same functional requirements.

### 3. Add Task Activity Tracking

Introduce an activity tracking feature that automatically records important task actions.

Whenever a task is created, updated, or deleted, your application should create an activity record.

An activity record could look like this:

```json
{
  "id": 1,
  "task_id": 12,
  "action": "updated",
  "timestamp": "2026-11-04T10:30:00Z",
  "changes": {
    "status": {
      "old": "todo",
      "new": "in_progress"
    },
    "priority": {
      "old": "medium",
      "new": "high"
    }
  }
}
```

Your implementation should:

- Record an activity when a task is created.
- Record an activity when a task is updated.
- Record an activity when a task is deleted.
- For updates, record which fields actually changed and their previous and new values.
- Include a timestamp for each activity.
- Record activities only for successful operations.
- Preserve activity records after the associated task is deleted.

You can choose how to store change details: for example, JSON text in SQLite or nested objects in MongoDB. Regardless of your database, keep the API response structure consistent.

### 4. Implement the Activity History API

Add an endpoint to retrieve activity history for a specific task.

| Method | Endpoint | Description |
|---|---|---|
| GET | `/tasks/{task_id}/history` | Retrieve the activity history of a task |

Example request:

```http
GET /tasks/12/history
```

Example response:

```json
{
  "task_id": 12,
  "activities": [
    {
      "id": 2,
      "action": "updated",
      "timestamp": "2026-11-04T10:30:00Z",
      "changes": {
        "status": {
          "old": "todo",
          "new": "in_progress"
        }
      }
    },
    {
      "id": 1,
      "action": "created",
      "timestamp": "2026-11-03T09:00:00Z",
      "changes": {}
    }
  ]
}
```

Requirements:

- Retrieve activity records using database queries.
- Return activities ordered from newest to oldest.
- Return an empty activity list when no records exist.
- Make history accessible even if the original task has been deleted.
- Return JSON-compatible identifiers and timestamps.

### 5. Activity Filtering and Pagination

Add an endpoint to explore activity records across all tasks.

```http
GET /activities
```

Example requests:

```text
/activities?action=updated
/activities?task_id=12
/activities?limit=10&offset=0
/activities?action=deleted&limit=5
```

Requirements:

- Filter by action: `created`, `updated`, or `deleted`.
- Filter by task ID.
- Support combining filters.
- Use a default pagination `limit` of `10` and a maximum of `50`.
- Use a default `offset` of `0`.
- Sort by timestamp, newest first.
- Perform filtering, sorting, and pagination in the database, rather than loading every record and processing it in Python.

Depending on your choice, you might use:

- **SQLite:** `WHERE`, `ORDER BY`, `LIMIT`, and `OFFSET`.
- **MongoDB:** `find()`, `sort()`, `limit()`, and `skip()`.

### 6. Productivity Insights Using Database Aggregation

Implement an endpoint for activity statistics.

```http
GET /activities/stats
```

Example response:

```json
{
  "total_activities": 120,
  "by_action": {
    "created": 40,
    "updated": 65,
    "deleted": 15
  },
  "most_updated_tasks": [
    {
      "task_id": 12,
      "update_count": 15
    },
    {
      "task_id": 7,
      "update_count": 10
    }
  ]
}
```

Requirements:

- Calculate the total number of activities.
- Calculate activity counts grouped by action.
- Identify the **three most frequently updated tasks**.
- Use database queries or aggregation operations for these calculations.
- Do not retrieve all activity records and manually count them using Python loops.

For SQLite, consider `COUNT()`, `GROUP BY`, `ORDER BY`, and `LIMIT`. For MongoDB, consider aggregation pipeline stages such as `$match`, `$group`, `$sort`, and `$limit`.

### 7. Database Query Performance

Investigate and improve the performance of at least one database query in your application. For example, examine the query that retrieves activities for a specific task.

Your work should include:

- Identifying one frequently used query.
- Measuring or examining its execution **before** optimization.
- Adding an appropriate database index.
- Examining its execution **after** optimization.
- Documenting what you observed.

You can use:

- **SQLite:** `EXPLAIN QUERY PLAN` and `CREATE INDEX` (or equivalent library features).
- **MongoDB:** `explain()` and `create_index()`.

In your README, briefly explain which query and index you selected, why you selected them, and what changed in the query execution results.

You do not need to achieve a specific speed improvement. The goal is to learn how to investigate and improve database queries.

### 8. Turn Requirements into an Implementation Plan

Before implementing the features, create a short technical plan in your README. Address these questions:

- What new functionality are you adding?
- What information needs to be stored?
- Which existing endpoints need to change?
- Which new endpoints will you introduce?
- How will you verify that the changes work?

Include **at least three implementation steps** and their expected outcomes.

### 9. Testing

Extend the automated tests from Mini Project 2. Cover at least the following:

- Creating a task produces an activity record.
- Updating a task produces an activity record with the correct changes.
- Deleting a task produces an activity record.
- Activity history is returned in the correct order.
- Activity history remains accessible after task deletion.
- Filtering by action works correctly.
- Filtering by task ID works correctly.
- Pagination returns the expected records.
- Activity statistics return correct counts.
- Invalid query parameters are handled properly.
- Existing task management features continue to work.

Use a separate test database or collection to avoid changing real application data. For example, use a temporary SQLite database or a dedicated MongoDB test database.

---

## Optional Bonus Features

You can implement one or more of the following after completing the core requirements.

### Bonus 1: Activity Timeline in the Frontend

Extend your existing interface to show a task's activity history.

```text
Task: Prepare Backend Assignment

November 4, 10:30
Status changed from todo to in_progress

November 3, 14:15
Priority changed from medium to high

November 3, 09:00
Task created
```

You can use Jinja2 templates or JavaScript with the Fetch API.

### Bonus 2: Activity Filtering by Date

Extend `/activities` with date filters.

```text
/activities?start_date=2026-11-01&end_date=2026-11-04
```

Use database queries to retrieve activities in the selected date range.

### Bonus 3: AI-Generated Activity Summary

If you implemented the AI feature in Mini Project 1, you can extend it to summarize recent activities.

```http
POST /activities/summarize
```

Example response:

```json
{
  "summary": "You created 5 tasks, completed 3 tasks, and updated the priority of 2 tasks."
}
```

Keep API keys secure, avoid sending sensitive data unnecessarily, handle AI API failures, and clearly identify AI-generated summaries. The core activity tracking features should still work without an AI service.

---

## Suggested Project Structure

You can extend your existing structure as follows:

```text
project/
├── app/
│   ├── main.py
│   ├── database.py
│   ├── models.py
│   ├── schemas.py
│   ├── routes/
│   │   ├── tasks.py
│   │   └── activities.py
│   ├── services/
│   │   └── activity_service.py
│   └── templates/
│       └── index.html
├── tests/
│   ├── test_tasks.py
│   └── test_activities.py
├── requirements.txt
├── .gitignore
└── README.md
```

You do not have to follow this exact structure. Depending on your database choice, you may need additional files for models, collections, or configuration. Keep the code easy to understand and build on your existing modules where appropriate.

---

## Estimated Time Breakdown

| Work Item | Estimated Time |
|---|---:|
| Review previous project and prepare implementation plan | 2 hours |
| Configure database and activity data structure | 3 hours |
| Implement automatic activity tracking | 5 hours |
| Implement activity history and filtering endpoints | 4 hours |
| Implement activity statistics using aggregation | 3 hours |
| Investigate and optimize query performance | 2 hours |
| Write and extend automated tests | 4 hours |
| Update README and clean up the project | 1 hour |
| **Total** | **24 hours** |

---

## Submission Guidelines

Your submission should include:

- Your updated GitHub repository from Mini Project 2
- A working FastAPI application with the new functionality
- SQLite or MongoDB storage for activity records
- Updated automated tests
- An updated README

Your README should explain:

- Which database you selected and why
- How to configure and connect to the database
- How to install dependencies and run the application
- How to run the tests
- How activity tracking works
- The new API endpoints and example requests
- Your implementation plan
- Your database performance investigation and findings

### AI Use Guidelines

AI tools such as ChatGPT can be useful learning resources, but **you are expected to write the project code yourself**.

**You may use AI tools to:**

- Ask how a Python method, library function, or database operation works.
- Look up syntax, parameters, and usage examples.
- Ask for explanations of programming concepts, error messages, and documentation.
- Discuss possible approaches to solving a problem before implementing your own solution.
- Get suggestions about what to investigate when debugging or testing.

**You should not:**

- Ask an AI tool to generate the complete assignment or individual project features for submission.
- Copy and paste AI-generated code into your project.
- Submit code that you cannot explain or did not implement yourself.
- Share API keys, passwords, private credentials, or sensitive application data in AI prompts.

If an AI tool shows you an example, use it to understand the concept, then write and test your **own implementation**. The purpose of this assignment is to build your programming skills through practice, not to assemble AI-generated solutions.

Include a short **AI Usage** section in your README explaining whether you used AI tools and, if so, what concepts, syntax, or errors you asked about.

---

## Important Notes

- Continue working in your Mini Project 2 repository.
- Do not remove or break previously implemented functionality.
- You can choose either SQLite or MongoDB.
- Keep activity records after associated tasks are deleted.
- Perform filtering and aggregation through database queries.
- Handle invalid input and database errors appropriately.
- Do not hardcode database credentials.
- Keep your code organized and readable.
- Write tests for new functionality.
- Commit your changes regularly with meaningful commit messages.
- Write your code yourself, using AI only as a learning and reference tool.

The goal of this assignment is not simply to add endpoints. It is to practice extending a working backend application while considering data persistence, correctness, maintainability, and performance.
