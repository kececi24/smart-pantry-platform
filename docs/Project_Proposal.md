# BBF 302E Database Systems

## Term Project Proposal

*Milestone 1  ·  Fall 2026*

# 1. Team Information

| **Team name** | CHAOS |
| --- | --- |
| **Selected project topic** | Recipe & Smart Pantry Platform |
| **Topic registered in the mapping document?** | Yes |
| **Course section(s)** | All except Ahmet Abalı enrolled in the course section with CRN 12492, Ahmet Abalı is enrolled in the course section with CRN 12495. Although him being from a different section, we believe our team cohesion will benefit positively due to mutual friendship between team members. |

| **#** | **Full name** | **Student ID** | **Section** | **Role in the team** | **E-mail (ITU)** |
| --- | --- | --- | --- | --- | --- |
| 1 | Can Sei Keçeci | 150240007 | 12492 | Team Leader – Backend | [kececi24@itu.edu.tr](mailto:kececi24@itu.edu.tr) |
| 2 | Hamza Çevikbaş | 150240025 | 12492 | Authentication | [cevikbash24@itu.edu.tr](mailto:cevikbash24@itu.edu.tr) |
| 3 | Ahmet Abalı | 150240052 | 12495 | Frontend | [abali24@itu.edu.tr](mailto:abali24@itu.edu.tr) |
| 4 | Ömer Türk | 150240323 | 12492 | Backend - Shopping & Purchase Module | [turkom24@itu.edu.tr](mailto:turkom24@itu.edu.tr) |
| 5 | Selim Efe Aytemur | 150230070 | 12492 | Database design | [aytemur23@itu.edu.tr](mailto:aytemur23@itu.edu.tr) |

# 2. Project Overview

## 2.1 Project description

Recipe & Smart Pantry Platform is a web application that helps users keep track of food at home and turn that inventory into practical meal and shopping decisions. Users record ingredients as individual pantry lots with quantities, storage locations and expiration dates. The platform shows what is available, highlights items approaching their recorded expiration date, and distinguishes those items from food already past that date. Maintaining separate lots prevents a newly purchased package from hiding the expiration date of an older package of the same ingredient.

The platform aims to reduce household food waste and meal-planning effort by connecting what a person has at home with what they can cook and what they still need to buy. It targets individuals who cook for themselves and manage their own groceries.

Users can create and publish recipes, specify ingredient quantities and base servings, and search for recipes that match their available pantry quantities and dietary preference. When users select a recipe and serving count, the platform identifies shortages and can add them to a shopping list. Users can also maintain lists manually and record purchased items back into the pantry. Cooking a recipe or recording direct consumption reduces the appropriate pantry lots and creates a consumption record in one transaction. The initial version supports personal pantries, rule-based recipe matching and a shared recipe catalog; shared households, retailer integrations, barcode recognition, nutrition calculations and machine-learning recommendations are outside the core scope.

## 2.2 Why does this project need a database?

The project requires a relational database because the same ingredient is referenced by pantry lots, recipe ingredient lines, shopping items and consumption details. Users own multiple recipes, pantry lots, shopping lists and consumption events. Junction tables resolve the many-to-many relationships between recipes and ingredients, lists and ingredients, and consumption events and pantry lots. A separate preference record provides a one-to-one relationship with each user. Primary keys, foreign keys, uniqueness constraints and quantity checks prevent orphan records, duplicate recipe lines and invalid stock values.

Database transactions are central to correct behavior. Cooking may deduct several lots and insert several consumption lines, while marking a shopping item as purchased updates a list and adds pantry stock. Each operation must either be completed fully or leave all tables unchanged. Row-level locks prevent concurrent requests from consuming the same remaining quantity twice. SQL joins, aggregation, common table expressions and window functions support recipe eligibility, expiring-stock prioritization, missing-ingredient calculations and consumption summaries. Persistent storage also allows users to return to the same inventory and history across devices and sessions.

# 3. User Roles

| **Role** | **Description** | **What this role can do** |
| --- | --- | --- |
| Guest | Visitor without an account. | Browse published recipes and ingredient summaries, view recipe details, register or log in. No access to pantry, shopping or consumption data. |
| Member | Authenticated owner of a personal pantry. | Manage own preferences, pantry, recipes, lists and consumption, match public or own recipes, publish or unpublish own recipes. |
| Admin | Authenticated administrator with a maintenance role. | Maintain the ingredient catalog, moderate published recipes and manage account status. Private pantry, shopping and consumption data remain owner-only, administrative status does not bypass ownership checks. |

# 4. Functional Requirements

| **ID** | **Requirement** | **User role(s)** | **Related table(s)** | **C/R/U/D** | **Responsible** |
| --- | --- | --- | --- | --- | --- |
| FR-01 | Register an account and create its default preference profile atomically. Reject duplicate normalized e-mail addresses. | Guest | Users, User​Preferences | C, R | Can |
| FR-02 | Log in and log out, update own account details/password. An admin can disable an account or change its access role. | Member, Admin | Users | R, U | Can |
| FR-03 | View and change dietary preference, expiration alert window and preferred recipe sort order. | Member | User​Preferences | R, U | Can |
| FR-04 | Browse and search canonical ingredients. Admins create/edit catalog entries and delete only unreferenced entries. | Guest, Member, Admin | Ingredients | C, R, U, D | Hamza |
| FR-05 | Add, view and edit own pantry lots, including quantity, date and storage location. Delete only empty lots with no consumption references. | Member | Pantry​Items, Ingredients | C, R, U, D | Hamza |
| FR-06 | Show available, expiring and expired inventory. Aggregate usable quantities across compatible lots and filter by ingredient/location. | Member | Pantry​Items, Ingredients, User​Preferences | R | Hamza |
| FR-07 | Create/edit/delete an owned recipe draft and publish/unpublish it. Publishing requires valid instructions and at least one ingredient line. | Member | Recipes, Recipe​Ingredients | C, R, U, D | Ahmet |
| FR-08 | Add, edit or remove ingredient quantities in an owned recipe, scale required amounts to a selected serving count. | Member | Recipes, Recipe​Ingredients, Ingredients | C, R, U, D | Ahmet |
| FR-09 | Search visible recipes by title, preparation time and dietary compatibility, view ingredient requirements and instructions. | Guest, Member | Recipes, Recipe​Ingredients, Ingredients, User​Preferences | R | Ahmet |
| FR-10 | Match recipes to the member's pantry, distinguishing fully cookable recipes from recipes with shortages and ranking expiring-stock use. | Member | Recipes, Recipe​Ingredients, Ingredients, Pantry​Items, User​Preferences | R | Ahmet |
| FR-11 | Create, rename, view, archive/reopen or delete own shopping lists and their item lines. | Member | Shopping​Lists, Shopping​List​Items | C, R, U, D | Ömer |
| FR-12 | Add, edit, remove and mark shopping items as purchased. Keep one ingredient row per list and retain each item's quantity. | Member | Shopping​Lists, Shopping​List​Items, Ingredients | C, R, U, D | Ömer |
| FR-13 | Generate a recipe-shortage list for selected servings. Add missing amounts to a new list or merge into an existing open list. | Member | Recipes, Recipe​Ingredients, Pantry​Items, Shopping​Lists, Shopping​List​Items | C, R, U | Ömer |
| FR-14 | Confirm a purchase by creating a pantry lot and updating the shopping item in one transaction. Repeated confirmation must not duplicate stock. | Member | Shopping​Lists, Shopping​List​Items, Pantry​Items, Ingredients | C, R, U | Ömer |
| FR-15 | Record cooking for selected servings, allocate ingredient deductions to available lots and create a consumption event with its detail lines atomically. | Member | Recipes, Recipe​Ingredients<br><br>Pantry​Items, Consumption​Events, Consumption​Items | C, R, U | Selim |
| FR-16 | Record direct food consumption without a recipe. Specify an ingredient and quantity and deduct it from valid owned pantry lots. | Member | Pantry​Items, Consumption​Events, Consumption​Items | C, R, U | Selim |
| FR-17 | View/filter own consumption history. Correct or delete an event through a transaction that reverses old stock effects before applying changes. | Member | Consumption​Events, Consumption​Items, Pantry​Items | C, R, U, D | Selim |
| FR-18 | Show a monthly dashboard of quantities consumed by ingredient and canonical unit, event counts and most frequently cooked recipes. | Member | Consumption​Events, Consumption​Items, Pantry​Items, Ingredients, Recipes | R | Selim |
| FR-19 | Reset preference settings or delete an owned account after removing dependent records through the guided application workflow. | Member | Users, User​Preferences, all owned records | R, U, D | Can |
| FR-20 | Moderate a published recipe by unpublishing it with a recorded reason. The owner may edit and resubmit it. | Admin, Member | Recipes | R, U | Hamza |

# 5. Non-Functional Requirements

| **ID** | **Category** | **Requirement** | **Responsible** |
| --- | --- | --- | --- |
| NFR-01 | Authentication | Store Argon2id password hashes only. Protected routes return 401 without a valid, unexpired token, disabled accounts are refused. Automated tests cover all protected endpoint groups. | Can |
| NFR-02 | Authorization | Every owner-scoped SELECT/UPDATE/DELETE filters by the authenticated user ID. Cross-account requests return 403 or 404. Admin-only actions return 403 for members, tests include guessed record IDs. | Can |
| NFR-03 | SQL and input safety | Use parameterized raw SQL for every external value, allowlist sort fields. Reject negative stock, nonpositive line quantities, malformed dates and incompatible units before mutation, injection fixtures must leave data unchanged. | Hamza |
| NFR-04 | Inventory performance | On a documented local test environment with 100,000 pantry rows, pantry search and expiration queries target p95 server response time below 500 ms over 100 measured requests after warm-up. Validate indexes with EXPLAIN. | Hamza |
| NFR-05 | Recipe performance | On 200 recipes and a separate 50,000-line recipe fixture, paginated recipe matching targets p95 below 1 second over 100 measured requests. Document hardware, query plans and dataset size. | Ahmet |
| NFR-06 | Usability | At 360 px and desktop widths, inventory, recipe matching and shopping flows remain usable. Five peer testers must each complete add-food, find-recipe and create-list tasks within three minutes per task without guidance. | Ahmet |
| NFR-07 | Matching correctness | Deterministic fixtures verify serving scaling, lot aggregation, expired-lot exclusion, unknown dates, dietary filtering and missing quantities. Each expected result must match the independently specified fixture output. | Ömer |
| NFR-08 | Purchase reliability | A repeated purchase confirmation creates exactly one pantry addition. Forced failure between stock creation and list update rolls back both, two simultaneous confirmations produce one success and one no-op/conflict. | Ömer |
| NFR-09 | Transaction integrity | Concurrent consumption of the last available quantity never creates negative stock. Failed cooking, correction or deletion leaves stock and event rows unchanged, a two-request concurrency test and injected-failure tests verify this. | Selim |
| NFR-10 | Maintainability and recovery | Keep numbered raw-SQL migrations, a fixed-seed generator and setup instructions in Git. A clean installation must build the schema and run checks without an ORM, restore a mysqldump backup and compare row counts and key aggregates. | Selim |

# 6. Planned Technology Stack

| **Relational DBMS** | MySQL with foreign keys, CHECK/UNIQUE constraints, NUMERIC quantities, transactions, row locks and indexes. |
| --- | --- |
| **Database driver (raw SQL)** | Python Flask API with mysqlclient and a small connection pool. All reads and mutations use handwritten, parameterized raw SQL, no ORM or automatic schema/code generator. |
| **Front-end / web UI** | HTML, CSS and JavaScript with Flask templates and fetch-based interaction. Forms cover inventory, recipes, matching, shopping and history, layouts support mobile and desktop. |
| **Authentication method** | Argon2id password hashing, signed JWT access token in an HttpOnly cookie with 30-minute expiration, Secure in deployment and SameSite=Lax. CSRF tokens protect mutations. Validate Users.is_active and token_version on protected requests, logout clears the cookie and password changes invalidate existing tokens. |
| **Caching layer (optional)** | Not planned for the core version. Pantry balances and transactional operations read MySQL directly, use indexes, pagination and query optimization first. |
| **Data source** | Fixed-seed Python generator plus original ingredient labels and fictional recipe records. No external dataset or real personal records. The same application services validate generated and user-entered data. |

# 7. Work Distribution Summary

| **Member** | **Tables (2 per member)** | **Functional req. IDs** | **Non-functional req. IDs** |
| --- | --- | --- | --- |
| Can: Account and preference module | Users, User​Preferences | FR-01–03, FR-19 | NFR-01–02 |
| Hamza: Ingredient and pantry module | Ingredients, Pantry​Items | FR-04–06, FR-20 | NFR-03–04 |
| Ahmet: Recipe and matching module | Recipes, Recipe​Ingredients | FR-07–10 | NFR-05-06 |
| Ömer: Shopping and purchase module | Shopping​Lists, Shopping​List​Items | FR-11–14 | NFR-07–08 |
| Selim: Consumption and reporting module | Consumption​Events, Consumption​Items | FR-15–18 | NFR-09–10 |

# 8. Pre-Submission Checklist

☒  The team has 5 members, and the team leader is indicated.

☒  The project topic is selected from the provided list and our team name is written in the topic–team mapping document.

☒  The proposed functionality is sufficiently complex to support at least 10 related database tables; detailed design is deferred to Milestone 2.

☒  Every functional and non-functional requirement has a responsible member.

☒  The primary database is a relational DBMS other than SQLite.

☒  No ORM or auto code generator will be used; all queries will be written in raw SQL.

☒  Authentication (login/register and protected routes) and user roles are planned.

☒  All gray guidance text and example rows have been removed.

# For Instructor Use Only

| **Decision** | ☐ Approved     ☐ Approved with minor changes     ☐ Revision required |
| --- | --- |
| **Comments** |  |
| **Reviewed by / date** |  |
