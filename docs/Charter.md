# Project Charter: Recipe and Smart Pantry Platform

**Course:** BBF 302E Database Systems  
**Team:** CHAOS  
**Term:** Fall 2026  
**Charter version:** 1.0

## 1. Project Purpose

The Recipe and Smart Pantry Platform is a web application that helps individuals track food stored at home and turn that inventory into practical cooking, shopping, and consumption decisions. Users maintain pantry items as separate physical lots with quantities, units, storage locations, and expiration dates. Keeping lots separate ensures that a newly purchased package does not hide the age or remaining quantity of an older package of the same ingredient.

The project addresses three connected problems:

- Household food is difficult to track accurately when quantities and expiration dates are spread across packages and storage locations.
- Meal planning requires users to compare recipe requirements with available ingredients manually.
- Shopping and consumption records are often disconnected from the pantry, which can produce duplicate purchases, inaccurate stock, and avoidable waste.

The intended outcome is a working academic web application backed by a normalized relational database. It will connect pantry inventory, recipes, dietary preferences, shopping lists, purchase confirmation, and consumption history while preserving ownership, transaction integrity, and traceable data relationships.

## 2. Objectives and Success Criteria

The project will:

1. Maintain accurate pantry balances for separately dated and located food lots.
2. Identify available, expiring, and expired inventory without mixing incompatible units or owners.
3. Match visible recipes against a member's pantry quantities and dietary preferences.
4. Calculate recipe shortages and add or merge those shortages into shopping lists.
5. Confirm purchases idempotently by creating pantry stock and updating the corresponding shopping item in one transaction.
6. Record cooking and direct consumption atomically, including exact deductions from individual pantry lots.
7. Provide searchable consumption history and monthly summaries by ingredient and recipe.
8. Enforce authentication, role permissions, record ownership, validation, and parameterized SQL throughout the application.

The project will be considered successful when the following conditions are met:

| Area | Acceptance criterion |
| --- | --- |
| Core workflows | Account, pantry, recipe, matching, shopping, purchase, consumption, and reporting workflows operate end to end. |
| Data integrity | Foreign keys, unique constraints, quantity checks, transactions, and row locks prevent orphaned records, duplicates, partial updates, and negative stock. |
| Security | Passwords use Argon2id hashes; protected operations validate the signed session token, active account, role, ownership, CSRF token, and input parameters. |
| Inventory performance | On the documented 100,000-row pantry fixture, inventory search and expiration queries target a p95 server response time below 500 ms over 100 warmed requests. |
| Recipe performance | On 200 recipes and a separate 50,000-line recipe fixture, paginated recipe matching targets a p95 response time below one second over 100 measured requests. |
| Usability | At mobile and desktop widths, five peer testers can each add food, find a recipe, and create a shopping list within three minutes per task without guidance. |
| Reliability | Failure-injection and concurrency tests demonstrate atomic purchase and consumption behavior, idempotent purchase confirmation, and nonnegative stock. |
| Reproducibility | A clean installation can apply numbered migrations, generate fixed-seed data, run the test suite, restore a database backup, and reproduce key row counts and aggregates. |

## 3. Stakeholders and Team Roles

### Academic stakeholders

- **Course instructors and teaching assistants:** define academic requirements, review milestone submissions, and evaluate the final database and application.
- **Team CHAOS:** designs, implements, tests, documents, and demonstrates the platform.
- **Target users:** individuals who cook for themselves and manage their own groceries.

### Application roles

| Role | Interest and authority |
| --- | --- |
| Guest | Browse published recipes and ingredient summaries, view recipe details, and register or sign in. |
| Member | Manage their own preferences, pantry lots, recipes, shopping lists, purchases, and consumption history. |
| Admin | Maintain the ingredient catalog, moderate published recipes, and manage account status without bypassing private-record ownership rules. |

### Team responsibilities

| Member | Project responsibility | Primary module ownership |
| --- | --- | --- |
| Can Sei Keçeci | Team leader and backend developer | Accounts and preferences; FR-01–03 and FR-19 |
| Hamza Çevikbaş | Authentication and data-safety contributor | Ingredients and pantry; FR-04–06 and FR-20 |
| Ahmet Abalı | Frontend developer | Recipes and matching; FR-07–10 |
| Ömer Türk | Backend developer | Shopping and purchase confirmation; FR-11–14 |
| Selim Efe Aytemur | Database designer | Consumption and reporting; FR-15–18 |

**Repository:** [smart-pantry-platform](https://github.com/kececi24/smart-pantry-platform)

## 4. Project Scope

### In scope

The initial delivery includes:

- Registration, login, logout, account maintenance, role checks, account disabling, and token invalidation.
- Member preferences for expiration alerts, recipe sorting, and dietary tags.
- A canonical ingredient and measurement-unit catalog maintained by administrators.
- Personal pantry lots with quantities, units, storage locations, purchase dates, and expiration dates.
- Inventory views for available, expiring, and expired items, including aggregation across compatible lots.
- Recipe drafting, ingredient lines, serving scaling, dietary tags, publication, search, and moderation.
- Rule-based recipe matching that distinguishes fully cookable recipes from recipes with shortages and prioritizes expiring stock.
- Personal shopping lists with manual items, recipe-shortage generation, purchase flags, and confirmed-purchase links to pantry lots.
- Atomic cooking and direct-consumption recording using first-in, first-out lot allocation where applicable.
- Consumption history, correction and deletion with stock restoration, and monthly summaries.
- Responsive web pages for mobile and desktop use.
- A MySQL schema, numbered migrations, deterministic seed generator, tests, benchmark fixtures, setup documentation, and backup/restore evidence.

### Out of scope

The following items are excluded from the core delivery:

- Shared household pantries or multi-owner inventory.
- Retailer, delivery-service, or live price integrations.
- Barcode scanning or image-based ingredient recognition.
- Nutrition calculations and medical or dietary advice.
- Machine-learning recommendation models.
- Native mobile applications.
- Offline synchronization.
- Automatic unit conversion across incompatible measurement dimensions.
- Caching infrastructure before indexed database queries and pagination have been optimized.

Any out-of-scope feature requires an explicit scope review and must not delay the required database, transaction, security, or testing work.

## 5. Major Deliverables

| Deliverable | Description |
| --- | --- |
| Project proposal | Problem definition, functional and non-functional requirements, technology plan, and initial work distribution. |
| ER diagram and relational design | Entities, cardinalities, 14-table relational mapping, keys, constraints, data types, and query planning. |
| Database implementation | MySQL/InnoDB schema, indexes, constraints, and versioned raw-SQL migrations. |
| Backend application | Flask services and endpoints using handwritten parameterized SQL and explicit transactions. |
| Web interface | Responsive HTML, CSS, JavaScript, Flask templates, and fetch-based interactions for the core workflows. |
| Security implementation | Argon2id password handling, signed JWT cookie sessions, CSRF protection, role checks, and owner-scoped access. |
| Seed and benchmark data | Fixed-seed functional data plus large fixtures for pantry and recipe-query performance tests. |
| Verification evidence | Unit, integration, authorization, failure-injection, concurrency, usability, and performance test results. |
| Operations documentation | Setup, migration, seed, test, backup, restore, and demonstration instructions. |

## 6. Technical Approach

### Application stack

- **Database:** MySQL with InnoDB, foreign keys, unique and check constraints, decimal quantities, transactions, row locks, indexes, CTEs, and window functions.
- **Backend:** Python and Flask with `mysqlclient` and a small connection pool.
- **Data access:** Handwritten, parameterized raw SQL; no ORM or automatic schema/code generator.
- **Frontend:** HTML, CSS, JavaScript, Flask templates, and fetch-based interactions.
- **Authentication:** Argon2id password hashes and signed JWT access tokens stored in HttpOnly cookies with a 30-minute expiration.
- **Request protection:** Secure cookies in deployment, `SameSite=Lax`, CSRF tokens for mutations, account-status checks, and token-version invalidation.
- **Data generation:** Fixed-seed Python generator using fictional accounts, ingredients, recipes, pantry lots, lists, and consumption records.

### Data-design principles

- Store each physical pantry lot independently.
- Represent reusable units and dietary tags as reference data.
- Resolve many-to-many relationships with explicit junction tables.
- Use decimal arithmetic and validate unit compatibility before mutation.
- Scope private reads and writes by the authenticated user's identifier.
- Use restrictive foreign keys unless a documented lifecycle requires another action.
- Preserve exact per-lot consumption deductions so corrections can restore stock.
- Separate marking a shopping item as purchased from confirming its delivery into a pantry lot.
- Use consistent lock ordering and atomic transactions for multi-row inventory changes.

## 7. Milestones

| Milestone | Evidence or outcome | Status / target |
| --- | --- | --- |
| Project proposal | Scope, requirements, roles, architecture direction, and work distribution | Completed |
| ER diagram and data-model design | Entities, relationships, 14 planned tables, constraints, CRUD plan, and complex-query plan | Completed |
| Mandatory TA meeting | All team members attend the scheduled project meeting | November 30–December 4, 2026 |
| Schema and seed foundation | Numbered migrations, constraints, indexes, lookup data, and deterministic generator | Course implementation schedule |
| Core module implementation | Account, pantry, recipe, matching, shopping, purchase, consumption, and reporting workflows | Course implementation schedule |
| Integration and security verification | Ownership, authorization, transaction rollback, retry, and concurrency tests | Before final demonstration |
| Performance and usability validation | Benchmark fixtures, query plans, response-time results, responsive checks, and peer-task results | Before final demonstration |
| Final documentation and demonstration | Reproducible setup, test evidence, backup/restore instructions, and end-to-end demo | Course submission schedule |

## 8. Assumptions and Constraints

- The application serves personal pantries; every private record belongs to one authenticated user.
- MySQL is the authoritative persistent store.
- All database queries are written in raw SQL.
- The team uses fictional or generated data and does not depend on external datasets or real personal records.
- The initial release is a responsive web application, not a native mobile application.
- The team has five members with module ownership defined in this charter.
- Course deadlines, submission formats, and instructor feedback take precedence over internal planning.
- All members must attend the mandatory TA meeting during November 30–December 4, 2026.

## 9. Key Risks and Mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Concurrent requests consume the same remaining stock | Negative or inconsistent pantry balances | Lock eligible lots in a consistent order, recheck balances inside the transaction, and run two-request concurrency tests. |
| A multi-step purchase or consumption operation fails partially | Pantry, shopping, and history records disagree | Wrap related writes in one transaction and use injected failures to verify complete rollback. |
| A retried purchase creates duplicate stock | Incorrect pantry quantity | Store the confirmed pantry-lot link on the shopping item and return that link on repeated confirmation. |
| Unit conversion produces invalid comparisons | Incorrect shortages or deductions | Store measurement dimensions and base multipliers, reject incompatible units, and test deterministic conversion fixtures. |
| Owner filters or role checks are omitted | Unauthorized data access | Centralize authenticated-user context, require owner-scoped SQL, and test guessed identifiers and cross-account requests. |
| Large fixtures make matching or inventory queries too slow | Performance targets are missed | Add indexes based on access paths, inspect plans with `EXPLAIN`, paginate results, and benchmark before final integration. |
| Module interfaces diverge during parallel work | Integration delays and schema conflicts | Treat migrations and requirement IDs as shared contracts, review pull requests, and integrate in small increments. |
| Optional features expand the workload | Core requirements remain incomplete | Keep the out-of-scope list binding until the core acceptance criteria and test evidence are complete. |

## 10. Governance and Change Control

- The team leader coordinates integration, milestone readiness, and unresolved cross-module decisions.
- Module owners implement and test their assigned requirements, but schema and transaction changes require team review because they affect shared data.
- Every schema change is introduced through a numbered migration and reflected in the data model and setup instructions.
- A requirement change must update its functional or non-functional requirement mapping, affected tables, responsible member, tests, and documentation.
- Pull requests should include a concise description, migration impact, test evidence, and any follow-up work.
- Security, ownership, transaction integrity, and data-loss issues block release until resolved.
- Scope changes are recorded in this charter and the relevant milestone document before implementation.

## 11. Reference Documents

- [Repository overview](../README.md)
- [Project proposal](Project_Proposal.md)
- [Milestone 2 ER diagram and data model](Milestone2.md)

## 12. Charter Approval

This charter establishes the shared purpose, boundaries, responsibilities, technical direction, and acceptance criteria for Team CHAOS. Team acceptance indicates agreement to prioritize the in-scope database-backed workflows and their verification before optional extensions.

| Role | Name | Approval / date |
| --- | --- | --- |
| Team leader | Can Sei Keçeci |  |
| Team member | Hamza Çevikbaş |  |
| Team member | Ahmet Abalı |  |
| Team member | Ömer Türk |  |
| Team member | Selim Efe Aytemur |  |
