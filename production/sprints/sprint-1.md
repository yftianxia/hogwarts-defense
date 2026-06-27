# Sprint 1 -- 2026-06-25 to 2026-06-26

## Sprint Goal

Polish the completed Level 1 prototype, establish the production pipeline
(milestones, sprints, QA), and create design documentation so the project
is ready to scale to additional levels.

## Capacity

- Total days: 2
- Buffer (20%): 0.4 days reserved for unplanned work
- Available: 1.6 days

## Tasks

### Must Have (Critical Path)

| ID | Task | Agent/Owner | Est. Days | Dependencies | Acceptance Criteria |
|----|------|-------------|-----------|-------------|---------------------|
| STORY-001 | Complete Level 1 GDD (8 sections) | game-designer | 0.5 | game-concept.md | All 8 required sections filled; design-review passes |
| STORY-002 | Create Milestone M1 definition | producer | 0.25 | STORY-001 | M1 doc written with all deliverables and status |
| STORY-003 | Write automated tests for combat systems | technical-director | 0.5 | src/index.html | Unit tests for damage formulas, spell cooldowns, wave spawning, HP/win/loss; all pass |
| STORY-004 | Balance pass on Level 1 | game-designer | 0.25 | STORY-001 | HP/damage/cooldown values reviewed and documented with rationale in GDD |
| STORY-005 | QA smoke check on Level 1 | qa-lead | 0.25 | STORY-003, STORY-004 | Smoke check doc written; all S1 bugs documented; no regressions from 06-24 state |

### Should Have

| ID | Task | Agent/Owner | Est. Days | Dependencies | Acceptance Criteria |
|----|------|-------------|-----------|-------------|---------------------|
| STORY-006 | Set up risk register | producer | 0.25 | None | Risk register file created with initial risks for M2 scope |

### Nice to Have

| ID | Task | Agent/Owner | Est. Days | Dependencies | Acceptance Criteria |
|----|------|-------------|-----------|-------------|---------------------|
| STORY-007 | Draft Level 2 concept in game-concept.md | game-designer | 0.25 | STORY-001 | New character, new map, new wave config sketched |

## Carryover from Previous Sprint

None -- this is the first sprint.

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|------------|
| Test framework not set up | Medium | High | STORY-003 starts by scaffolding minimal test harness (plain JS assertions in an HTML test page) |
| Balance changes break existing gameplay | Low | Medium | Smoke check (STORY-005) runs after balance pass to catch regressions |
| Single developer bottleneck | High | Low | Stories are small (0.25-0.5 days) and can be done sequentially by one person |

## Dependencies on External Factors

- None -- all work is self-contained within the repo.

## Definition of Done for this Sprint

- [ ] All Must Have tasks completed
- [ ] All tasks pass acceptance criteria
- [ ] QA plan exists (`production/qa/qa-plan-sprint-1.md`)
- [ ] All Logic/Integration stories have passing unit/integration tests
- [ ] Smoke check passed (`/smoke-check sprint`)
- [ ] QA sign-off report: APPROVED or APPROVED WITH CONDITIONS (`/team-qa sprint`)
- [ ] No S1 or S2 bugs in delivered features
- [ ] Design documents updated for any deviations
- [ ] Code reviewed and merged

> **QA Plan**: Not yet created. Run `/qa-plan sprint` before starting implementation.
> The Production to Polish gate requires a QA sign-off report, which requires a QA plan.
