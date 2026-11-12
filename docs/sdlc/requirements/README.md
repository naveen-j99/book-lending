# Product Requirements

> Maintained by QuorQ AI SDLC. Text between `sdlc:auto` markers is regenerated; anything outside them is yours and is kept.

<!-- sdlc:auto:start narrative -->
## Summary

### REQ-SCOPE-001 · draft · source: requirements:scope

Defines the lightweight delivery scope for v1: what capabilities are built now and what is explicitly excluded.

Delivery scope: lightweight. Requirements version 1.
In scope: email/password registration & sign-in; catalog add/edit/delete (title, author, copy counts); catalog search/browse by title & author with availability; instant self-service borrowing with 14-day due date; 5-active-loan cap per member; block borrowing when zero copies remain; admin-only mark-returned that restores copy count; member dashboard of current loans/due dates; admin loans overview of all active loans; Users/Books/Loans data model; getting-started and API documentation; no renewals; no fees/payment processing.
Out of scope -- not designed or built: reservation/hold waitlist when a book is unavailable; email sending (SendGrid was removed from the stack, so no due-soon/overdue email reminders despite being mentioned in the original understanding).

### REQ-STACK-001 · approved · source: requirements:stack

Technology stack approved by the product owner; design must use exactly these, no substitutions.

- Backend: FastAPI 0.115 -- chosen for simple server-side rules (5-loan cap, block zero-copy borrow, 14-day due date, admin-only returns) and because it is the platform default.
- Cloud: AWS -- hosts web app, database and outbound reminder-check infra; always-on, scales from launch to thousands of members; platform default.
- Database: PostgreSQL 16 -- enforces the three-table relational structure (Users, Books, Loans) and handles concurrent copy-count decrements safely; platform default.
- Frontend: React 19 -- builds the responsive member/admin screen-at-a-time interface; platform default.

**Related:** EPIC-007

### REQ-SVC-001 · approved · source: requirements:services

Redis-backed queue approved as the only runtime service, for scheduling due-soon/overdue checks in the background.

Runtime need: Redis (as a queue), container redis:7-alpine, runs inside the platform. Required API key/env var: REDIS_URL. Note: the queue is for checking due-soon/overdue status on a schedule; since email sending is out of scope, no outbound email is actually dispatched by this service in this version (see AC-039).

**Related:** EPIC-006

### EPIC-001 · draft · source: requirements:EPIC-001

Accounts, registration and sign-in. Members self-register with email/password; admin accounts are only seeded or promoted, never self-registered.

- US-001 (critical) Member self-registration: valid unused email + password creates a Member account (AC-001); duplicate email is rejected (AC-002); invalid email format or blank password is rejected with guidance (AC-003); password is stored only as a hash, never plain text (AC-004).
- US-002 (critical) Sign in/out: correct credentials sign the user in to their role's dashboard (AC-005); wrong credentials give a generic auth-failed message (AC-006); signing out ends the session (AC-007); an idle session beyond its validity period is rejected on next request, requiring re-sign-in (AC-008).
- US-003 (high) Admin accounts seeded/promoted only: public self-registration always yields a Member account, never Admin (AC-009); signed-in Admins can reach catalog/loan management (AC-010); Members attempting admin-only features are denied (AC-011).

**Related:** US-001, US-002, US-003, NFR-001, NFR-002, NFR-003

### EPIC-002 · draft · source: requirements:EPIC-002

Book catalog management by admins: add, edit, delete. A book record holds only title, author and copy counts.

- US-004 (critical) Add a book: admin adds title + author + copy count >=1, creating the book with total and available copies set to that count (AC-012); blank title/author or negative count is rejected with guidance (AC-013); record holds only title/author/copy counts, no cover/ISBN/genre (AC-014).
- US-005 (high) Edit a book: title/author changes are saved and visible to members (AC-015); changing total copy count on a book with active loans adjusts available copies so they never go negative or exceed total minus on-loan copies (AC-016); invalid edits (blank title, total below copies on loan) are rejected with reason (AC-017).
- US-006 (medium) Delete a book: a book with no active loans is deleted and disappears from search/browse (AC-018); a book with active loans cannot be deleted, and the admin is told why (AC-019).

**Related:** US-004, US-005, US-006

### EPIC-003 · draft · source: requirements:EPIC-003

Members search and browse the catalog by title/author and see current copy availability.

- US-007 (high) Browse with availability: catalog listing shows title, author and available copies (AC-020); a book with zero available copies is shown as unavailable (AC-021).
- US-008 (high) Search by title/author: partial title or author matches return results with current availability (AC-022); no matches shows an empty-results message, not an error (AC-023). Search is limited to title and author only, since no other metadata is stored.

**Related:** US-007, US-008

### EPIC-004 · draft · source: requirements:EPIC-004

Self-service instant borrowing with a fixed 14-day due date, blocked at zero copies or the 5-active-loan cap. No renewals, no waitlist, no fees.

- US-009 (critical) Borrow an available book: if copies available and member is below cap, borrowing creates a loan, decrements available copies by one, and sets due date 14 days out (AC-024); no payment/fee is ever requested (AC-025); due date is fixed at 14 days, no renewal/extension (AC-026).
- US-010 (critical) Blocked at zero copies: borrowing a book with zero available copies is refused, no loan created, member told it's unavailable (AC-027); no hold/reservation/waitlist is offered (AC-028).
- US-011 (high) Blocked at 5-loan cap: a member with 5 active loans is refused another borrow and told they've reached the limit (AC-029); once one of those loans is marked returned, borrowing is allowed again (AC-030).

**Related:** US-009, US-010, US-011, NFR-006

### EPIC-005 · draft · source: requirements:EPIC-005

Only admins can mark a loan returned; members cannot self-return. Marking returned restores the copy count.

- US-012 (critical) Admin marks returned: marking an active loan returned closes it and increments the book's available copies by one (AC-031); attempting to mark an already-returned loan again makes no further change and the admin is told it's already returned (AC-032).
- US-013 (high) No member self-return: a member viewing their active loans has no self-return action available (AC-033) -- the physical return must be confirmed by staff before the copy is freed.

**Related:** US-012, US-013

### EPIC-006 · draft · source: requirements:EPIC-006

Member dashboard of current loans/due dates, and admin overview of all active loans, both with overdue flagging. No email reminders in this version.

- US-014 (high) Member dashboard: shows each active loan's book title and due date (AC-034); a loan past its due date is clearly flagged as overdue (AC-035); a member with no active loans sees an empty state (AC-036).
- US-015 (high) Admin loans overview: shows every active loan across all members with member, book title and due date (AC-037); loans past due date are flagged as overdue to distinguish from on-time loans (AC-038).
- Note/conflict (AC-039): the original understanding mentioned due-soon/overdue email reminders, but email is out of scope for this MVP -- overdue handling here is on-screen flagging only, no emails sent.

**Related:** US-014, US-015, NFR-004, NFR-009

### EPIC-007 · draft · source: requirements:EPIC-007

Core data model: three related entities -- Users, Books, Loans -- underpinning accounts, catalog and borrowing at a scale of thousands of members/books.

- US-016 (high): Users entity holds role (member/admin) and hashed password; Books entity holds title, author, total and available copy counts; Loans entity links a user to a book with borrow date, due date and returned state (AC-040).
- Borrow/return operations must keep Loans records and the Book's available copy count consistent: never negative, never above total copies (AC-041).

**Related:** US-016, NFR-006

### EPIC-008 · draft · source: requirements:EPIC-008

Getting-started guide for operators and API documentation for developers integrating with the app.

- US-017 (medium): a getting-started guide lets an operator configure and run the app and seed/promote an initial admin account for a new deployment (AC-042).
- US-018 (medium): API documentation describes endpoints for authentication, catalog management, search/browse, borrowing and returns, including inputs and responses (AC-043).

**Related:** US-017, US-018

### NFR-001 · draft · source: requirements:NFR-001

Security: passwords must be stored hashed, never retained or logged in plain text.

Applies to US-001 (registration) and US-016 (Users data model).

**Related:** US-001, US-016

### NFR-002 · draft · source: requirements:NFR-002

Security: sessions expire after inactivity and sign-out ends them; failed sign-in uses a generic message that doesn't reveal whether the email is registered.

Applies to US-002 (sign in/out).

**Related:** US-002

### NFR-003 · draft · source: requirements:NFR-003

Security: admin powers are restricted -- public registration can only create member accounts; admin role is seeded/promoted only; catalog/loan management is admin-only.

Applies to US-003 (admin account provisioning).

**Related:** US-003

### NFR-004 · draft · source: requirements:NFR-004

Privacy: a member's dashboard shows only their own loans and due dates, never other members' borrowing.

Applies to US-014 (member dashboard).

**Related:** US-014

### NFR-005 · draft · source: requirements:NFR-005

Scalability: service must support growth from quiet launch to on the order of thousands of members and books without re-platforming.

Applies to the whole system.

### NFR-006 · draft · source: requirements:NFR-006

Scalability/concurrency: borrowing and returning must keep a book's available copy count consistent (never below zero, never above total) even under simultaneous access, so two members can't take the last copy at once.

Applies to US-009, US-010 (borrowing) and US-016 (data model).

**Related:** US-009, US-010, US-016

### NFR-007 · draft · source: requirements:NFR-007

Compatibility: responsive, mobile-friendly web app that works in current mobile and desktop browsers, not a native app.

Applies to the whole system.

### NFR-008 · draft · source: requirements:NFR-008

Availability: the public service must be reliably available to members and admins throughout the day.

Applies to the whole system.

### NFR-009 · draft · source: requirements:NFR-009

Usability (accepted risk): no due-soon or overdue email reminders are sent in this version; overdue is on-screen flagging only, viewed when dashboards are opened.

Applies to US-014 (member dashboard) and US-015 (admin loans overview). Accepted as a known limitation of this scope.

**Related:** US-014, US-015
<!-- sdlc:auto:end narrative -->

<!-- sdlc:auto:start summary -->
**Requirements version:** v1  
**Delivery scope:** lightweight  
**Approved:** 2026-11-12

**Capabilities in scope:** Email and password registration and sign-in, Catalog management: add, edit, delete title, author and copy counts, Catalog search and browse by title and author with availability, Instant self-service borrowing that sets a 14-day due date, Block borrowing at the 5-loan cap and when zero copies remain, Mark a loan returned and restore the copy count, Member dashboard of current loans and due dates, Loans overview showing all active loans, Users, Books and Loans data model, Getting-started and API documentation, Per-member cap of 5 active loans, Fixed 14-day loan period with no renewals, Returns recorded by staff, not self-returned, No hold or waitlist when a book is unavailable, Books hold only title, author and copy counts, Free service with no fees or payment processing

**Approved technology stack:**

| Layer | Choice | Rationale |
|---|---|---|
| backend | FastAPI | The rules that matter here are all server-side and simple: enforce the 5-loan cap, block borrowing a book with zero copies, set the 14-day due date, and let only admins mark returns. FastAPI is a straightforward way to express those rules and the three-table data flow, and it is the platform default, keeping the build predictable for a modest free service. |
| cloud | AWS | A public community library app needs reliable, always-on hosting that scales from a quiet launch to thousands of members without re-platforming. AWS covers the web app, database and the outbound reminder emails in one place, and is the platform default, so your free service runs on well-trodden infrastructure. |
| database | PostgreSQL | You confirmed exactly three related tables — Users, Books and Loans — and borrowing must safely decrement copies without two members grabbing the last copy at once. PostgreSQL enforces those relationships and handles that concurrency correctly out of the box, and comfortably covers thousands of members and books. It is also the platform default. |
| frontend | React | Your library is a responsive, mobile-friendly web app with two clear views — a member side for searching, borrowing and seeing due dates, and an admin side for managing the catalog and loans. React handles exactly this kind of interactive, screen-at-a-time interface well, and it is the default this platform builds, so there is nothing exotic to maintain for a community library. |
<!-- sdlc:auto:end summary -->

<!-- sdlc:auto:start epics -->
## EPIC-001: Accounts, Registration and Sign-in

Email-and-password registration and sign-in for the public community library. Members self-register; admin accounts are seeded or promoted rather than self-registered as admin. Sign-in and session security are expressed as checkable outcomes: passwords are stored hashed, sessions expire, and signing out ends the session.

### US-001: Member self-registration with email and password

**Priority:** critical

> As a member of the public, I want to register for an account using my email address and a password, so that I can sign in and borrow books from the library.

| ID | Given | When | Then |
|---|---|---|---|
| AC-001 | I am a visitor on the registration page | I submit a valid, not-yet-registered email address and a password | a Member account is created for me and I can sign in with those credentials |
| AC-002 | an account already exists for a given email address | I try to register again with that same email address | registration is rejected and I am told the email is already in use |
| AC-003 | I submit the registration form | the email is not a valid email format or the password is blank | registration is rejected and I am shown what to correct |
| AC-004 | a new account has been created | the password is stored | it is stored as a hash and never retained or displayed in plain text |

### US-002: Sign in and sign out

**Priority:** critical

> As a registered user (member or admin), I want to sign in with my email and password and sign out when finished, so that only I can act on my account and my session does not stay open indefinitely.

| ID | Given | When | Then |
|---|---|---|---|
| AC-005 | I have a registered account | I submit my correct email and password | I am signed in and taken to my dashboard with the role (member or admin) that my account holds |
| AC-006 | I am on the sign-in page | I submit an email/password combination that does not match an account | I am not signed in and I am shown a generic authentication-failed message |
| AC-007 | I am signed in | I choose to sign out | my session ends and I must sign in again to access any member or admin area |
| AC-008 | I am signed in and leave the session idle beyond its validity period | I next make a request | the expired session is rejected and I am required to sign in again |

### US-003: Admin accounts are seeded or promoted, not self-registered

**Priority:** high

> As a library operator, I want admin accounts to be seeded or promoted rather than available through public sign-up, so that members of the public cannot grant themselves catalog and loan management powers.

| ID | Given | When | Then |
|---|---|---|---|
| AC-009 | the public registration flow | a visitor completes self-registration | the resulting account is always a Member and never an Admin (admin role is assigned by seeding or promotion only — assumption: no self-service admin sign-up) |
| AC-010 | an account that holds the Admin role | that admin signs in | they can access the catalog and loan management features reserved for admins |
| AC-011 | an account that holds only the Member role | that member attempts to reach an admin-only feature | access is denied |

## EPIC-002: Book Catalog Management

Admins maintain the book catalog. A book record holds only title, author and copy counts — no cover images, ISBN or genre metadata. Admins can add, edit and delete books.

### US-004: Add a book to the catalog

**Priority:** critical

> As a library admin, I want to add a book with its title, author and number of copies, so that members can find and borrow it.

| ID | Given | When | Then |
|---|---|---|---|
| AC-012 | I am signed in as an admin | I add a book with a title, an author and a copy count of one or more | the book is created in the catalog with its total and available copies set to that count |
| AC-013 | I am adding a book | I leave the title or author blank, or enter a negative copy count | the book is not created and I am shown what to correct |
| AC-014 | the catalog record for a book | it is stored | it holds only title, author and copy counts (assumption: no cover image, ISBN or genre fields) |

### US-005: Edit an existing book

**Priority:** high

> As a library admin, I want to edit a book's title, author and copy counts, so that the catalog stays accurate when details change or stock is adjusted.

| ID | Given | When | Then |
|---|---|---|---|
| AC-015 | an existing book in the catalog | I update its title or author | the change is saved and shown to members browsing the catalog |
| AC-016 | a book that currently has active loans | I change its total copy count | the available copy count is adjusted consistently so that available copies never fall below zero or exceed the new total minus copies currently on loan |
| AC-017 | I am editing a book | I submit an invalid value such as a blank title or a total count lower than the copies currently on loan | the change is rejected and I am told why |

### US-006: Delete a book

**Priority:** medium

> As a library admin, I want to delete a book from the catalog, so that titles the library no longer holds are removed.

| ID | Given | When | Then |
|---|---|---|---|
| AC-018 | a book that has no active loans | I delete it | it is removed from the catalog and no longer appears in search or browse results |
| AC-019 | a book that currently has one or more active loans | I attempt to delete it | the deletion is prevented and I am told the book still has active loans |

## EPIC-003: Catalog Search and Browse

Members search and browse the catalog by title and author and see current availability (copies remaining). Search is limited to title and author because book records hold no other metadata.

### US-007: Browse the catalog with availability

**Priority:** high

> As a member, I want to browse the list of books and see how many copies are available, so that I can decide what I can borrow right now.

| ID | Given | When | Then |
|---|---|---|---|
| AC-020 | I am signed in as a member | I open the catalog | I see books listed with title, author and the number of copies currently available |
| AC-021 | a book whose available copies is zero | I view it in the catalog | it is shown as unavailable to borrow |

### US-008: Search the catalog by title and author

**Priority:** high

> As a member, I want to search for books by title or author, so that I can quickly find a specific book.

| ID | Given | When | Then |
|---|---|---|---|
| AC-022 | the catalog contains matching books | I search by part of a title or author name | the results show the books whose title or author matches, each with its current availability |
| AC-023 | no book matches my search terms | I run the search | I am shown an empty-results message rather than an error (assumption: search is by title and author only) |

## EPIC-004: Self-service Borrowing

Members borrow books instantly and self-service. Borrowing decrements available copies and sets a fixed 14-day due date. Borrowing is blocked at the per-member cap of 5 active loans and when zero copies remain. There are no renewals, no waitlist/holds, and the service is free with no fees.

### US-009: Borrow an available book

**Priority:** critical

> As a member, I want to borrow an available book in one action, so that I can take it out immediately without staff involvement.

| ID | Given | When | Then |
|---|---|---|---|
| AC-024 | a book with one or more available copies and I am below my active-loan cap | I borrow it | a loan is created for me, the book's available copies is decremented by one, and a due date 14 days from today is set |
| AC-025 | I have just borrowed a book | the loan is created | no payment, fee or charge is requested at any point (assumption: the service is free — no membership fees, late fees or payment processing) |
| AC-026 | I have an active loan | I view it | its due date is fixed at 14 days from borrowing and cannot be renewed or extended (assumption: no loan renewals/extensions) |

### US-010: Borrowing blocked when no copies are available

**Priority:** critical

> As a member, I want to be prevented from borrowing a book that has zero available copies, so that the catalog never shows more books on loan than the library holds.

| ID | Given | When | Then |
|---|---|---|---|
| AC-027 | a book whose available copies is zero | I attempt to borrow it | the borrow is refused, no loan is created, and I am told the book is currently unavailable |
| AC-028 | a book with zero available copies | I view it | I am offered no hold, reservation or waitlist option (assumption: no waitlist/holds in this version) |

### US-011: Borrowing blocked at the per-member loan cap

**Priority:** high

> As a member, I want to be stopped from borrowing once I reach my active-loan limit, so that loans are shared fairly across the community.

| ID | Given | When | Then |
|---|---|---|---|
| AC-029 | I already have 5 active (not-yet-returned) loans | I attempt to borrow another book | the borrow is refused and I am told I have reached my limit of 5 active loans (assumption: per-member cap defaults to 5) |
| AC-030 | I have 5 active loans and one of them is marked returned by an admin | I attempt to borrow again | the borrow is allowed because I am now below the cap |

## EPIC-005: Returns Handled by Admins

Only admins mark a loan returned; members cannot self-return. Marking a return closes the loan and restores the book's available copy count.

### US-012: Admin marks a loan returned

**Priority:** critical

> As a library admin, I want to mark an active loan as returned, so that the copy becomes available for others to borrow.

| ID | Given | When | Then |
|---|---|---|---|
| AC-031 | an active loan shown in the admin loans view | I mark it returned | the loan is closed and the related book's available copies is incremented by one |
| AC-032 | a loan that has already been marked returned | I attempt to mark it returned again | no further change is made to the copy count and I am told it is already returned |

### US-013: Members cannot self-return

**Priority:** high

> As a library admin, I want returns to be recorded only by staff, so that the physical return is confirmed before a copy is freed.

| ID | Given | When | Then |
|---|---|---|---|
| AC-033 | I am signed in as a member viewing my active loans | I look for a way to mark a loan returned | no self-return action is available to me (assumption: only admins mark returns) |

## EPIC-006: Dashboards and Overdue Visibility

A member dashboard shows the member their current loans and due dates. An admin loans overview shows all active loans, with overdue loans flagged for both admins and the affected member. Note: the understanding mentioned due-soon/overdue email reminders, but email is excluded from this MVP scope, so overdue handling here is limited to on-screen flagging.

### US-014: Member dashboard of current loans and due dates

**Priority:** high

> As a member, I want a dashboard listing my current loans with their due dates, so that I know what I have out and when each book is due.

| ID | Given | When | Then |
|---|---|---|---|
| AC-034 | I am signed in as a member with active loans | I open my dashboard | I see each active loan with its book title and due date |
| AC-035 | a loan of mine whose due date has passed | I view my dashboard | that loan is clearly flagged as overdue |
| AC-036 | I have no active loans | I open my dashboard | I see an empty state indicating I currently have nothing borrowed |

### US-015: Admin loans overview with overdue flags

**Priority:** high

> As a library admin, I want an overview of all active loans with overdue ones flagged, so that I can see what is out and chase books that are late.

| ID | Given | When | Then |
|---|---|---|---|
| AC-037 | there are active loans across members | I open the admin loans overview | I see every active loan with its member, book title and due date |
| AC-038 | one or more active loans are past their due date | I view the loans overview | those loans are flagged as overdue so I can distinguish them from on-time loans |
| AC-039 | the understanding referenced due-soon and overdue email reminders | this MVP version is delivered | overdue handling is limited to on-screen flagging and no emails are sent (assumption/conflict: email delivery is excluded from this scope) |

## EPIC-007: Data Model: Users, Books and Loans

The application is organised around three entities — Users, Books and Loans — on a standard relational database at modest scale (order of thousands of members and books). This foundation underpins registration, catalog, borrowing and returns.

### US-016: Core Users, Books and Loans entities

**Priority:** high

> As a developer building the app, I want a data model with Users, Books and Loans and the relationships between them, so that accounts, catalog and borrowing records are stored consistently.

| ID | Given | When | Then |
|---|---|---|---|
| AC-040 | the data model | it is defined | it includes a Users entity (with role member/admin and hashed password), a Books entity (title, author, total and available copy counts) and a Loans entity linking a user to a book with borrow and due dates and a returned state |
| AC-041 | a borrow or return operation | it changes a book's availability | the Loans record and the Book's available copy count stay consistent (available copies never negative, never above total copies) |

## EPIC-008: Getting-started and API Documentation

Documentation so that operators can set the app up and developers can integrate with its API.

### US-017: Getting-started guide

**Priority:** medium

> As a library operator setting up the app, I want a getting-started guide, so that I can run the application and seed the first admin account.

| ID | Given | When | Then |
|---|---|---|---|
| AC-042 | a new deployment | I follow the getting-started guide | I can configure, run the app, and seed or promote an initial admin account |

### US-018: API documentation

**Priority:** medium

> As a developer integrating with the app, I want documentation of the API endpoints, so that I understand how registration, catalog, borrowing and return operations are called.

| ID | Given | When | Then |
|---|---|---|---|
| AC-043 | the API documentation | I read it | it describes the available endpoints for authentication, catalog management, search/browse, borrowing and returns, including their inputs and responses |
<!-- sdlc:auto:end epics -->
