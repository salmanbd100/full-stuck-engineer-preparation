# Written Communication

Master professional writing for technical documentation, emails, pull requests, and design documents.

## 📚 Overview

Written communication is critical for:
- **Documentation:** READMEs, technical guides, API docs
- **Code reviews:** PR descriptions, review comments
- **Email:** Professional correspondence
- **Slack/Chat:** Team communication
- **Design docs:** RFCs, architecture proposals
- **Reports:** Project updates, incident postmortems

Strong written communication scales your impact beyond face-to-face interactions.

## 📖 Technical Documentation

### 1. README Files

**Essential Sections:**
```markdown
# Project Name

Brief one-sentence description of what the project does.

## Overview
2-3 paragraphs explaining the project in detail, use cases, and why it exists.

## Features
- Feature 1: Description
- Feature 2: Description
- Feature 3: Description

## Installation
\`\`\`bash
npm install package-name
\`\`\`

## Usage
\`\`\`javascript
import { Component } from 'package-name';

const example = new Component();
example.doSomething();
\`\`\`

## API Reference
### method1(param)
- **param** (type): Description
- **Returns:** Description

### method2()
...

## Contributing
Instructions for contributing to the project

## License
MIT License
```

**Best Practices:**
✅ Start with what, not how (high-level first)
✅ Include code examples
✅ Keep it updated (outdated docs worse than no docs)
✅ Use simple language (assume beginner knowledge)
✅ Include screenshots/diagrams for UI components

---

### 2. API Documentation

**Structure:**
```markdown
## POST /api/v1/users

Create a new user account.

### Request

**Headers:**
- `Authorization: Bearer <token>`
- `Content-Type: application/json`

**Body:**
\`\`\`json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "name": "John Doe"
}
\`\`\`

### Response

**Success (201 Created):**
\`\`\`json
{
  "id": "user_123",
  "email": "user@example.com",
  "name": "John Doe",
  "createdAt": "2024-12-07T10:30:00Z"
}
\`\`\`

**Error (400 Bad Request):**
\`\`\`json
{
  "error": {
    "code": "INVALID_EMAIL",
    "message": "Email format is invalid"
  }
}
\`\`\`

### Example

\`\`\`bash
curl -X POST https://api.example.com/v1/users \\
  -H "Authorization: Bearer token123" \\
  -H "Content-Type: application/json" \\
  -d '{"email":"user@example.com","password":"pass","name":"John"}'
\`\`\`
```

---

### 3. Code Comments

**When to Comment:**
```javascript
// ❌ BAD: Obvious comment
// Increment counter by 1
counter++;

// ✅ GOOD: Explain WHY
// We increment by 2 to skip odd indices because they contain
// metadata rather than actual data
counter += 2;

// ❌ BAD: Redundant
// Get user by ID
const user = getUserById(id);

// ✅ GOOD: Explain non-obvious behavior
// getUserById returns null for soft-deleted users, which is intentional
// to prevent accidental data exposure
const user = getUserById(id);
```

**Complex Logic Comments:**
```javascript
// ✅ GOOD: Explain algorithm choice
// Using binary search instead of linear search because array is
// guaranteed sorted by timestamp. This reduces lookup from O(n) to O(log n).
// Trade-off: We must maintain sorted order on insert (small cost).
function findEventByTime(events, targetTime) {
  let left = 0;
  let right = events.length - 1;
  // ... binary search implementation
}
```

**TODOs and FIXMEs:**
```javascript
// TODO(salman): Refactor this to use the new API client once migration is complete
// Tracked in JIRA-1234

// FIXME: This has a race condition when multiple users update simultaneously
// Need to implement optimistic locking. See discussion in PR #567

// HACK: Temporary workaround for Safari bug. Remove once Safari 17 is released.
```

---

## 💻 Pull Request Descriptions

### PR Template

```markdown
## Summary
[One-line description of what this PR does]

Closes #[issue number]

## Changes
- Changed X to Y to improve Z
- Added new component W for feature V
- Refactored U for better performance

## Motivation
[Why are we making this change? What problem does it solve?]

## Technical Details
### Approach
[How did you implement this? Key decisions made?]

### Trade-offs
- Chose approach A over B because...
- This adds 10KB to bundle size, which is acceptable because...

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing completed
- [ ] Tested on Chrome, Firefox, Safari

### Test Plan
1. Go to dashboard
2. Click on new button
3. Verify modal opens
4. Submit form with valid data
5. Confirm success message appears

## Screenshots
[Before/After screenshots if UI change]

## Performance Impact
- Bundle size: +10KB (acceptable for new feature)
- Load time: No change (1.8s)
- Lighthouse score: 92 → 91 (minimal impact)

## Deployment Notes
- Requires database migration (included in PR)
- Feature flag: `enable_new_dashboard`
- Backward compatible: Yes

## Questions for Reviewers
1. Is the abstraction level appropriate?
2. Should we add more test coverage for edge case X?
3. Thoughts on the naming of component Y?

---
**Reviewers:** @john @sarah
**Related PRs:** #123, #456
```

### PR Best Practices

✅ **DO:**
- Write clear, descriptive title
- Explain WHY, not just WHAT
- Include screenshots for UI changes
- List testing steps
- Keep PRs focused (< 400 lines if possible)
- Reference related issues/PRs

❌ **DON'T:**
- Write "misc fixes" or "updates"
- Submit giant PRs (1000+ lines)
- Forget to update tests
- Skip the description
- Use vague language

---

## 💬 Code Review Comments

### Giving Feedback

**Structure:**
```markdown
**Severity:** [Blocking / Non-blocking / Nit]
**Issue:** [What's wrong]
**Suggestion:** [How to fix it]
**Rationale:** [Why it matters]
```

**Examples:**

```markdown
**Blocking:** This has a potential SQL injection vulnerability

The user input is directly interpolated into the SQL query without
sanitization. This could allow malicious users to execute arbitrary SQL.

Suggestion: Use parameterized queries instead:
\`\`\`javascript
db.query('SELECT * FROM users WHERE id = ?', [userId]);
\`\`\`

References:
- OWASP SQL Injection: [link]
- Our security guidelines: [link]
```

```markdown
**Non-blocking:** Consider using useMemo here for performance

This calculation runs on every render, which could be expensive for
large datasets (n > 1000 items).

Suggestion:
\`\`\`javascript
const filtered = useMemo(
  () => items.filter(item => item.active),
  [items]
);
\`\`\`

This would only recalculate when items change. Not blocking since
performance is acceptable currently, but good optimization for the future.
```

```markdown
**Nit:** Typo in comment

Line 45: "recieve" should be "receive"
```

### Receiving Feedback

**Responding to Comments:**
```markdown
✅ GOOD:
"Great catch! I've updated the code to use parameterized queries.
Thanks for the security tip!"

"Good point about useMemo. I've added it and measured a 30% reduction
in render time for large lists."

"I considered that approach, but chose this one because [reasoning].
What do you think about [alternative]?"

❌ BAD:
"This works fine, no need to change."
"I don't think this is a problem."
"Whatever, I'll fix it."
```

**When You Disagree:**
```markdown
"I appreciate the feedback! I have a different perspective on this.

My reasoning for the current approach:
1. [Reason 1]
2. [Reason 2]

The alternative you suggested would [trade-off]. For our use case, I
think the current approach is better because [justification].

Would you like to discuss this synchronously? Happy to jump on a quick
call if you think that would help clarify."
```

---

## 📧 Professional Emails

### Email Structure

**Subject Line:**
```
❌ BAD:
- "Question"
- "Help needed"
- "Interview"

✅ GOOD:
- "Question about Senior Frontend Role - Timeline"
- "Follow-up: Interview with Sarah on Dec 7"
- "Thank you: Frontend Engineer Interview"
```

**Body Template:**
```
[Greeting]
Hi [Name],

[Context - if needed]
I hope this email finds you well. I wanted to follow up on...

[Main content - keep concise]
[Paragraph 1: Purpose of email]
[Paragraph 2: Additional details if needed]

[Call to action]
Could you please...?
Would it be possible to...?

[Closing]
Thank you for your time.

Best regards,
[Your name]
```

### Common Email Scenarios

**1. Thank You After Interview:**
```
Subject: Thank you - Senior Frontend Engineer Interview

Hi Sarah,

Thank you for taking the time to interview me today for the Senior
Frontend Engineer position. I enjoyed learning about the team's work on
the React platform and the technical challenges you're tackling.

I'm particularly excited about the opportunity to contribute to the
performance optimization initiatives we discussed. The problems you're
solving align well with my experience improving load times by 60% at my
current company.

Please let me know if you need any additional information. I look
forward to hearing about next steps.

Best regards,
Salman Rahman
```

**2. Following Up:**
```
Subject: Following up: Senior Frontend Role Application

Hi Sarah,

I hope this email finds you well. I wanted to follow up on my interview
for the Senior Frontend Engineer position on December 1st.

I remain very enthusiastic about the opportunity to join your team. Is
there any update on the timeline for next steps in the process?

Please let me know if you need any additional information from my side.

Thank you for your time.

Best regards,
Salman Rahman
```

**3. Asking Questions:**
```
Subject: Questions about Senior Frontend Role

Hi Sarah,

Thank you for the interview invitation! I have a few questions before we
schedule:

1. Will this be a technical or behavioral interview?
2. What technologies should I prepare to discuss?
3. How long should I expect the interview to last?

I'm excited about the opportunity and want to ensure I'm well-prepared.

Looking forward to your response.

Best regards,
Salman Rahman
```

---

## 💬 Slack/Chat Communication

### Best Practices

**Message Structure:**
```
❌ BAD:
"hey"
[waits for response]
"are you there?"
[waits]
"I have a question"
[waits]
"about the PR"

✅ GOOD:
"Hi @john! Quick question about PR #123 - should the error handling
return null or throw an exception? I'm leaning toward throwing since
it's an unexpected state, but wanted your input before finalizing."
```

**Threading:**
```
✅ Use threads for:
- Follow-up discussion
- Keeping channel clean
- Multi-person conversations

❌ Don't use threads for:
- Important announcements
- Questions to entire channel
- Time-sensitive issues
```

**Asking for Help:**
```
❌ BAD:
"My code doesn't work. Help!"

✅ GOOD:
"I'm debugging an issue with the user authentication flow. Getting
'token expired' errors even with fresh tokens.

What I've tried:
- Verified token generation timestamp
- Checked server time sync
- Reviewed token validation logic

Error occurs in UserService.ts line 45. I've added console logs showing
the token is valid but still fails.

Has anyone seen similar issues? Code: [link to specific lines]"
```

**Sharing Wins:**
```
"🎉 Shipped the new dashboard to production! Thanks to @sarah for the
code review and @john for testing. Load time improved from 4s to 1.6s.

Metrics:
- 60% faster load time
- 30% reduction in bounce rate
- 100% of tests passing

Rollout: 10% of users now, 100% by end of week."
```

---

## 📄 Design Documents (RFCs)

### RFC Template

```markdown
# RFC: [Feature Name]

**Author:** [Your Name]
**Date:** [Date]
**Status:** [Draft / In Review / Approved / Implemented]

## Summary
[2-3 sentence high-level description]

## Motivation
### Problem
[What problem are we solving?]

### User Impact
[How does this benefit users?]

### Business Impact
[How does this benefit the business?]

## Proposal
### High-Level Design
[Architecture diagram + explanation]

### API Changes
[New APIs, breaking changes, etc.]

### Data Model
[Database schema changes if applicable]

### Implementation Plan
Phase 1: [Description] (2 weeks)
Phase 2: [Description] (3 weeks)
Phase 3: [Description] (1 week)

## Alternatives Considered
### Alternative 1: [Name]
**Pros:**
- Pro 1
- Pro 2

**Cons:**
- Con 1
- Con 2

**Decision:** Rejected because...

### Alternative 2: [Name]
...

## Trade-offs
- **Performance vs Simplicity:** Choosing simplicity initially, can
  optimize later if needed
- **Time to Market vs Completeness:** Shipping MVP in Phase 1, adding
  advanced features in Phase 2

## Risks & Mitigations
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|---------|------------|
| Database migration fails | Low | High | Extensive testing, rollback plan |
| Third-party API downtime | Medium | Medium | Implement retry logic, fallback |

## Success Metrics
- Load time < 2 seconds (currently 4s)
- 95% adoption within 3 months
- < 0.1% error rate

## Timeline
- Week 1-2: Design review and approval
- Week 3-4: Implementation Phase 1
- Week 5-6: Testing and refinement
- Week 7: Production rollout (gradual)

## Open Questions
1. Should we support IE11? (affects implementation complexity)
2. What's the migration strategy for existing users?
3. Do we need feature flags for gradual rollout?

## References
- [Link to related PRs]
- [Link to user research]
- [Link to competitive analysis]

---
**Reviewers:** @tech-lead @architect @product-manager
**Discussion:** [Link to Slack thread or doc comments]
```

---

## ✅ Writing Checklist

**Before Sending:**
- [ ] Clear purpose/goal stated upfront
- [ ] Spell-checked (Grammarly or similar)
- [ ] Grammar correct (verb tenses, subject-verb agreement)
- [ ] Concise (removed unnecessary words)
- [ ] Structured (headings, bullets, short paragraphs)
- [ ] Actionable (clear next steps if applicable)
- [ ] Professional tone
- [ ] Links work (if any)
- [ ] Code formatted (if applicable)

---

**Related:** [Technical Communication](./01-technical-communication.md) | [Cross-Cultural Communication](./06-cross-cultural-communication.md) | [Behavioral Interview](./02-behavioral-interview.md)

---

**Write clearly, concisely, professionally.** ✍️
