# Huntd QA Portfolio

**Web & Android Testing · GraphQL Investigation · Defect Analysis · Test Documentation**

A QA portfolio documenting testing of Huntd, a web3-oriented recruitment platform with shared GraphQL backend services across web and mobile applications.

The work focuses on more than reporting symptoms: reproducing failures, isolating triggering conditions, comparing client and backend behavior, and building evidence-based root-cause hypotheses with browser DevTools, GraphQL response analysis, Apollo cache inspection, ADB logcat, and database error investigation.

## Navigation

- [Overview](#overview)
- [Key Findings](#key-findings)
- [Bug Showcase](#bug-showcase)
- [Cross-Module Defect Patterns](#cross-module-defect-patterns)
- [Test Documentation](#test-documentation)
- [Web Testing Artifacts](#web-testing-artifacts)
- [Mobile Testing Artifacts](#mobile-testing-artifacts)
- [Skills Demonstrated](#skills-demonstrated)
- [Repository Structure](#repository-structure)

## Overview

This repository contains testing artifacts for both the web and Android applications, including test planning, requirements traceability, test-run summaries, defect reports, and supporting screenshots.

| Area | Artifacts |
|---|---|
| Test planning | [Test Plan](test-plan.md) |
| Permissions | [Permission Testing Table](permission-testing-table.md) |
| Web application | [Defect Reports](web-app/defect-reports/), [RTM](web-app/requirements-traceability-matrix.md), [TestRail Run Summary](web-app/testrail-test-runs-summary.md) |
| Mobile application | [Defect Reports](mobile-app/defect-reports/), [RTM](mobile-app/requirements-traceability-matrix.md), [TestRail Run Summary](mobile-app/testrail-test-runs-summary.md) |

## Key Findings

The examples below illustrate issues that required investigation beyond surface-level functional testing.

- **Database constraint and transaction failure:** A `varchar(255)` constraint without a corresponding client-side guard caused silent data loss in multiple flows. During creation, PostgreSQL error `25P02` aborted the transaction block and prevented subsequent saves until page reload, without clear user feedback.
- **Broken recruiter–candidate relationship continuity:** Profile IDs regenerated during edit cycles (`13173 → 13176 → 13179)), causing duplicate chats, inflated contact history, and previously contacted candidates to appear new.
- **OAuth state inconsistency:** Cancelled authorization attempts left ghost tokens in the database. Apollo cache inspection showed all three providers using the same `providerId` from the initiating account.
- **Invisible tooltip intercepting clicks:** A `Tooltip_toolTipTop__G_POy` element with `opacity: 0` remained in the DOM at `z-index: 12`, intercepting clicks on modal dialogs across five pages.
- **Role-specific pagination crash:** The Candidates List crashed for Recruiter during pagination when code accessed `workPlaces[n].title` on a profile where `workPlaces: null`. The Candidate role was unaffected with the same data up to offset 400.
- **Deleted chat records and failed messaging:** Deleted chat records persisted in the database. Messaging to deleted chats succeeded, then subsequent send attempts triggered `SequelizeUniqueConstraintError` (`code: 23505`); the behavior was investigated for both Recruiter and Candidate roles.

## Bug Showcase

Selected defect reports demonstrating reproduction, technical investigation, and supporting evidence.

| Report | Finding | Investigation |
|---|---|---|
| [HUNTD-62](web-app/defect-reports/HUNTD-62-list-crashes-on-workplace-null.md) | Candidates List crashes for Recruiter during pagination | Isolated the role-specific crash and identified offending profiles through a DevTools console query at the failing offset |
| [HUNTD-58](web-app/defect-reports/HUNTD-58-duplicate-chats-and-contact-his.md) | Profile reactivation creates duplicate chats and inflates contact history | Traced profile ID regeneration across edit cycles and its effect on existing recruiter–candidate relationships |
| [HUNTD-73](web-app/defect-reports/HUNTD-73-cancelled-oauth-creates-connections.md) | Cancelled OAuth attempts persist ghost tokens for all three providers | Compared provider state and confirmed the shared `providerId` through Apollo cache inspection |
| [HUNTD-65](web-app/defect-reports/HUNTD-65-chat-records-persist-after-deletion.md) | Deleted chats persist and subsequent messaging triggers a uniqueness error | Analysed GraphQL mutations for both roles and captured error `23505` |
| [HUNTD-81](web-app/defect-reports/HUNTD-81-form-creation-resets-on-255-char-limit.md) | `varchar(255)` violation triggers PostgreSQL `25P02` | Followed the failure beyond the initial mutation to identify its effect on later saves in the same session |

Browse the complete [web defect-report directory](web-app/defect-reports/) and [mobile defect-report directory](mobile-app/defect-reports/).

## Cross-Module Defect Patterns

Testing revealed recurring defect classes across otherwise unrelated areas of the application.

- **Silent persistence failures:** the same `varchar(255)` root cause affected Experience create/edit flows and Recruiter Company Details, with different symptoms including silent discard, form reset, and transaction abort.
- **UI state not reset after interaction:** disabled OAuth buttons stayed disabled; deleted-chat UI remained visible; ghost tokens persisted; and required-field validation was not re-triggered after clearing fields.
- **Shared component defects:** the Tooltip component, Technologies multi-select, Job Experience dropdown, and salary fields showed repeated problems across pages where the components were reused.

Recognising these patterns helps distinguish isolated symptoms from defects that may have a shared underlying cause.

## Test Documentation

- [Test Plan](test-plan.md) — scope, approach, and planning artifacts.
- [Permission Testing Table](permission-testing-table.md) — role and permission-focused coverage.

## Web Testing Artifacts

- [Web defect reports](web-app/defect-reports/)
- [Web evidence screenshots](web-app/evidence-screenshots/)
- [Requirements Traceability Matrix](web-app/requirements-traceability-matrix.md)
- [TestRail test-run summary](web-app/testrail-test-runs-summary.md)

## Mobile Testing Artifacts

- [Mobile defect reports](mobile-app/defect-reports/)
- [Mobile evidence screenshots](mobile-app/evidence-screenshots/)
- [Requirements Traceability Matrix](mobile-app/requirements-traceability-matrix.md)
- [TestRail test-run summary](mobile-app/testrail-test-runs-summary.md)

## Skills Demonstrated

- **Test design:** equivalence partitioning, boundary value analysis, decision table testing, state transition testing, and use case testing.
- **Grey-box investigation:** DevTools Network, Console, and Elements panels; GraphQL mutation and response analysis; Apollo cache inspection; and ADB logcat.
- **Mobile testing:** Android device testing, ADB through PowerShell, scrcpy screen mirroring, and React Native error identification.
- **Defect reporting:** reproducible steps, evidence collection, root-cause hypotheses, and cross-module pattern recognition.
- **Test management:** TestRail test cases, runs, and reports; Jira bug tracking, epics, and stories.
- **Documentation:** test planning, requirements traceability, permission testing, and test-run summaries.

## Repository Structure

```text
.
├── README.md
├── test-plan.md
├── permission-testing-table.md
├── web-app/
│   ├── defect-reports/
│   ├── evidence-screenshots/
│   ├── testrail-test-runs-summary.md
│   └── requirements-traceability-matrix.md
└── mobile-app/
    ├── defect-reports/
    ├── evidence-screenshots/
    ├── testrail-test-runs-summary.md
    └── requirements-traceability-matrix.md
```

---

[Back to top](#huntd-qa-portfolio)
