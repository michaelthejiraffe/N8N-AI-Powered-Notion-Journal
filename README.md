# N8N-AI-Powered-Notion-Journal

# AI Personal Journal & Expense Tracker

An **n8n-based personal productivity and journaling automation** that allows you to record daily activities and expenses through Telegram, automatically categorises the information using Google Gemini, stores the structured data in n8n Data Tables, and generates an AI-powered daily journal in Notion.

The goal of this workflow is to make personal time tracking and journaling almost frictionless: instead of manually maintaining a journal or expense spreadsheet, you simply send a message to Telegram.

---

## ✨ Features

- 📱 **Telegram-based input**
  - Send journal entries and expenses directly through Telegram.
- 🤖 **AI-powered classification**
  - Google Gemini determines whether an incoming message is a journal entry or expenditure record.
- 📝 **Automatic journal logging**
  - Journal entries are stored with their timing and description.
- 💰 **Expense tracking**
  - Expenses are automatically separated into an item and monetary value.
- 📊 **Daily expense summaries**
  - The workflow calculates and formats the day's expenditure.
- 🧠 **AI-powered daily reflection**
  - Gemini analyses the day's activities and provides insights and recommendations.
- 📓 **Automatic Notion journal creation**
  - A new Notion database page is created containing the day's activities, insights, improvements, and expenditure.
- ✅ **Telegram confirmations**
  - The user receives confirmation after successfully recording an expense or journal entry.

---

# 🏗️ Architecture

The workflow consists of two primary pipelines:

```text
                    ┌──────────────────────┐
                    │      Telegram        │
                    │    User Message      │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │    Google Gemini     │
                    │  Message Classifier  │
                    └──────────┬───────────┘
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
                 ▼                           ▼
        ┌─────────────────┐        ┌─────────────────┐
        │ Journal Entry   │        │ Expenditure     │
        └────────┬────────┘        └────────┬────────┘
                 │                           │
                 ▼                           ▼
        ┌─────────────────┐        ┌─────────────────┐
        │ n8n Data Table  │        │ n8n Data Table  │
        │ Journal         │        │ Expenses        │
        └────────┬────────┘        └────────┬────────┘
                 │                           │
                 └─────────────┬─────────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │  Daily AI Analysis   │
                    │    Google Gemini     │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │       Notion         │
                    │   Automated Journal  │
                    └──────────────────────┘
```

---

# 🔧 Technologies & Tools

| Technology | Purpose |
|---|---|
| **n8n** | Workflow automation and orchestration |
| **Telegram** | User-facing interface for submitting records |
| **Google Gemini** | Message classification, summarisation and daily analysis |
| **n8n Data Tables** | Storage for journal entries and expenditure |
| **Notion** | Final destination for the generated daily journal |
| **JavaScript** | Data transformation and formatting inside n8n Code nodes |

The workflow uses several n8n LangChain nodes, including Gemini chat models, AI Agents, structured output parsers and an AI Agent Tool. fileciteturn0file0L23-L103

---

# 🔄 Workflow 1 — Telegram Data Entry

The primary entry point is the **Telegram Trigger**.

Whenever a Telegram message is received, the message is passed to a Google Gemini-powered AI Agent.

The classifier determines whether the message represents:

1. `Journal Entry`
2. `Expenditure Record`
3. `Income Record`
4. An empty value if the message does not fit the expected categories

The classifier is instructed to return one of these categories based on the content of the Telegram message. fileciteturn0file0L189-L201

### Example

A message such as:

```text
10:30 Studied Python for two hours
```

can be interpreted as a:

```text
Journal Entry
```

Whereas:

```text
Lunch 8.50
```

can be interpreted as:

```text
Expenditure Record
```

---

# 🤖 AI Classification

The classification stage uses:

- **Google Gemini Chat Model**
- **n8n AI Agent**
- **Structured Output Parser**

The structured parser expects an output containing a `category` field.

```json
{
  "category": "Expenditure Record"
}
```

This structured result is then passed into an n8n **Switch** node.

The Switch separates journal entries from expenditure records. fileciteturn0file0L240-L312

---

# 📝 Journal Entry Pipeline

When Gemini identifies a message as a `Journal Entry`, the message is passed through a JavaScript Code node.

The current implementation expects the first word of the message to represent the timing.

For example:

```text
14:30 Worked on my website
```

is transformed into:

```text
timing: 14:30
record: Worked on my website
```

The workflow then stores this information in the **Journal Data Table** together with the current date. fileciteturn0file0L358-L367

The Data Table contains fields for:

- `journal_entry`
- `date`
- `timing`

The corresponding insert operation maps the parsed values into these fields. fileciteturn0file0L609-L668

After recording the entry, Telegram sends a confirmation to the user.

---

# 💰 Expenditure Pipeline

If Gemini classifies the message as an:

```text
Expenditure Record
```

the workflow sends it through a JavaScript Code node that separates the description from the final value.

For example:

```text
Coffee 5.50
```

becomes approximately:

```text
item: Coffee
price: 5.50
```

The workflow then inserts the information into the expenditure Data Table.

The table stores:

- `Expenditure_Name`
- `date`
- `Cost`

The workflow maps these values automatically before inserting the record. fileciteturn0file0L546-L599

A Telegram confirmation is then sent to the user:

```text
You have successfully recorded your purchase...
```

---

# 📊 Daily Expenditure Summary

The workflow contains a second branch that runs on a schedule.

The **Schedule Trigger** runs at 23:00. fileciteturn0file0L4-L20

At this point, the workflow retrieves the day's expenditure records from the n8n Data Table.

The JavaScript formatting node loops through the records and generates a summary similar to:

```text
Item: Coffee | Price: $5.50
Item: Lunch | Price: $8.50
Item: Transport | Price: $3.00

Total Sum: $17.00
```

The calculated summary is then passed into the daily journal generation process. fileciteturn0file0L505-L514

---

# 🕐 Daily Activity Summary

The scheduled workflow also retrieves the day's journal records.

The journal records are formatted into a chronological text representation containing:

- Start timing
- End timing
- Duration
- Event

This information becomes the input for the daily AI analysis. fileciteturn0file0L518-L528

---

# 🧠 AI Daily Reflection

The central AI component analyses the events recorded throughout the day.

The AI Agent is instructed to:

1. Document the activities and events of the day.
2. Consider the duration of activities.
3. Evaluate whether the user is engaging in positive and productive habits.
4. Provide insights, learnings and experiences from the day.
5. Provide recommendations for using time more effectively.
6. Present the explanation chronologically.
7. Clearly separate different ideas.
8. Provide comparative examples.
9. Generate a positive and cheerful title.

The detailed analysis is performed through an AI Agent Tool powered by Google Gemini. fileciteturn0file0L58-L84

---

# 📦 Structured AI Output

The daily analysis uses a structured output parser.

The expected output contains four fields:

```json
{
  "Title": "Enter a catchy title that summarises the mood and events of the day!",
  "EventList": "Event Name",
  "Insights": "Here are the insights based on todays events",
  "Improvements": "Here are the improvements for today"
}
```

This makes the Gemini response predictable and allows the resulting fields to be inserted directly into Notion. fileciteturn0file0L93-L103

---

# 📓 Notion Integration

The final destination of the daily workflow is a Notion database.

The workflow creates a new database page containing:

### Flow of the Day

A chronological summary of the recorded activities.

### Insights

AI-generated observations about the user's day.

### Improvements and Recommendations

Suggestions generated by the AI based on the activities recorded.

### Expenditure

The formatted list of expenses and the calculated total.

The Notion page title is generated dynamically using the current date and the AI-generated title. fileciteturn0file0L106-L157

The resulting Notion page effectively becomes an **automatically generated daily journal**.

---

# ⏰ Scheduled Workflow

The daily reflection pipeline is triggered automatically by the n8n Schedule Trigger.

Current configuration:

```text
Trigger time: 23:00
```

At this time the workflow:

```text
Retrieve today's expenses
        ↓
Format expenses
        ↓
Retrieve today's journal entries
        ↓
Format activities
        ↓
AI analysis
        ↓
Generate structured reflection
        ↓
Create Notion journal page
```

The schedule and data retrieval nodes are connected directly to the two daily Data Tables. fileciteturn0file0L744-L760

---

# 🗄️ Data Storage

The workflow currently uses two n8n Data Tables.

## Journal Data Table

Stores:

| Field | Description |
|---|---|
| `journal_entry` | Description of the activity |
| `date` | Date of the activity |
| `timing` | Time associated with the activity |

## Expenditure Data Table

Stores:

| Field | Description |
|---|---|
| `Expenditure_Name` | Name/description of the purchase |
| `date` | Date of the purchase |
| `Cost` | Cost of the purchase |

The scheduled workflow retrieves records using the current date as the filter. fileciteturn0file0L385-L431

---

# 🔑 Required Credentials

To reproduce this workflow, you will need to configure your own credentials in n8n.

### Telegram

Required for:

- Receiving messages
- Sending confirmation messages

The workflow uses Telegram Trigger and Telegram message nodes. fileciteturn0file0L166-L184

### Google Gemini

Required for:

- Message classification
- Daily activity analysis
- Event title generation

Multiple Gemini chat model nodes are used throughout the workflow. fileciteturn0file0L42-L49

### Notion

Required for:

- Creating the final daily journal database page

The Notion node creates a database page and populates its content dynamically. fileciteturn0file0L150-L160

---

# 🚀 Installation

## 1. Install n8n

Set up an n8n instance using either:

- Self-hosted n8n
- n8n Cloud

## 2. Import the workflow

Import the provided n8n workflow JSON into your n8n instance.

## 3. Configure Telegram

Create a Telegram bot and connect its credentials to:

- `Telegram Trigger`
- `Send Confirmation`
- `Send Confirmation1`

## 4. Configure Google Gemini

Add your Gemini API credentials to the Gemini Chat Model nodes.

The workflow currently uses Gemini models through n8n's Google Gemini integration.

## 5. Configure Notion

Create a Notion integration and give it access to the database where the journals should be created.

Then connect the Notion credentials to:

```text
Create a database page
```

## 6. Configure Data Tables

Create two n8n Data Tables corresponding to the workflow:

### Journal

```text
journal_entry
date
timing
```

### Expenditure

```text
Expenditure_Name
date
Cost
```

Update the Data Table references in the imported workflow to point to your own tables.

## 7. Activate the workflow

Once credentials and Data Tables have been configured, activate the workflow.

You can then interact with the system through Telegram.

---

# 💬 Example Usage

## Record an activity

Send:

```text
09:00 Went to the gym
```

The workflow classifies this as a journal entry and stores the activity.

---

## Record another activity

```text
11:30 Worked on my programming project
```

The entry is stored with its timing and description.

---

## Record an expense

```text
Lunch 12.50
```

The workflow identifies this as an expenditure and records:

```text
Expenditure_Name: Lunch
Cost: 12.50
```

A Telegram confirmation is then returned.

---

# 🌙 End-of-Day Automation

At 23:00, the workflow automatically gathers the day's information.

For example:

```text
09:00 Went to the gym
11:30 Worked on my programming project
14:00 Studied
18:00 Met friends
```

along with:

```text
Lunch: $12.50
Transport: $4.00
Coffee: $5.00
```

Gemini then analyses the day's activities and generates:

```text
Flow of the Day
        ↓
Insights
        ↓
Improvements & Recommendations
        ↓
Expenditure Summary
```

The final result is stored as a new Notion journal page.

---

# 🧩 Workflow Components

The major n8n components used in the workflow are:

### Triggers

- Schedule Trigger
- Telegram Trigger

### AI / LangChain

- AI Agent
- AI Agent Tool
- Google Gemini Chat Model
- Structured Output Parser

### Logic

- Switch
- Merge
- JavaScript Code

### Storage

- n8n Data Tables
- Notion

### Communication

- Telegram

---

# ⚠️ Current Limitations

This workflow is designed around a specific input format and personal workflow.

### Expense format

The expenditure parser assumes that the **last part of the Telegram message represents the price**.

For example:

```text
Coffee 5.50
```

works naturally with the current implementation.

More complicated inputs may require additional parsing logic.

### Journal timing

Journal entries currently assume that the first space-separated portion of the message represents the timing.

For example:

```text
14:30 Worked on my website
```

becomes:

```text
timing = 14:30
record = Worked on my website
```

### AI classification

Classification is dependent on the Gemini model correctly interpreting the user's message.

Ambiguous messages may therefore require more explicit input.

### Personal configuration

The workflow currently references specific Notion and n8n Data Table resources from the original setup.

These should be replaced with your own resources when importing the workflow.

---

# 🔮 Possible Improvements

Some potential future improvements include:

- More robust natural-language expense parsing
- Support for income records
- Automatic currency detection
- Better duration extraction
- Automatic end-time calculation
- Calendar integration
- Weekly and monthly productivity reports
- Spending analytics
- Habit tracking
- Productivity scoring
- Graphs and dashboards
- Automatic weekly reviews
- Telegram commands such as `/journal`, `/expense`, and `/summary`
- More sophisticated AI-based categorisation
- Error handling for malformed messages

---

# 📁 Repository Structure

A simple GitHub repository could be structured as:

```text
.
├── README.md
├── workflow.json
└── screenshots/
    ├── workflow-overview.png
    └── notion-output.png
```

Where:

- `README.md` contains the documentation.
- `workflow.json` contains the exported n8n workflow.
- `screenshots/` contains optional screenshots demonstrating the workflow.

---

# 🔐 Security

**Do not commit API keys, bot tokens, credentials, database IDs, or other secrets to GitHub.**

Before publishing the workflow:

1. Remove or replace personal credentials.
2. Check exported workflow JSON for sensitive information.
3. Replace personal Notion/Data Table references where appropriate.
4. Verify that Telegram webhook or credential information does not expose private information.
5. Use environment variables or n8n credentials for secrets.

The exported workflow contains credential references, so it should be reviewed carefully before being made public.

---

# 📜 License

Choose an appropriate open-source license if you intend to allow others to reuse or modify the workflow.

For example:

```text
MIT License
```

---

# ⭐ Motivation

This project was created around a simple idea:

> **Make recording your life easier than forgetting it.**

Rather than spending time manually maintaining a journal, the workflow turns short Telegram messages into structured records and uses AI to transform those records into a meaningful daily reflection.

The end result is a personal feedback loop:

```text
Record
   ↓
Store
   ↓
Reflect
   ↓
Learn
   ↓
Improve
```

This makes the workflow more than an expense tracker or journal — it is intended to become a lightweight **personal productivity feedback system**.
