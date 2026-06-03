## NLEx — Natural Language to Excel

### Product Requirements Document

---

**Version:** 1.0 (Draft for stakeholder meeting)  
**Date:** June 4, 2026  
**Author:** Serafim Soldatov  
**Status:** Pending Approval  

---

### 1. Executive Summary

**NLEx** is a self-service data access tool that enables business users to formulate analytical questions in natural language and receive results as formatted Excel files — without writing SQL, without involving the data team, without waiting in a request queue.

**Problem.** A gap exists between decision-makers and those who can extract data. Business analysts and managers know what questions to ask but cannot independently retrieve answers from databases. Data teams are overloaded with repetitive ad-hoc requests. Data retrieval takes hours or days.

**Solution.** A single web interface where users describe their needs in plain language. The system handles query interpretation, dialect-aware SQL generation, execution, and packaging of results into a formatted Excel file. When ambiguity is detected, the system does not guess — it clarifies through a structured dialogue with the user.

---

### 2. Business Goals

| ID | Goal | Measurement |
|----|------|-------------|
| BG-1 | Reduce ad-hoc data retrieval time | From hours/days to minutes |
| BG-2 | Offload data team from routine requests | 40%+ reduction in ad-hoc tickets |
| BG-3 | Enable self-service data access for business users | Number of active users querying data without data team involvement |

---

### 3. User Personas

#### Primary Persona

| Persona | Description | Core Need | Frequency |
|---------|-------------|-----------|-----------|
| **Business / Product Analyst** | Sales, marketing, finance, or product professional. Deeply understands business questions. Builds reports in Excel. SQL is a barrier, not a skillset | Get answers to analytical questions today to make decisions today. Avoid waiting for the data team. Self-serve and iterate in Excel | Daily, up to several times per day |

This is the persona we design for. Every product decision starts with their workflow.

#### Secondary Persona

| Persona | Description | Core Need | Frequency |
|---------|-------------|-----------|-----------|
| **Data Team Analyst** | Member of the data team who can write SQL and already has tools for query reuse. Their real problem is not writing queries — it's being interrupted to write queries for others | Redirect routine business requests to self-service. Validate generated SQL once, let business users reuse. Free up time for complex analytical work | Several times per week — primarily template creation and SQL review |

The value proposition for this persona is not "write SQL faster." It is "write SQL for other people less often."

---

### 4. Product Champion

**Head of Analytics / Lead Data Analyst**

Not a primary user, but the key figure for product adoption:

- Sees the full volume of ad-hoc requests and their impact on team velocity
- Owns time-to-answer as a personal or team KPI
- Wants the team solving complex problems, not writing repetitive SELECT statements
- Has the authority to change internal processes: "routine requests go through NLEx, data team handles the rest"
- Is directly incentivized for successful adoption: it improves their team's effectiveness and their own metrics

---

### 5. User Scenarios

#### Scenario 1: Simple Query (No Ambiguity)

1. Business Analyst opens NLEx
2. Selects "CRM Production" database from the dropdown
3. Types: "Top 10 customers by order amount for March 2026"
4. Clicks "Find"
5. System analyzes schema, locates `customers` and `orders` tables, finds no ambiguity
6. Generates dialect-appropriate SQL, executes query, formats result
7. User sees data preview: customer name, order total
8. Clicks "Download Excel" — receives formatted file

**Expected time:** 10–15 seconds from "Find" click to result.

#### Scenario 2: Query with Ambiguity

1. Business Analyst types: "Show sales for last month"
2. System detects that "sales" matches both `sales.orders` and `erp.invoices`
3. User sees structured clarification cards:
   - **"CRM Orders"** — table `sales.orders`, column `total_amount`, order amounts excluding returns
   - **"ERP Invoices"** — table `erp.invoices`, column `invoice_amount`, invoiced amounts including VAT
4. User selects "CRM Orders"
5. System asks a second clarifying question: "Last month" — calendar month (May 1–31) or rolling 30 days?
6. User selects "Calendar month"
7. System generates SQL incorporating all clarifications, executes, returns result

**Expected time:** 20–30 seconds including user responses.

#### Scenario 3: Cross-Database Query (MVP 2.0)

1. Business Analyst types: "LTV of customers acquired through Facebook campaigns Q1 2026"
2. System identifies customer and order data in Oracle, campaign data in ClickHouse
3. LLM decomposes query into subqueries; system executes them in parallel
4. Results are merged on `client_id = user_id`
5. User receives a single Excel file

**Expected time:** 20–40 seconds.

---

### 6. Functional Requirements

| ID | Feature | Priority | Description |
|----|---------|----------|-------------|
| F-1 | Database connection | P0 | Data Engineer adds connection via admin panel: DB type, host, port, name, credentials. "Test Connection" button with color indicator |
| F-2 | Schema introspection | P0 | Automatic extraction of tables, columns, data types. Explicit FK constraints. Heuristic detection of implicit relationships via naming conventions |
| F-3 | Semantic schema search | P0 | System finds relevant tables and columns by user query, even when database naming does not literally match search terms |
| F-4 | Ambiguity detection | P0 | System identifies when a term matches multiple tables, columns, or possible interpretations |
| F-5 | Clarification dialogue | P0 | User sees structured choice cards. Each card shows: option name, source (table and column), brief description. Up to 3 clarification rounds |
| F-6 | SQL generation | P0 | LLM generates SQL based on user text, schema context, and dialect rules of the target database. SELECT-only |
| F-7 | SQL validation | P0 | Syntax check of generated query. Verification of all tables and columns against actual schema. DDL/DML rejection. Up to 2 retries on error |
| F-8 | SQL execution | P0 | Query executed via connection pool with 30-second timeout. Maximum 100,000 returned rows |
| F-9 | Excel export | P0 | .xlsx file with two sheets: "Data" — query result with auto-filters and formatting, "Info" — original request, generated SQL, data source, execution timestamp |
| F-10 | Excel auto-formatting | P0 | Numeric columns: thousand separators, two decimal places. Dates: unified format. Booleans: "Yes"/"No". Auto-width columns. Frozen header row |
| F-11 | Excel download | P0 | Single-use presigned download URL, valid 1 hour. File auto-deleted from storage after 24 hours |
| F-12 | Query history | P1 | List of user's recent queries with option to rerun and re-download results |
| F-13 | Authentication | P1 | Password or OAuth login. JWT access and refresh tokens |
| F-14 | Admin panel | P1 | Database connection management, extracted schema viewer, basic usage statistics |

---

### 7. MVP Scope

#### MVP 1.0 — In Scope

- Single database connection at a time
- Support for PostgreSQL, Oracle, ClickHouse
- Automatic schema introspection with relationship detection
- Natural language query input field
- Ambiguity detection and clarification dialogue
- Dialect-aware SQL generation
- Query execution and result retrieval
- Formatted Excel export (.xlsx)
- File download via secure link
- User query history
- Basic authentication
- Admin panel for connection management

#### Explicitly Out of Scope for MVP 1.0

- Simultaneous cross-database queries
- Query templates with parameters
- Template sharing between users
- Data visualization (charts, graphs, dashboards)
- Scheduled automated reports
- Mobile version
- Row-level security by user role
- NoSQL support (MongoDB, Redis, Elasticsearch)

#### MVP 2.0 — Planned

- Cross-database queries: single natural language question → data from two or more databases simultaneously, with automatic result merging
- Query templates: save frequently used questions with parameters (period, region, product)
- Template sharing: analyst creates template, business users execute it with parameter substitution

---

### 8. Non-Functional Requirements

| Category | Requirement | Target |
|----------|-------------|--------|
| **Performance** | Time from "Find" click to result (no clarification) | < 15 seconds (P95) |
| **Performance** | Ambiguity detection response time | < 3 seconds |
| **Performance** | Excel generation for 10K rows | < 5 seconds |
| **Security** | Database credentials | Encrypted (AES-256) |
| **Security** | Database access | Read-only user |
| **Security** | Data sent to LLM | Schema only, no row data |
| **Security** | Transport channel | HTTPS (TLS 1.3) |
| **Reliability** | System uptime (MVP) | 99.5% |
| **Reliability** | Behavior on LLM failure | User-facing message, system does not crash |
| **Reliability** | Behavior on database failure | Descriptive message, system does not crash |
| **Usability** | Learning curve | User understands the interface within 5 seconds |
| **Usability** | Interface language | Russian and English |
| **Usability** | Error messages | Human-readable, no technical stack traces |

---

### 9. Success Metrics

| Metric | Target (MVP) |
|--------|--------------|
| Successful query completion rate (no manual intervention) | > 70% |
| Average query execution time (excluding clarifications) | < 15 seconds |
| Queries requiring clarification | < 40% |
| Average clarification depth (rounds) | < 1.5 |
| Active users | > 5 (for academic project: full team + reviewers) |

---

### 10. Assumptions & Dependencies

#### External Dependencies

- LLM API (OpenAI, Anthropic, or self-hosted model)
- Network access from NLEx server to customer databases
- Read-only database credentials

#### Key Assumptions

| Assumption | Risk if Invalid |
|------------|-----------------|
| Database tables and columns have meaningful names | Unreadable names prevent LLM from matching them to user queries |
| Relevant schema subset fits within LLM context (≤ 10 tables) | Queries requiring 20+ tables reduce SQL generation accuracy |
| LLM is available with < 3s latency for generation | System cannot process queries if LLM is unavailable |
| Users are willing to respond to clarification prompts | Satisfaction decreases if users perceive clarification as friction |
| Read-only database access is sufficient | Write requirements demand a more complex security model |

---

### 11. Explicitly Out of Scope

- Data visualization (charts, graphs, dashboards)
- Scheduled reporting (cron, recurring jobs)
- Mobile application
- NoSQL database support
- Collaborative editing
- BI system integration (Tableau, Power BI)
- Row-level security by user role
- Natural Language → non-SQL query generation (API, NoSQL)

---
