# Task: {TASK_DESCRIPTION}

## Metadata
- **Task ID**: {TASK_ID}
- **Branch**: {BRANCH_NAME}
- **Worktree Path**: {WORKTREE_PATH}
- **Current Iteration**: 1
- **Max Iterations**: 5
- **Started**: {START_TIMESTAMP}
- **Last Updated**: {LAST_UPDATED_TIMESTAMP}

## Goal

{DETAILED_GOAL_DESCRIPTION}

## Requirements

### Functional Requirements
- {REQUIREMENT_1}
- {REQUIREMENT_2}

### Non-Functional Requirements
- {NFR_1}
- {NFR_2}

### Constraints
- {CONSTRAINT_1}

## Iteration History

### Iteration 1 (In Progress)

**Phase**: Requirements Gathering

**What was done**:
- Task initialized
- Awaiting requirements clarification

**Commits**:
- (None yet)

**Review Results**:
- (Not reviewed yet)

**Next Actions**:
- [ ] Clarify requirements with user
- [ ] Explore codebase for similar patterns
- [ ] Design architecture approach
- [ ] Implement initial version

**Notes**:
- (Add notes here for next iteration)

## Current Status

- **Phase**: Iteration 1 - Requirements Gathering
- **Blocked**: No
- **Ready for Review**: No
- **Awaiting User Input**: Yes

## Decision Log

(Record important decisions made during implementation)

**Format**:
```
[Iteration N] Decision Topic: Decision made (Rationale)
```

**Examples**:
- [Iteration 1] Authentication Method: JWT tokens (Stateless API requirement)
- [Iteration 2] Database: PostgreSQL over MongoDB (Relational data model)

## Related Files

(List of files created or modified during this task)

- (None yet)

## Action Items

(Current todo list for the next iteration)

- [ ] Item 1
- [ ] Item 2

## Notes for Next Iteration

(Free-form notes that will help the next iteration understand context)

- Remember to...
- Be careful about...
- Consider...

---

## Instructions for AI Agents

### How to Read This File

1. **Start with Metadata**: Check current iteration number and max iterations
2. **Read Goal and Requirements**: Understand what needs to be accomplished
3. **Review Iteration History**: See what's been done and what issues were found
4. **Check Current Status**: Understand the current phase
5. **Read Decision Log**: Understand why certain choices were made
6. **Review Action Items**: Know what to do next
7. **Read Notes**: Get context-specific guidance

### How to Update This File

1. **After Each Iteration**:
   - Increment "Current Iteration" in Metadata
   - Update "Last Updated" timestamp
   - Add new section to Iteration History
   - Update Current Status
   - Record any decisions in Decision Log
   - Update Related Files list
   - Clear completed Action Items
   - Add new Action Items for next iteration
   - Update Notes for Next Iteration

2. **Iteration History Format**:
```markdown
### Iteration N (Completed|In Progress|Failed)

**Phase**: Requirements|Exploration|Design|Implementation|Review|Fixing

**What was done**:
- Bullet point 1
- Bullet point 2

**Commits**:
- `hash` commit message

**Review Results**:
- **Critical** (N): issue description (file:line)
- **High** (N): issue description (file:line)
- **Medium** (N): issue description (file:line)
- **Low** (N): issue description (file:line)

**Next Actions**:
- [ ] Specific action 1
- [ ] Specific action 2

**Notes**:
- Important context for next iteration
```

3. **Be Specific**:
   - Include file names and line numbers
   - Reference commit hashes
   - Describe concrete actions, not vague statements
   - Use checkboxes for actionable items

4. **Keep it Concise**:
   - Focus on actionable information
   - Avoid redundant explanations
   - Link to code/docs instead of copying

### Example of a Complete Update

```markdown
### Iteration 2 (Completed)

**Phase**: Fixing Critical Issues

**What was done**:
- Fixed all 3 critical issues from Iteration 1 review
- Added error handling in auth.ts:45
- Strengthened token validation in auth.ts:67
- Applied prepared statements in auth.ts:89

**Commits**:
- `a1b2c3d` fix: Add comprehensive error handling to auth module
- `b2c3d4e` security: Strengthen JWT token validation
- `c3d4e5f` security: Prevent SQL injection in user queries

**Review Results**:
- **Critical** (0): ✅ All resolved
- **High** (2):
  - Test coverage insufficient for edge cases (tests/auth.test.ts)
  - Type definitions incomplete for AuthResponse (types/auth.ts:12)
- **Medium** (5): Various style and documentation improvements

**Next Actions**:
- [ ] Add test cases for edge cases: empty email, invalid tokens, expired tokens
- [ ] Complete AuthResponse type definition with all possible fields
- [ ] Review medium priority issues and decide which to address

**Notes**:
- All critical security issues now resolved
- High priority items are non-blocking but important for quality
- Consider addressing medium items in Iteration 4 if time permits
```

### Success Criteria

This file is well-maintained when:
- ✅ Any agent can read it and understand the full context
- ✅ The next iteration knows exactly what to do
- ✅ All decisions are documented with rationale
- ✅ Progress is clearly tracked
- ✅ Issues are specific with file:line references
- ✅ No redundant or outdated information
