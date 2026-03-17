# CFC Phase 2 — Project Timeline

**Start Date**: March 2, 2026 (Monday)
**End Date**: April 29, 2026 (Wednesday)
**Duration**: 43 working days (excludes weekends)
**Team**: 2 developers (Backend + Frontend), 1 designer (part-time), 1 QA (part-time)

---

## Timeline Overview

```
Week 1-2   ████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░  M1: Foundation
Week 3-4   ░░░░░░░░████████████████░░░░░░░░░░░░░░░░░░░░░░  M2: Core Platform
Week 5-6   ░░░░░░░░░░░░░░░░████████████████░░░░░░░░░░░░░░  M3: User Engagement
Week 7-8   ░░░░░░░░░░░░░░░░░░░░░░░░████████████████░░░░░░  M4: Intelligence
Week 9     ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░████████████░░  M5: Polish & Release
```

---

## Milestone 1: Foundation (Days 1-10)
**March 2 — March 13, 2026**

The platform's backbone: search, authentication, and access control.

| Days | Feature | Branch | Tasks | Deliverable |
|------|---------|--------|-------|-------------|
| 1-6 | Homepage (search, results, filters, sort, trending) | `001-homepage` | 86 | Visitors can search and discover projects via AI-powered semantic search with filters, sorting, and trending section |
| 5-10 | Registration & Roles (register, login, RBAC, password recovery) | `002-registration-roles` | 94 | Users can register, select role (Org Admin / Funder), log in, and access role-appropriate features. RBAC enforced on all endpoints |

**Overlap**: Registration backend starts Day 5 while Homepage frontend is finishing.

**Milestone 1 Exit Criteria**:
- Visitors can search projects via AI semantic search with filters and sorting
- Trending projects section visible on homepage
- Users can register with role selection (Org Admin / Funder)
- Login/logout works with JWT access + refresh tokens
- RBAC enforced on all protected endpoints
- Password recovery flow functional

---

## Milestone 2: Core Platform (Days 11-20)
**March 16 — March 27, 2026**

Organizations can publish projects; Super Admins can approve them.

| Days | Feature | Branch | Tasks | Deliverable |
|------|---------|--------|-------|-------------|
| 11-18 | Project Management (create, edit, publish, delete, approve/reject, taxonomy) | `003-project-management` | 94 | Org Admins can create projects with taxonomy tags, publish for review. Super Admins approve/reject. Full audit trail |
| 18-20 | Project Map | `008-project-map` | 16 | Project detail pages show a location map with geocoded coordinates |

**Overlap**: Project Map starts Day 18 while Project Management P2 stories (edit, delete, tags) are finishing.

**Milestone 2 Exit Criteria**:
- Org Admins can create, edit, publish, and delete projects
- Super Admins can approve/reject from review queue
- Predefined taxonomy enforced on all taxonomy fields
- All project lifecycle actions audit-logged
- Project detail pages show location map
- Approved projects appear in homepage search results

---

## Milestone 3: User Engagement (Days 21-30)
**March 30 — April 10, 2026**

Funders can save, track, and receive updates about projects.

| Days | Feature | Branch | Tasks | Deliverable |
|------|---------|--------|-------|-------------|
| 21-25 | Bookmarks (toggle, my bookmarks, update tracking) | `004-bookmarks` | 50 | Funders can bookmark projects, view bookmarks page, see update indicators for changed projects |
| 24-28 | Onboarding (questionnaire, skip, preferences, recommendations) | `005-onboarding` | 45 | Post-registration onboarding captures interests. "Recommended for you" section on homepage |
| 27-30 | Notifications (center, mark read, email, opt-out) | `006-notifications` | 57 | In-app notification center with unread badge. Email notifications for bookmark changes. Opt-out toggle |

**Overlap**: Features developed in a staggered pipeline — backend of next starts while frontend of current finishes.

**Milestone 3 Exit Criteria**:
- Funders can bookmark/unbookmark from anywhere, view My Bookmarks page
- Bookmark updates tracked from project audit log
- Onboarding questionnaire captures preferences (skippable)
- Homepage shows "Recommended for you" based on preferences
- Notification Center with unread badge and mark-as-read
- Email notifications with opt-out toggle

---

## Milestone 4: Intelligence (Days 31-38)
**April 13 — April 22, 2026**

AI-powered assistant and smart matchmaking.

| Days | Feature | Branch | Tasks | Deliverable |
|------|---------|--------|-------|-------------|
| 31-35 | AI Assistant (guided flow, contextual responses, results) | `007-ai-assistant` | 48 | Chat widget on homepage guides visitors through questions, shows matching project results via LLM |
| 35-38 | Matchmaking (structured + AI scoring, ranked matches) | `009-matchmaking` | 32 | Funders see AI-ranked project matches with scores and reasons. Replaces basic keyword recommendations |

**Overlap**: Matchmaking backend starts Day 35 while AI Assistant frontend is finishing.

**Milestone 4 Exit Criteria**:
- AI Assistant widget on homepage with guided question flow
- Contextual LLM responses during assistant conversation
- Assistant results feed into existing search results page
- Matchmaking produces composite-scored (60% structured + 40% AI) rankings
- "Recommended for you" section upgraded with match scores and reasons
- AI fallback to structured-only when LLM unavailable

---

## Milestone 5: Polish & Release (Days 39-43)
**April 23 — April 29, 2026**

Cross-feature integration testing, performance, and release readiness.

| Days | Activity | Deliverable |
|------|----------|-------------|
| 39-40 | End-to-end integration testing across all features | All E2E test suites pass. Cross-feature flows verified (register → onboard → search → bookmark → notify) |
| 40-41 | Performance testing and optimization | All pages load within 2 seconds. Search results within 3 seconds. AI responses within 3 seconds |
| 41-42 | Responsive design audit (mobile, tablet, desktop) | All pages verified at 375px, 768px, and 1440px viewports |
| 42-43 | Bug fixes, final QA pass, deployment preparation | Release candidate ready. QA report with release recommendation |

**Milestone 5 Exit Criteria**:
- All E2E test suites pass
- 80%+ code coverage across all modules
- Performance targets met (2s page load, 3s search, 3s AI)
- Responsive design verified on all breakpoints
- No critical or major bugs outstanding
- Release candidate deployed to staging

---

## Feature Dependency Map

```
M1: Foundation
├── 001-Homepage (search, trending, filters)
└── 002-Registration & Roles (auth, RBAC)
         │
M2: Core Platform
├── 003-Project Management (CRUD, approval, taxonomy)
│   └── depends on: 002-Registration (Org Admin + Super Admin roles)
└── 008-Project Map (location map on detail page)
    └── depends on: 003-Project Management (project detail page)
         │
M3: User Engagement
├── 004-Bookmarks (save, track)
│   └── depends on: 002-Registration (Funder role)
├── 005-Onboarding (preferences, recommendations)
│   └── depends on: 002-Registration
└── 006-Notifications (center, email)
    └── depends on: 004-Bookmarks + 005-Onboarding
         │
M4: Intelligence
├── 007-AI Assistant (guided chat)
│   └── depends on: 001-Homepage (search results)
└── 009-Matchmaking (AI ranking)
    └── depends on: 005-Onboarding (preferences) + 001-Homepage (embeddings)
         │
M5: Polish & Release
└── depends on: all above
```

---

## Risk Factors

| Risk | Impact | Mitigation |
|------|--------|------------|
| AI/embedding service integration takes longer than expected | M1 (search) and M4 (assistant, matchmaking) delayed | Start AI integration Day 1. Fallback to keyword search if embeddings aren't ready |
| Taxonomy seed data not provided by client on time | M2 (project management) blocked | Use placeholder taxonomy data for development. Swap in real data when available |
| Email service configuration delays | M3 (notifications) email features delayed | In-app notifications work independently. Email can be added later without blocking |
| LLM contextual responses quality issues | M4 (AI assistant) quality below expectations | Prompt engineering in parallel. Can ship with simpler template responses as MVP |
| Cross-feature integration issues during M5 | Release delayed | Continuous integration testing throughout — don't defer all testing to M5 |

---

## Summary

| Milestone | Days | Working Days | Features | Tasks |
|-----------|------|-------------|----------|-------|
| M1: Foundation | Mar 2 — Mar 13 | 10 | Homepage, Registration & Roles | 180 |
| M2: Core Platform | Mar 16 — Mar 27 | 10 | Project Management, Project Map | 110 |
| M3: User Engagement | Mar 30 — Apr 10 | 10 | Bookmarks, Onboarding, Notifications | 152 |
| M4: Intelligence | Apr 13 — Apr 22 | 8 | AI Assistant, Matchmaking | 80 |
| M5: Polish & Release | Apr 23 — Apr 29 | 5 | Integration testing, QA, deployment | — |
| **Total** | **Mar 2 — Apr 29** | **43** | **9 features** | **522 tasks** |
