# Recipe and Smart Pantry Platform — User Stories

This backlog is derived from `Requirements_Specification.md` and `MVC_Architectural_Design.md`. Priorities use MoSCoW, and estimates are relative story points using the Fibonacci scale.

## Accounts and Preferences

### US-01 — Register an Account

**Title:** Register an account with default preferences  
**Priority:** Must  
**As a Guest, I want** to register with my name, email, and password **so that** I can use private pantry and planning features.  
**Notes:** Covers FR-01. Normalize the email and create the user and default preference record atomically.  
**Assumptions:** The email is not already registered after normalization.  
**Constraints:** Store only an Argon2id password hash; a failure must leave neither record committed.  
**Estimate:** 5 story points

### US-02 — Sign In

**Title:** Sign in securely  
**Priority:** Must  
**As a registered user, I want** to sign in with valid credentials **so that** I can access features permitted by my role.  
**Notes:** Covers FR-02 and NFR-01.  
**Assumptions:** The account is active.  
**Constraints:** Use a signed, secure, HTTP-only session cookie with expiry and token-version checks; reject invalid or disabled accounts.  
**Estimate:** 5 story points

### US-03 — Sign Out

**Title:** Sign out safely  
**Priority:** Must  
**As an authenticated user, I want** to sign out **so that** my session can no longer be used from the current browser.  
**Notes:** Covers FR-02. Logout is a state-changing action.  
**Assumptions:** The user has an active or previously issued session cookie.  
**Constraints:** Use POST with CSRF protection and clear the authentication cookie.  
**Estimate:** 2 story points

### US-04 — Maintain Account Details

**Title:** Update personal account details  
**Priority:** Must  
**As a Member, I want** to update my own account details **so that** my profile remains accurate.  
**Notes:** Covers FR-02.  
**Assumptions:** Email changes follow the same normalization and uniqueness rules as registration.  
**Constraints:** Every read and update must be scoped to the authenticated user.  
**Estimate:** 3 story points

### US-05 — Change Password

**Title:** Change account password  
**Priority:** Must  
**As a Member, I want** to change my password after identity verification **so that** I can keep my account secure.  
**Notes:** Covers FR-02 and NFR-01.  
**Assumptions:** The Member supplies the current password and a valid replacement.  
**Constraints:** Hash with Argon2id and invalidate previously issued tokens by changing the token version.  
**Estimate:** 3 story points

### US-06 — Manage Preferences

**Title:** Manage recipe and expiry preferences  
**Priority:** Must  
**As a Member, I want** to set my expiry-alert window, preferred recipe sort order, and dietary tags **so that** results reflect my needs.  
**Notes:** Covers FR-03. Scalar settings and tag selections should be saved together.  
**Assumptions:** Dietary tags are selected from the canonical catalog.  
**Constraints:** Settings are private and owner-scoped; sort choices must come from an allowlist.  
**Estimate:** 5 story points

### US-07 — Reset Preferences

**Title:** Reset preferences to defaults  
**Priority:** Should  
**As a Member, I want** to reset my preferences **so that** I can quickly return to the documented default behavior.  
**Notes:** Covers FR-03 and FR-19.  
**Assumptions:** Default values are centrally defined and documented.  
**Constraints:** Reset only the authenticated Member's preferences and dietary-tag selections.  
**Estimate:** 2 story points

### US-08 — Delete Account

**Title:** Delete my account through a guided workflow  
**Priority:** Must  
**As a Member, I want** to delete my account after verifying my identity **so that** I can remove my account and dependent data.  
**Notes:** Covers FR-19. Explain the effect before final confirmation.  
**Assumptions:** The documented retention and deletion rules identify what is deleted or cleared.  
**Constraints:** Delete dependencies in a valid order within one transaction; interruption must leave the account unchanged.  
**Estimate:** 8 story points

## Ingredient Catalog and Pantry

### US-09 — Browse Ingredients and Units

**Title:** Browse and search the ingredient catalog  
**Priority:** Must  
**As a visitor, I want** to search canonical ingredients and compatible units **so that** I can understand the data used by recipes and pantry lots.  
**Notes:** Covers FR-04. Public read access is allowed for Guest, Member, and Admin roles.  
**Assumptions:** Catalog entries have canonical names, categories, measurement types, and default units.  
**Constraints:** Search values must be parameterized and results paginated when necessary.  
**Estimate:** 3 story points

### US-10 — Maintain Ingredient Catalog

**Title:** Maintain canonical ingredients and units  
**Priority:** Must  
**As an Admin, I want** to create and edit catalog entries and delete unreferenced entries **so that** application quantities remain consistent.  
**Notes:** Covers FR-04.  
**Assumptions:** Canonical-name and unit-conversion rules are defined.  
**Constraints:** Members receive `403`; referenced ingredients or units cannot be deleted; mutations require CSRF protection.  
**Estimate:** 8 story points

### US-11 — Add a Pantry Lot

**Title:** Add a separate pantry lot  
**Priority:** Must  
**As a Member, I want** to record an ingredient lot with quantity, unit, location, and optional expiry date **so that** my physical stock is represented accurately.  
**Notes:** Covers FR-05. Separate purchases may remain separate lots.  
**Assumptions:** The ingredient and selected compatible unit already exist.  
**Constraints:** Quantities must be valid decimals and nonnegative; remaining quantity cannot exceed the initial quantity.  
**Estimate:** 5 story points

### US-12 — View and Filter Pantry

**Title:** View and filter my pantry inventory  
**Priority:** Must  
**As a Member, I want** to filter my pantry by ingredient, location, and stock status **so that** I can find the food I have.  
**Notes:** Covers FR-05 and FR-06. Show available, expiring, expired, and depleted states.  
**Assumptions:** The expiry-alert window comes from the Member's preferences.  
**Constraints:** Return only the authenticated owner's lots; the UI must work at 360 px and desktop widths.  
**Estimate:** 5 story points

### US-13 — View Compatible Pantry Totals

**Title:** View usable inventory totals  
**Priority:** Must  
**As a Member, I want** compatible pantry quantities aggregated in canonical units **so that** I know how much usable stock I have.  
**Notes:** Covers FR-06. Also show the earliest known expiration where applicable.  
**Assumptions:** Unit conversion multipliers and measurement dimensions are correct.  
**Constraints:** Do not combine incompatible dimensions; expired and depleted lots contribute no usable quantity; unknown dates create no expiry alert.  
**Estimate:** 8 story points

### US-14 — Edit a Pantry Lot

**Title:** Correct an owned pantry lot  
**Priority:** Must  
**As a Member, I want** to edit a pantry lot I own **so that** quantities, dates, and storage details remain accurate.  
**Notes:** Covers FR-05.  
**Assumptions:** Protected historical relationships remain valid after permitted edits.  
**Constraints:** Enforce ownership, compatible units, valid dates, and quantity invariants before mutation.  
**Estimate:** 3 story points

### US-15 — Delete an Empty Pantry Lot

**Title:** Delete an unused empty pantry lot  
**Priority:** Should  
**As a Member, I want** to delete an empty unreferenced lot **so that** obsolete inventory records do not clutter my pantry.  
**Notes:** Covers FR-05.  
**Assumptions:** The lot has no remaining stock.  
**Constraints:** The lot must be owned, depleted, and have no consumption or confirmed-purchase reference.  
**Estimate:** 3 story points

## Recipes and Matching

### US-16 — Create a Recipe Draft

**Title:** Create a private recipe draft  
**Priority:** Must  
**As a Member, I want** to save a recipe draft with instructions, servings, preparation time, tags, and ingredients **so that** I can develop a recipe before sharing it.  
**Notes:** Covers FR-07 and FR-08.  
**Assumptions:** Ingredients, units, and dietary tags already exist.  
**Constraints:** Ingredient lines require positive decimal quantities, compatible units, and one line per ingredient.  
**Estimate:** 8 story points

### US-17 — Edit or Delete a Recipe Draft

**Title:** Maintain my recipe draft  
**Priority:** Must  
**As a recipe owner, I want** to edit or delete my draft **so that** I control my unpublished recipe content.  
**Notes:** Covers FR-07 and FR-08.  
**Assumptions:** Deletion follows the documented reference-preservation rules.  
**Constraints:** Only the owner may modify the draft; another user's guessed identifier must reveal no private content.  
**Estimate:** 5 story points

### US-18 — Publish or Unpublish a Recipe

**Title:** Publish or unpublish an owned recipe  
**Priority:** Must  
**As a recipe owner, I want** to publish a complete recipe or return it to draft **so that** I decide when other users can discover it.  
**Notes:** Covers FR-07.  
**Assumptions:** Publication makes the recipe visible through public search.  
**Constraints:** Publication requires nonblank instructions and at least one valid ingredient line; mutation uses POST and CSRF protection.  
**Estimate:** 3 story points

### US-19 — Search Visible Recipes

**Title:** Search recipes with useful filters  
**Priority:** Must  
**As a visitor, I want** to search visible recipes by title, preparation time, dietary compatibility, and sort order **so that** I can find a suitable meal.  
**Notes:** Covers FR-09. Owners may also see their own drafts.  
**Assumptions:** Dietary tags accurately describe recipes.  
**Constraints:** Apply visibility before filtering; paginate results; allowlist all sort columns and directions.  
**Estimate:** 8 story points

### US-20 — View and Scale a Recipe

**Title:** View recipe instructions for selected servings  
**Priority:** Must  
**As a visitor, I want** to choose a positive serving count and see scaled ingredient quantities **so that** I can prepare the correct amount.  
**Notes:** Covers FR-08 and FR-09. Scaling changes display values, not stored base quantities.  
**Assumptions:** The recipe is visible to the current actor.  
**Constraints:** Reject nonpositive servings and preserve decimal precision and compatible units.  
**Estimate:** 5 story points

### US-21 — Match Recipes Against Pantry

**Title:** Find recipes that match my pantry  
**Priority:** Must  
**As a Member, I want** visible recipes compared with my usable pantry stock **so that** I can identify meals I can cook now.  
**Notes:** Covers FR-10 and NFR-07. Show fully cookable recipes before partial matches.  
**Assumptions:** Current pantry, preferences, and selected servings are available.  
**Constraints:** Exclude expired and depleted lots, respect dietary filters and ownership, aggregate compatible units, and target p95 below one second.  
**Estimate:** 13 story points

### US-22 — Review Recipe Shortages

**Title:** Review missing recipe ingredients  
**Priority:** Must  
**As a Member, I want** to see available and missing quantities for each recipe line **so that** I know exactly what I need to buy.  
**Notes:** Covers FR-10 and FR-13. Rank with expiring-stock use as a secondary signal.  
**Assumptions:** A recipe match has been calculated for a positive serving count.  
**Constraints:** Shortages must be deterministic and never negative; pantry data must be current and owner-scoped.  
**Estimate:** 5 story points

## Shopping

### US-23 — Manage Shopping Lists

**Title:** Create and organize shopping lists  
**Priority:** Must  
**As a Member, I want** to create, rename, view, archive, reopen, and delete my shopping lists **so that** planned purchases stay organized.  
**Notes:** Covers FR-11.  
**Assumptions:** List status transitions are documented.  
**Constraints:** Operations are owner-scoped; deleting a list must not delete pantry lots created by confirmed purchases.  
**Estimate:** 8 story points

### US-24 — Manage Shopping Items

**Title:** Maintain shopping-list items  
**Priority:** Must  
**As a Member, I want** to add, edit, remove, and mark items purchased **so that** each list reflects what I still need.  
**Notes:** Covers FR-12.  
**Assumptions:** Items reference canonical ingredients and compatible units.  
**Constraints:** Quantities must be positive; keep at most one compatible unpurchased row per ingredient in a list; enforce ownership.  
**Estimate:** 8 story points

### US-25 — Add Shortages to Shopping

**Title:** Add recipe shortages to a shopping list  
**Priority:** Must  
**As a Member, I want** to add calculated shortages to a new or open shopping list **so that** I do not re-enter missing ingredients manually.  
**Notes:** Covers FR-13.  
**Assumptions:** Shortages come from a current visible recipe and selected serving count.  
**Constraints:** Merge positive compatible quantities under a lock; purchased or incompatible targets must follow the documented conflict behavior.  
**Estimate:** 8 story points

### US-26 — Confirm a Purchase

**Title:** Confirm a purchased item into the pantry  
**Priority:** Must  
**As a Member, I want** purchase confirmation to create and link one pantry lot **so that** delivered food appears in inventory exactly once.  
**Notes:** Covers FR-14 and NFR-08. A retry returns the existing linked lot.  
**Assumptions:** The Member supplies valid pantry-lot details for the purchased item.  
**Constraints:** Lock the owned item and create the lot, flag, and link in one transaction; failures roll back all changes; concurrent requests cannot duplicate stock.  
**Estimate:** 13 story points

## Consumption and Reporting

### US-27 — Record Recipe Cooking

**Title:** Record cooking from pantry stock  
**Priority:** Must  
**As a Member, I want** cooking a recipe to deduct the correct owned pantry lots **so that** my inventory and cooking history remain accurate.  
**Notes:** Covers FR-15 and NFR-09. Store one event and per-lot deduction lines.  
**Assumptions:** The recipe is visible, valid, and requested for a positive serving count.  
**Constraints:** Lock eligible lots, allocate FIFO, and commit deductions and event records atomically; never allow negative stock.  
**Estimate:** 13 story points

### US-28 — Record Direct Consumption

**Title:** Record direct ingredient consumption  
**Priority:** Must  
**As a Member, I want** to record use of one ingredient without a recipe **so that** pantry stock stays correct for snacks, waste, or other direct use.  
**Notes:** Covers FR-16 and NFR-09.  
**Assumptions:** The ingredient quantity and unit are valid and compatible.  
**Constraints:** The event has no recipe or servings; use FIFO allocation and one atomic transaction; insufficient stock changes nothing.  
**Estimate:** 8 story points

### US-29 — View Consumption History

**Title:** Filter my consumption history  
**Priority:** Must  
**As a Member, I want** to filter my consumption history by date, type, and recipe **so that** I can review how pantry stock was used.  
**Notes:** Covers FR-17. Include event details and per-lot deductions where useful.  
**Assumptions:** The Member has zero or more consumption events.  
**Constraints:** Return only owned events; paginate results and use privacy-preserving errors for inaccessible identifiers.  
**Estimate:** 5 story points

### US-30 — Correct a Consumption Event

**Title:** Correct a recorded consumption event  
**Priority:** Must  
**As a Member, I want** to correct an event I own **so that** both history and pantry balances reflect what actually happened.  
**Notes:** Covers FR-17 and NFR-09.  
**Assumptions:** The original event and affected pantry lots still exist.  
**Constraints:** Lock the event and lots, restore original deductions, apply replacements, and commit atomically; failure preserves the original state.  
**Estimate:** 13 story points

### US-31 — Delete a Consumption Event

**Title:** Delete an incorrect consumption event  
**Priority:** Must  
**As a Member, I want** to delete an owned event and restore its deductions **so that** an invalid record does not distort inventory or reports.  
**Notes:** Covers FR-17 and NFR-09.  
**Assumptions:** Restoring the recorded deductions does not violate lot invariants.  
**Constraints:** Restoration and deletion occur in one transaction with deterministic lock order; cross-owner access is rejected.  
**Estimate:** 8 story points

### US-32 — View Monthly Consumption Report

**Title:** Review monthly consumption summaries  
**Priority:** Should  
**As a Member, I want** monthly ingredient totals, event counts, and most-cooked recipes **so that** I can understand my consumption patterns.  
**Notes:** Covers FR-18. Provide a clear empty state for months with no activity.  
**Assumptions:** Canonical units support meaningful aggregation.  
**Constraints:** Use a half-open month range; do not multiply cooking-event counts by joined detail rows; show only owned data.  
**Estimate:** 8 story points

## Administration

### US-33 — Manage Account Access

**Title:** Disable accounts and manage roles  
**Priority:** Must  
**As an Admin, I want** to disable or re-enable accounts and apply allowed role changes **so that** access can be administered safely.  
**Notes:** Covers FR-02.  
**Assumptions:** Allowed transitions and safeguards for the final Admin are documented.  
**Constraints:** Admin authorization and CSRF are required; access changes invalidate earlier tokens; Admin status does not grant access to private pantry data.  
**Estimate:** 5 story points

### US-34 — Moderate a Published Recipe

**Title:** Unpublish a recipe with a moderation reason  
**Priority:** Must  
**As an Admin, I want** to return an unsuitable published recipe to draft with a reason **so that** public content can be corrected transparently.  
**Notes:** Covers FR-20.  
**Assumptions:** Only published recipes are eligible for this moderation action.  
**Constraints:** Save a nonblank reason, require Admin role and CSRF protection, and do not expose unrelated private records.  
**Estimate:** 5 story points

### US-35 — Review and Resubmit a Moderated Recipe

**Title:** Correct and resubmit a moderated recipe  
**Priority:** Must  
**As a recipe owner, I want** to see the moderation reason, edit my recipe, and resubmit it **so that** I can address the problem and publish a valid version.  
**Notes:** Covers FR-20.  
**Assumptions:** The recipe remains owned by its original author in draft state.  
**Constraints:** Only the owner sees the saved reason through the normal owner-scoped recipe view; publication validation still applies.  
**Estimate:** 3 story points

## Quality, Operations, and Release

### US-36 — Use an Accessible Responsive Interface

**Title:** Complete core workflows on mobile and desktop  
**Priority:** Must  
**As a user, I want** accessible pantry, recipe, shopping, and consumption screens at 360 px and desktop widths **so that** I can complete tasks on my preferred device.  
**Notes:** Covers NFR-06 and the MVC view contract. Use semantic HTML, visible labels, text-supported status indicators, and reusable Jinja components.  
**Assumptions:** Bulma loads before project CSS.  
**Constraints:** Five peer testers should complete add-food, find-recipe, and create-list tasks within three minutes each without guidance.  
**Estimate:** 8 story points

### US-37 — Receive Safe and Useful Feedback

**Title:** Receive consistent validation and error feedback  
**Priority:** Must  
**As a user, I want** clear validation, success, conflict, and failure messages **so that** I understand the outcome and can recover safely.  
**Notes:** Covers the MVC error contract. Use Post/Redirect/Get after successful form mutations.  
**Assumptions:** Services raise typed application errors.  
**Constraints:** Map failures consistently to `400`, `401`, `403`, `404`, `409`, or `500`; never expose SQL, secrets, or private data.  
**Estimate:** 5 story points

### US-38 — Protect Private Data and Mutations

**Title:** Enforce authentication, ownership, roles, and request safety  
**Priority:** Must  
**As a Member, I want** every private action protected **so that** another user cannot read or change my data.  
**Notes:** Covers NFR-01 through NFR-03. Include guessed-ID, cross-account, wrong-role, injection, and invalid-input tests.  
**Assumptions:** Authentication middleware exposes a verified current user.  
**Constraints:** Owner-filter every private SQL operation, parameterize external values, allowlist dynamic SQL, protect all mutations with CSRF, and escape rendered output.  
**Estimate:** 13 story points

### US-39 — Reproduce the Database and Test Data

**Title:** Build the schema and fixtures reproducibly  
**Priority:** Must  
**As a maintainer, I want** numbered raw-SQL migrations and fixed-seed generators **so that** any team member can recreate the schema and test datasets.  
**Notes:** Covers NFR-10. Include functional and benchmark fixtures.  
**Assumptions:** A supported MySQL/InnoDB environment is available.  
**Constraints:** Use handwritten parameterized SQL through `mysqlclient`; no ORM or automatic schema generator.  
**Estimate:** 8 story points

### US-40 — Verify Performance Targets

**Title:** Measure pantry and recipe query performance  
**Priority:** Should  
**As a course reviewer, I want** reproducible latency measurements and query plans **so that** performance claims can be evaluated objectively.  
**Notes:** Covers NFR-04 and NFR-05. Record hardware, fixture sizes, warm-up, samples, pagination, and `EXPLAIN` output.  
**Assumptions:** Benchmark fixtures contain 100,000 pantry rows, 200 recipes, and a separate 50,000-line recipe dataset.  
**Constraints:** Measure 100 requests; target pantry p95 below 500 ms and paginated recipe matching p95 below one second.  
**Estimate:** 8 story points

### US-41 — Verify Transaction and Concurrency Integrity

**Title:** Prove transaction and concurrency behavior  
**Priority:** Must  
**As a course reviewer, I want** injected-failure, retry, and simultaneous-request evidence **so that** purchases and consumption cannot corrupt inventory.  
**Notes:** Covers NFR-08 and NFR-09.  
**Assumptions:** Tests run against MySQL/InnoDB rather than a transactionally different substitute.  
**Constraints:** Verify exact-once purchase confirmation, rollback at every multi-step boundary, deterministic locks, and no negative stock.  
**Estimate:** 13 story points

### US-42 — Back Up and Restore the Database

**Title:** Rehearse database backup and recovery  
**Priority:** Must  
**As a maintainer, I want** a documented and tested backup/restore procedure **so that** project data can be recovered reliably.  
**Notes:** Covers NFR-10. Restore into an isolated database and compare row counts and key aggregates.  
**Assumptions:** `mysqldump` and compatible MySQL client tools are available.  
**Constraints:** The procedure must be repeatable from a clean checkout and must not expose secrets or overwrite the source database.  
**Estimate:** 5 story points

### US-43 — Reproduce and Assess the Release

**Title:** Reproduce setup, verification, and demonstration  
**Priority:** Must  
**As a course reviewer, I want** complete setup, test, limitation, and demonstration instructions **so that** I can assess the selected revision consistently.  
**Notes:** Covers UC-09 and NFR-10. Include the tested revision, known limitations, stable seed data, and fixed demo script.  
**Assumptions:** Example configuration contains no production secrets.  
**Constraints:** A clean checkout must support migration, seeding, testing, benchmarking, backup, restore, and the major end-to-end workflows without manual database repair.  
**Estimate:** 8 story points
