# Recipe and Smart Pantry Platform — Gherkin Acceptance Criteria

This document translates the 43 stories in `User_Stories.md` into behavior-driven acceptance criteria. Story IDs are preserved for traceability. The scenarios assume MySQL/InnoDB, owner-scoped access, parameterized SQL, CSRF protection for mutations, and the server-rendered Flask MVC design described in the project documents.

## Accounts and Preferences

```gherkin
Feature: US-01 Register an account with default preferences
  As a Guest
  I want to register an account
  So that I can use private pantry and planning features

  Scenario: Register with valid unique details
    Given no account exists for normalized email "member@example.com"
    When the Guest registers as "Member One" with email " Member@Example.com " and a valid password
    Then exactly one user is stored with normalized email "member@example.com"
    And the password is stored only as an Argon2id hash
    And exactly one default preference record is stored for the user
    And the Guest is redirected to the documented post-registration page

  Scenario: Reject a duplicate normalized email
    Given an account exists for normalized email "member@example.com"
    When the Guest registers with email "MEMBER@example.com"
    Then registration is rejected with an email validation message
    And no additional user or preference record is stored

  Scenario: Roll back an interrupted registration
    Given the user insert can succeed
    And preference creation is configured to fail
    When the Guest submits a valid registration
    Then the registration transaction is rolled back
    And neither the user nor the preference record is committed

Feature: US-02 Sign in securely
  Scenario: Sign in with valid credentials
    Given an active account exists with a valid Argon2id password hash
    When the user submits the matching email and password
    Then a signed secure HTTP-only authentication cookie is issued
    And the user is redirected to the member dashboard

  Scenario Outline: Refuse an invalid sign-in
    Given an account is in the "<account_state>" state
    When the user submits "<credential_state>" credentials
    Then no authenticated session is created
    And a safe authentication error is shown

    Examples:
      | account_state | credential_state |
      | active        | incorrect        |
      | disabled      | correct          |
      | missing       | arbitrary        |

  Scenario Outline: Reject an unusable protected-session token
    Given the request contains a token that is "<token_state>"
    When the user requests a protected page
    Then the request is rejected as unauthenticated
    And no private data is returned

    Examples:
      | token_state       |
      | expired           |
      | badly signed      |
      | wrong version     |
      | owned by disabled account |

Feature: US-03 Sign out safely
  Scenario: Sign out through a protected mutation
    Given the user is authenticated
    And the request contains a valid CSRF token
    When the user submits the sign-out form
    Then the authentication cookie is cleared
    And the user is redirected to a public page

  Scenario: Reject sign-out through GET
    Given the user is authenticated
    When the user sends a GET request to the sign-out endpoint
    Then the request is rejected
    And the existing session is not mutated

Feature: US-04 Update personal account details
  Scenario: Update owned account details
    Given a Member is authenticated
    When the Member submits valid changed profile details
    Then only that Member's account is updated
    And the changed details appear on the next request

  Scenario: Reject an email already used by another account
    Given another account owns the normalized replacement email
    When the Member submits that email as an account update
    Then the update is rejected with a field error
    And the original account details remain unchanged

Feature: US-05 Change account password
  Scenario: Change a password after identity verification
    Given a Member is authenticated
    When the Member supplies the correct current password and a valid new password
    Then the new password is stored only as an Argon2id hash
    And the account token version is incremented
    And all previously issued tokens are rejected

  Scenario: Reject a password change with an incorrect current password
    Given a Member is authenticated
    When the Member supplies an incorrect current password
    Then the password and token version remain unchanged
    And a safe verification error is shown

Feature: US-06 Manage recipe and expiry preferences
  Scenario: Save valid preferences and dietary tags
    Given a Member is authenticated
    And the selected dietary tags exist
    When the Member saves a valid expiry-alert window, allowlisted sort order, and tag selection
    Then the scalar settings and tag selection are stored for that Member
    And they reappear on a new request

  Scenario: Prevent access to another Member's preferences
    Given two Members exist
    When the first Member guesses the second Member's preference identifier
    Then the request returns a privacy-preserving denial
    And no preference data is revealed or changed

Feature: US-07 Reset preferences to defaults
  Scenario: Reset owned preferences
    Given a Member has nondefault settings and dietary tags
    When the Member confirms preference reset with a valid CSRF token
    Then the documented scalar defaults are restored
    And the dietary-tag selection is reset as documented
    And no other Member's preferences are changed

Feature: US-08 Delete an account through a guided workflow
  Scenario: Delete an account after identity verification
    Given a Member is authenticated
    And the Member has reviewed the documented deletion effects
    When the Member verifies identity and confirms deletion
    Then owned dependent records are removed or cleared in the documented order
    And the account is deleted
    And the authentication cookie is cleared

  Scenario: Roll back interrupted account deletion
    Given a Member has dependent owned records
    And a failure is injected during dependency removal
    When the Member confirms account deletion
    Then the entire deletion transaction is rolled back
    And the account and all dependencies remain in their original committed state
```

## Ingredient Catalog and Pantry

```gherkin
Feature: US-09 Browse and search the ingredient catalog
  Scenario: Search the public catalog
    Given canonical ingredients and compatible units exist
    When a visitor searches by a partial ingredient name
    Then matching catalog entries are returned with category and default unit
    And authentication is not required

  Scenario: Return a safe empty result
    Given no catalog entry matches the search value
    When a visitor searches the catalog
    Then an empty state is displayed
    And no database or server detail is exposed

Feature: US-10 Maintain canonical ingredients and units
  Scenario: Admin creates a valid catalog entry
    Given an Admin is authenticated
    And the mutation contains a valid CSRF token
    When the Admin submits a unique canonical ingredient with a compatible default unit
    Then the catalog entry is stored
    And it appears in public catalog search

  Scenario: Member cannot mutate the catalog
    Given a Member is authenticated
    When the Member submits a catalog maintenance request
    Then the response status is 403
    And the catalog remains unchanged

  Scenario: Referenced catalog entry cannot be deleted
    Given an ingredient is referenced by a pantry lot or recipe line
    When an Admin requests deletion of that ingredient
    Then deletion is rejected with a preservation message
    And all references remain valid

Feature: US-11 Add a separate pantry lot
  Scenario: Add a valid pantry lot
    Given a Member is authenticated
    And a canonical ingredient and compatible unit exist
    When the Member enters valid initial and remaining quantities, location, and optional expiry date
    Then one separate pantry lot is stored for that Member
    And all submitted lot details appear in the pantry

  Scenario Outline: Reject an invalid pantry lot
    Given a Member is authenticated
    When the Member submits a lot with "<invalid_condition>"
    Then the lot is not stored
    And the affected field displays a validation error

    Examples:
      | invalid_condition                         |
      | a nonpositive initial quantity            |
      | a negative remaining quantity             |
      | remaining quantity above initial quantity |
      | an incompatible measurement unit          |
      | a malformed expiry date                   |

Feature: US-12 View and filter pantry inventory
  Scenario: View only owned pantry lots
    Given two Members have pantry lots
    When the first Member opens the pantry
    Then only the first Member's lots are displayed
    And each lot is labeled available, expiring, expired, or depleted

  Scenario Outline: Filter the pantry
    Given a Member has pantry lots with different ingredients, locations, and states
    When the Member filters by "<filter_type>"
    Then only owned lots matching "<filter_value>" are displayed

    Examples:
      | filter_type | filter_value |
      | ingredient  | Milk         |
      | location    | Freezer      |
      | status      | Expiring     |

  Scenario: Use pantry inventory on a narrow screen
    Given the viewport width is 360 pixels
    When a Member opens and filters the pantry
    Then all primary information and actions remain usable without hidden required controls

Feature: US-13 View usable inventory totals
  Scenario: Aggregate compatible usable quantities
    Given a Member owns available lots of the same ingredient in compatible units
    And the Member also owns expired and depleted lots of that ingredient
    When inventory totals are calculated
    Then available compatible quantities are converted and summed in the canonical unit
    And expired and depleted quantities are excluded
    And the earliest known usable expiry is displayed

  Scenario: Do not combine incompatible dimensions or unknown expiries
    Given pantry lots use incompatible measurement dimensions
    And one usable lot has no expiry date
    When inventory totals and alerts are calculated
    Then incompatible quantities are not combined
    And the unknown date contributes usable stock
    But the unknown date does not generate an expiry alert

Feature: US-14 Correct an owned pantry lot
  Scenario: Edit a valid owned lot
    Given a Member owns a pantry lot
    When the Member submits valid changed quantities, dates, or location
    Then that lot is updated
    And the new values appear on the next pantry request

  Scenario: Prevent cross-owner pantry editing
    Given another Member owns a pantry lot
    When the authenticated Member submits an edit using that lot identifier
    Then the request returns 403 or a privacy-preserving 404
    And the lot remains unchanged

Feature: US-15 Delete an unused empty pantry lot
  Scenario: Delete an empty unreferenced lot
    Given a Member owns a depleted pantry lot
    And the lot has no consumption or confirmed-purchase reference
    When the Member confirms deletion
    Then the lot is removed

  Scenario Outline: Preserve a protected pantry lot
    Given a Member owns a pantry lot that is "<protected_state>"
    When the Member requests deletion
    Then deletion is rejected with an explanatory message
    And the lot remains stored

    Examples:
      | protected_state                    |
      | not depleted                       |
      | referenced by consumption          |
      | linked to a confirmed purchase     |
```

## Recipes and Matching

```gherkin
Feature: US-16 Create a private recipe draft
  Scenario: Save a valid recipe draft
    Given a Member is authenticated
    And referenced ingredients, units, and dietary tags exist
    When the Member saves valid recipe metadata, instructions, and ingredient lines
    Then one draft recipe is stored for that Member
    And its lines and tags are stored
    And the draft is not visible to other visitors

  Scenario Outline: Reject an invalid recipe line
    Given a Member is editing an owned recipe draft
    When the Member adds a line with "<invalid_condition>"
    Then that line is rejected with a field error

    Examples:
      | invalid_condition                  |
      | a nonpositive quantity             |
      | an incompatible unit               |
      | an ingredient already in the recipe |

Feature: US-17 Maintain an owned recipe draft
  Scenario: Edit an owned draft
    Given a Member owns a recipe draft
    When the Member submits valid changed recipe data
    Then the metadata, lines, and tags are updated as one valid aggregate

  Scenario: Reject access to another owner's draft
    Given another Member owns a private recipe draft
    When the authenticated Member requests or mutates it by guessed identifier
    Then the response does not reveal the recipe content
    And the draft remains unchanged

  Scenario: Delete an eligible owned draft
    Given a Member owns a draft eligible for deletion
    When the Member confirms deletion
    Then the draft and its removable dependent lines and tags are deleted

Feature: US-18 Publish or unpublish an owned recipe
  Scenario: Publish a complete recipe
    Given a Member owns a draft with nonblank instructions and at least one valid ingredient line
    When the Member requests publication with a valid CSRF token
    Then the recipe status becomes published
    And the recipe becomes visible in public search

  Scenario Outline: Keep an incomplete recipe as draft
    Given a Member owns a draft with "<missing_requirement>"
    When the Member requests publication
    Then publication is rejected
    And the recipe remains a draft
    And the missing requirement is identified

    Examples:
      | missing_requirement         |
      | blank instructions          |
      | no valid ingredient lines   |

  Scenario: Unpublish an owned recipe
    Given a Member owns a published recipe
    When the Member requests unpublication
    Then the recipe returns to draft
    And it no longer appears in public results

Feature: US-19 Search recipes with useful filters
  Scenario: Guest searches published recipes
    Given published recipes and private drafts exist
    When a Guest searches by title, maximum preparation time, dietary tags, and an allowlisted order
    Then only matching published recipes are returned
    And the results are paginated in the selected allowed order

  Scenario: Owner can find an owned draft
    Given a Member owns a draft and other Members own drafts
    When the Member searches visible recipes
    Then the Member can see the owned draft
    But cannot see drafts owned by other Members

  Scenario: Reject an unapproved sort expression
    Given a visitor supplies a sort value outside the allowlist
    When recipe search is requested
    Then the value is rejected or replaced by the documented default
    And it is never inserted directly into SQL

Feature: US-20 View recipe instructions for selected servings
  Scenario: Scale requirements for positive servings
    Given a visible recipe has base servings and ingredient requirements
    When the visitor selects a positive serving count
    Then each displayed requirement is multiplied by the serving ratio
    And stored base quantities remain unchanged

  Scenario: Reject a nonpositive serving count
    Given a visible recipe exists
    When the visitor selects zero or negative servings
    Then scaling is rejected with a validation message
    And no recipe data is mutated

Feature: US-21 Find recipes that match my pantry
  Scenario: Distinguish full and partial pantry matches
    Given a Member owns usable, expired, depleted, and unknown-expiry pantry lots
    And visible recipes have compatible requirements
    When the Member calculates matches for positive servings
    Then only the Member's usable compatible quantities contribute
    And expired and depleted lots are excluded
    And fully cookable recipes appear before partial matches
    And each result has deterministic available and missing quantities

  Scenario: Apply dietary compatibility to matching
    Given a Member has selected dietary tags
    When recipe matches are calculated
    Then incompatible visible recipes are excluded
    And compatible visible recipes are evaluated against pantry stock

  Scenario: Prevent another pantry from contributing stock
    Given another Member has stock required by a recipe
    And the authenticated Member does not
    When the authenticated Member calculates matches
    Then the other Member's stock is ignored
    And the requirement is reported as missing

Feature: US-22 Review missing recipe ingredients
  Scenario: Display shortages for each recipe line
    Given a Member has calculated a recipe match for selected servings
    When the Member opens the match detail
    Then every line shows the required, available, and nonnegative missing amount
    And zero shortage is shown for satisfied lines

  Scenario: Rank matches by expiring-stock use
    Given two recipes have the same primary cookability status
    And one would use more known expiring stock
    When matches are ordered
    Then the recipe using more expiring stock is preferred by the documented secondary ranking
```

## Shopping

```gherkin
Feature: US-23 Create and organize shopping lists
  Scenario: Create and rename an owned list
    Given a Member is authenticated
    When the Member creates a list and later gives it a valid new name
    Then the owned list is stored with the new name

  Scenario Outline: Change an owned list status
    Given a Member owns a list in "<initial_state>" status
    When the Member performs "<action>"
    Then the list status becomes "<result_state>"

    Examples:
      | initial_state | action  | result_state |
      | open          | archive | archived     |
      | archived      | reopen  | open         |

  Scenario: Delete a list without deleting confirmed pantry stock
    Given a Member owns a list containing an item linked to a confirmed pantry lot
    When deletion is allowed and the Member deletes the list
    Then removable list data is deleted according to preservation rules
    But the confirmed pantry lot remains stored

  Scenario: Prevent cross-owner list access
    Given another Member owns a shopping list
    When the authenticated Member requests or mutates that list
    Then access is denied without revealing private list data

Feature: US-24 Maintain shopping-list items
  Scenario: Add and update a valid item
    Given a Member owns an open shopping list
    When the Member adds a canonical ingredient with a positive compatible quantity
    Then the item is stored on that list
    And the Member can edit its quantity and purchase status

  Scenario: Merge a duplicate compatible unpurchased ingredient
    Given an open list contains an unpurchased ingredient row
    When the Member adds the same ingredient in a compatible unit
    Then the requested quantities are merged according to the documented rule
    And at most one compatible unpurchased row remains for that ingredient

  Scenario Outline: Reject an invalid item mutation
    Given a Member owns a shopping list
    When the Member submits "<invalid_condition>"
    Then the mutation is rejected
    And existing item data remains unchanged

    Examples:
      | invalid_condition                 |
      | a nonpositive quantity            |
      | an incompatible unit              |
      | a change to another owner's item  |

Feature: US-25 Add recipe shortages to a shopping list
  Scenario: Add shortages to a new list
    Given current positive shortages exist for a visible recipe and selected servings
    When the Member chooses a new shopping list
    Then the list is created for the Member
    And one valid item is added for each positive shortage
    And zero-shortage lines are omitted

  Scenario: Merge shortages into an existing open list
    Given the Member owns an open list with compatible unpurchased rows
    When recipe shortages are imported
    Then the list is locked while compatible quantities are merged once
    And the final values equal the existing quantities plus the shortages

  Scenario: Handle an incompatible or purchased merge target
    Given a shortage targets an incompatible or already purchased row
    When the Member imports shortages
    Then the documented conflict or new-row outcome is returned
    And existing confirmed purchase data is not overwritten

Feature: US-26 Confirm a purchased item into the pantry
  Scenario: Confirm a purchase exactly once
    Given a Member owns a purchased shopping item with no linked pantry lot
    When the Member submits valid lot details for confirmation
    Then the item is locked
    And exactly one owned pantry lot is created
    And the item is linked to that lot in the same transaction

  Scenario: Retry an already completed confirmation
    Given a shopping item is already linked to a pantry lot
    When the Member repeats confirmation
    Then the existing linked lot is returned
    And no additional pantry stock is created

  Scenario: Roll back a failed confirmation
    Given a failure is injected after pantry-lot insertion but before item linking
    When the Member confirms the purchase
    Then the transaction is rolled back
    And no new pantry lot, purchase flag, or link is committed

  Scenario: Handle simultaneous confirmation requests
    Given an unconfirmed owned shopping item exists
    When two confirmation requests execute concurrently
    Then one request creates and links exactly one pantry lot
    And the other returns the existing result or a controlled conflict
    And duplicate stock is never created
```

## Consumption and Reporting

```gherkin
Feature: US-27 Record cooking from pantry stock
  Scenario: Cook a recipe with sufficient stock
    Given a Member selects a visible recipe and positive servings
    And sufficient compatible owned stock exists across multiple lots
    When the Member records cooking
    Then eligible lots are locked in deterministic FIFO order
    And older eligible quantities are deducted before later quantities
    And one cooking event and one detail per deducted lot are stored atomically
    And detail totals equal the committed lot reductions

  Scenario: Reject cooking with insufficient stock
    Given the Member lacks enough usable stock for at least one recipe requirement
    When the Member records cooking
    Then the transaction is rolled back
    And no lot balance or event row changes
    And the missing amounts are reported

  Scenario: Prevent negative stock under concurrent cooking
    Given only one final usable quantity remains
    When two cooking requests concurrently require that quantity
    Then at most one request commits the deduction
    And the other receives controlled insufficient-stock feedback
    And no pantry balance becomes negative

Feature: US-28 Record direct ingredient consumption
  Scenario: Record valid direct consumption
    Given a Member owns sufficient compatible lots for one ingredient
    When the Member records a positive direct-use quantity
    Then stock is deducted in FIFO order
    And one direct-consumption event with no recipe or servings is stored
    And per-lot detail totals equal the deduction

  Scenario Outline: Reject invalid direct consumption
    Given a Member is authenticated
    When the request contains "<invalid_condition>"
    Then no stock or event data is changed
    And a validation or authorization error is returned

    Examples:
      | invalid_condition                  |
      | a nonpositive quantity             |
      | mixed ingredients                  |
      | an incompatible unit               |
      | insufficient stock                 |
      | another Member's pantry lot        |

Feature: US-29 Filter my consumption history
  Scenario Outline: Filter owned events
    Given a Member has cooking and direct-use events on different dates
    And another Member has events
    When the Member filters history by "<filter_type>"
    Then only owned matching events and their details are returned
    And results are paginated

    Examples:
      | filter_type  |
      | date range   |
      | event type   |
      | recipe       |

  Scenario: Deny access to another Member's event
    Given another Member owns a consumption event
    When the authenticated Member requests its identifier
    Then the response is 403 or a privacy-preserving 404
    And no event details are revealed

Feature: US-30 Correct a recorded consumption event
  Scenario: Correct an owned event atomically
    Given a Member owns a consumption event with per-lot deductions
    And the replacement values can be allocated
    When the Member submits the correction
    Then the event and all affected lots are locked in deterministic order
    And the original deductions are restored
    And replacement deductions and details are applied
    And all changes commit in one transaction

  Scenario: Preserve the original event when correction fails
    Given an owned event has valid original deductions
    And its replacement cannot be allocated or a failure is injected
    When the Member submits the correction
    Then restoration and replacement changes are rolled back
    And the original event, details, and balances remain unchanged

Feature: US-31 Delete an incorrect consumption event
  Scenario: Restore stock and delete an owned event
    Given a Member owns a consumption event
    When the Member confirms deletion
    Then affected lots and the event are locked
    And the recorded deductions are restored
    And the event and its details are removed in the same transaction

  Scenario: Roll back an interrupted event deletion
    Given a failure is injected after stock restoration but before event removal
    When the Member deletes the event
    Then the transaction is rolled back
    And the original event, details, and stock balances remain unchanged

Feature: US-32 Review monthly consumption summaries
  Scenario: Display correct monthly aggregates
    Given a Member has events inside and outside a selected month
    When the Member opens the monthly report
    Then ingredient quantities are aggregated in canonical units for that Member
    And event counts include each event once regardless of detail-row count
    And the most frequently cooked recipes are displayed
    And adjacent-month events are excluded using half-open boundaries

  Scenario: Display an empty month
    Given a Member has no events in the selected month
    When the monthly report is opened
    Then a clear empty state is displayed
    And no error or misleading partial total is shown
```

## Administration

```gherkin
Feature: US-33 Disable accounts and manage roles
  Scenario: Admin disables an account
    Given an active Admin is authenticated
    When the Admin disables a permitted target account
    Then the target account becomes inactive
    And its existing protected sessions are rejected

  Scenario: Admin changes an allowed role
    Given an active Admin is authenticated
    When the Admin applies a valid role transition
    Then the new role is stored
    And previously issued target-account tokens are invalidated

  Scenario: Member cannot administer accounts
    Given a Member is authenticated
    When the Member attempts to disable an account or change a role
    Then the response status is 403
    And no account access state changes

  Scenario: Admin role does not bypass private ownership
    Given an Admin is authenticated
    When the Admin guesses a Member's pantry, shopping, or consumption identifier
    Then ordinary owner-scoped access rules deny the request
    And no private data is returned

Feature: US-34 Unpublish a recipe with a moderation reason
  Scenario: Admin moderates a published recipe
    Given an Admin is authenticated
    And a published recipe exists
    When the Admin submits a nonblank moderation reason
    Then the recipe returns to draft
    And the reason is stored
    And the recipe no longer appears in public search

  Scenario: Reject unauthorized moderation
    Given a Member does not own another Member's published recipe
    When the Member attempts to moderate it
    Then the response status is 403
    And the recipe status and moderation reason remain unchanged

Feature: US-35 Correct and resubmit a moderated recipe
  Scenario: Owner reviews a moderation reason
    Given a recipe was moderated to draft with a reason
    When its owner opens the recipe editor
    Then the owner can see the saved moderation reason
    And other Members cannot see the private draft or reason

  Scenario: Owner corrects and republishes a moderated recipe
    Given the owner has corrected the moderated draft
    And publication requirements are satisfied
    When the owner requests publication
    Then the recipe becomes published again
    And it returns to public search
```

## Quality, Operations, and Release

```gherkin
Feature: US-36 Complete core workflows on mobile and desktop
  Scenario Outline: Use a core workflow at a supported width
    Given the viewport width is "<width>"
    When a user completes "<workflow>"
    Then visible labels identify every input
    And status is communicated with text rather than color alone
    And primary information and actions remain usable
    And keyboard focus and validation feedback remain understandable

    Examples:
      | width  | workflow          |
      | 360px  | add pantry food   |
      | 360px  | find a recipe     |
      | 360px  | create a list     |
      | desktop | add pantry food  |
      | desktop | find a recipe    |
      | desktop | create a list    |

  Scenario: Peer usability acceptance
    Given five peer testers receive no task guidance
    When each tester performs add-food, find-recipe, and create-list tasks
    Then each task outcome and duration is recorded
    And the target is completion within three minutes per task
    And observed usability findings are documented

Feature: US-37 Receive consistent validation and error feedback
  Scenario Outline: Map an application failure safely
    Given a service raises "<error_type>"
    When the controller handles the request
    Then the response status is "<status>"
    And a recoverable user-facing message is shown
    And no SQL, secret, stack trace, or private data is exposed

    Examples:
      | error_type           | status |
      | ValidationError      | 400    |
      | AuthenticationError  | 401    |
      | AuthorizationError   | 403    |
      | NotFoundError        | 404    |
      | ConflictError        | 409    |
      | unexpected failure   | 500    |

  Scenario: Redirect after a successful form mutation
    Given a user submits a valid state-changing form
    When the mutation commits
    Then the controller redirects to a safe GET endpoint
    And refreshing the resulting page does not repeat the mutation

Feature: US-38 Enforce authentication, ownership, roles, and request safety
  Scenario Outline: Protect a private resource
    Given a private resource belongs to a Member
    When a request uses "<attack_case>"
    Then access is rejected
    And no private data is read or mutated

    Examples:
      | attack_case                         |
      | no authentication token             |
      | an expired token                    |
      | another Member's resource ID        |
      | a Member token for an Admin action  |
      | a missing or invalid CSRF token     |

  Scenario Outline: Reject unsafe input without mutation
    Given the database is in a known committed state
    When a request contains "<unsafe_input>"
    Then the request is rejected or safely normalized
    And the database remains consistent
    And the value is never concatenated into SQL

    Examples:
      | unsafe_input              |
      | an SQL injection payload  |
      | a malformed date          |
      | a nonallowlisted order    |
      | an incompatible unit      |
      | a nonpositive quantity    |

Feature: US-39 Build the schema and fixtures reproducibly
  Scenario: Create the schema from a clean checkout
    Given a supported empty MySQL database
    And documented environment configuration
    When numbered raw-SQL migrations are applied in order
    Then the expected schema, keys, constraints, and indexes exist
    And no ORM or automatic schema generator is used

  Scenario: Generate repeatable fixtures
    Given the same fixed seed and empty schema
    When functional and benchmark data are generated twice in isolated databases
    Then both generated datasets have the same expected row counts and key values
    And all generated rows satisfy database constraints

Feature: US-40 Measure pantry and recipe query performance
  Scenario: Measure pantry performance
    Given the documented environment contains 100000 pantry rows
    And warm-up requests have completed
    When 100 documented pantry search and expiry requests are measured
    Then sample latencies, hardware, query mix, and EXPLAIN plans are recorded
    And the target p95 server latency is below 500 milliseconds

  Scenario: Measure recipe-matching performance
    Given the documented environment contains 200 recipes
    And the separate fixture contains 50000 recipe lines
    And warm-up requests have completed
    When 100 paginated matching requests are measured
    Then sample latencies, pagination, hardware, and EXPLAIN plans are recorded
    And the target p95 server latency is below one second

Feature: US-41 Prove transaction and concurrency behavior
  Scenario Outline: Roll back at a multi-step transaction boundary
    Given the database is in a known committed state
    And a failure is injected at "<boundary>"
    When the associated workflow executes
    Then all changes from that workflow are rolled back
    And the last committed state remains intact

    Examples:
      | boundary                              |
      | purchase after lot insertion          |
      | cooking after balance deduction       |
      | correction after original restoration |
      | deletion after balance restoration    |

  Scenario Outline: Preserve integrity under concurrent requests
    Given two requests target the same final resource
    When both execute "<operation>" concurrently
    Then row locks serialize the conflicting work
    And the documented winner and loser outcomes are returned
    And "<integrity_rule>"

    Examples:
      | operation             | integrity_rule                    |
      | purchase confirmation | exactly one pantry lot is created |
      | final-stock cooking   | stock never becomes negative      |

Feature: US-42 Rehearse database backup and recovery
  Scenario: Restore a verified backup safely
    Given a populated source database and documented backup command
    When a maintainer creates a backup and restores it into an isolated database
    Then the source database remains unchanged
    And restored row counts match the source
    And documented key aggregates match the source
    And no secret is written into project documentation

Feature: US-43 Reproduce setup, verification, and demonstration
  Scenario: Reproduce the release from a clean checkout
    Given the selected repository revision and example configuration
    When a reviewer follows the documented setup procedure
    Then the reviewer can migrate the schema
    And generate the stable seed data
    And run functional, authorization, failure, concurrency, and performance checks
    And execute backup and restore verification
    And complete the fixed demonstration without manual database repair

  Scenario: Report release evidence and limitations
    Given release verification has completed
    When the results are consolidated
    Then the tested revision, environment, results, defects, and limitations are documented
    And every release-blocking ownership, security, data-loss, negative-stock, or transaction defect is resolved before acceptance
```

