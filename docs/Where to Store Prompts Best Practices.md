# Where to Store Prompts: Best Practices

## The Core Question

Should prompts live in:
1. Code files?
2. Config files?
3. Database?
4. External service?

**Answer: It depends on your stage and needs.**

---

## Pattern 1: In-Code (What You Have Now)

### Structure
```
/app/api/clarify/route.ts
  - System prompt defined as string literal inside the route
  
/app/api/next-task/route.ts
  - Prompt built dynamically inside the route
```

### When This Works
- MVP/prototype phase
- Single developer
- Prompts change infrequently
- No A/B testing needed

### Limitations
- Hard to find prompts (scattered across codebase)
- Easy to accidentally break when editing
- Can't iterate on prompts without code changes
- No version history for prompts specifically
- Hard to compare different versions

---

## Pattern 2: Prompts Directory (Recommended for You Now)

### Structure
```
/prompts/
  /agents/
    clarification-agent.ts
    recommendation-agent.ts
    coordinator-agent.ts
    weekly-review-agent.ts
  /system/
    gtd-horizons.md
    max-driver-personality.md
  /templates/
    context-builder.ts
    user-profile-formatter.ts
  /examples/
    good-project-titles.md
```

### Implementation

**Simple version:**
```typescript
// /prompts/agents/clarification-agent.ts
export const CLARIFICATION_SYSTEM_PROMPT = `
You are a productivity coach helping Dan clarify vague inbox items...

CURRENT ITEM TO CLARIFY:
{{currentItem}}

YOUR GOAL:
1. Ask questions to understand what Dan is actually trying to achieve
...
`;

export function buildClarificationPrompt(currentItem: string) {
  return CLARIFICATION_SYSTEM_PROMPT.replace('{{currentItem}}', currentItem);
}
```

**Used in API route:**
```typescript
// /app/api/clarify/route.ts
import { buildClarificationPrompt } from '@/prompts/agents/clarification-agent';

export async function POST(req: Request) {
  const { messages, currentItem } = await req.json();
  
  const systemPrompt = buildClarificationPrompt(currentItem.description);
  
  const result = streamText({
    model: anthropic('claude-3-5-sonnet-20241022'),
    system: systemPrompt,
    messages: messages,
  });
  
  return result.toTextStreamResponse();
}
```

### When This Works
- Small to medium apps
- 1-5 developers
- Want easy iteration on prompts
- Need version control for prompts
- Want separation of concerns

### Benefits
- ✅ Easy to find all prompts
- ✅ Can edit without touching API code
- ✅ Clear git diffs for prompt changes
- ✅ Can version prompts alongside code
- ✅ Reusable across multiple routes
- ✅ Can share prompts between agents

---

## Pattern 3: Config File

### Structure
```
/config/prompts.ts  or  /config/prompts.json
```

### Implementation
```typescript
// /config/prompts.ts
export const PROMPTS = {
  clarification: {
    system: `You are a productivity coach...`,
    version: '2.1',
    temperature: 0.7,
  },
  recommendation: {
    system: `Based on this context, suggest...`,
    version: '1.5',
    temperature: 0.5,
  },
};
```

### When This Works
- Many small, simple prompts
- Want centralized configuration
- All prompts follow similar pattern
- Don't need complex prompt building logic

### Limitations
- Gets messy with large prompts
- Hard to maintain if prompts have complex logic
- JSON doesn't support multi-line strings well
- Can't easily add helper functions

---

## Pattern 4: Database Storage

### Structure
```sql
CREATE TABLE prompts (
  id UUID PRIMARY KEY,
  name VARCHAR(100),
  version VARCHAR(20),
  content TEXT,
  metadata JSONB,
  created_at TIMESTAMP,
  is_active BOOLEAN
);
```

### When This Works
- Production app with multiple environments
- Want to update prompts without deploying code
- Need A/B testing
- Want prompt versioning in production
- Multiple team members editing prompts
- Want analytics on which prompt versions perform best

### Implementation
```typescript
// Fetch prompt from database
const prompt = await db.prompts.findFirst({
  where: { 
    name: 'clarification-agent',
    is_active: true 
  }
});

const result = streamText({
  model: anthropic('claude-3-5-sonnet-20241022'),
  system: prompt.content,
  messages: messages,
});
```

### Benefits
- ✅ Update without deploying
- ✅ Easy A/B testing
- ✅ Version history in database
- ✅ Can rollback instantly
- ✅ Analytics on prompt performance
- ✅ Non-developers can edit prompts

### Considerations
- ⚠️ More complexity
- ⚠️ Need database migration strategy
- ⚠️ Need UI for managing prompts
- ⚠️ Harder to test locally
- ⚠️ Version control less obvious

---

## Pattern 5: External CMS/Service

### Examples
- Contentful
- Sanity
- Custom prompt management service
- LangSmith (LangChain's prompt management)

### When This Works
- Large team
- Non-technical people managing prompts
- Need sophisticated versioning
- Want prompt analytics
- Multiple applications sharing prompts

### Considerations
- ⚠️ Additional dependency
- ⚠️ Cost
- ⚠️ Complexity
- ⚠️ Vendor lock-in potential

---

## Versioning Strategies

### Strategy 1: File-Based Versioning
```
/prompts/agents/clarification/
  v1.ts
  v2.ts
  v3-experimental.ts
  current.ts  (symlink or export)
```

### Strategy 2: Git-Based Versioning
```typescript
// /prompts/agents/clarification-agent.ts
// Version: 2.1
// Last updated: 2025-01-15
// Changes: Added examples for imperative mood

export const CLARIFICATION_PROMPT = `...`;
```

Track changes through git history.

### Strategy 3: Database Versioning
```sql
SELECT * FROM prompts 
WHERE name = 'clarification-agent' 
ORDER BY created_at DESC;
```

All versions stored, can activate any version.

---

## Your Specific Recommendation

**Current state:** Prompts in API routes (Pattern 1)

**Next step:** Move to Prompts Directory (Pattern 2)

**Why:**
- You're solo developer in MVP phase
- Need to iterate quickly on prompts
- Want version control through git
- Don't need production prompt updates yet
- Keeps things simple but organized

**Future (when productizing):**
- Consider database storage (Pattern 4)
- When you have users and want to A/B test
- When you want to update prompts without deploying

---

## Migration Path

### Step 1: Create Structure (Now)
```
/prompts/
  /agents/
  /system/
  /templates/
```

### Step 2: Extract Prompts (Now)
Move existing prompts from API routes to dedicated files.

### Step 3: Add Versioning (Soon)
When you want to experiment with variants.

### Step 4: Add Testing (When Needed)
Create test suite for prompts with example inputs/outputs.

### Step 5: Database Storage (Later)
When you need production updates and A/B testing.

---

## Decision Matrix

| Need | Pattern 1 (In-Code) | Pattern 2 (Directory) | Pattern 4 (Database) |
|------|---------------------|----------------------|---------------------|
| Easy to find prompts | ❌ | ✅ | ✅ |
| Easy to edit | ⚠️ | ✅ | ✅ |
| Version control | ⚠️ | ✅ | ✅ |
| Update without deploy | ❌ | ❌ | ✅ |
| A/B testing | ❌ | ⚠️ | ✅ |
| Simple to implement | ✅ | ✅ | ❌ |
| Works offline | ✅ | ✅ | ❌ |
| Non-dev editing | ❌ | ❌ | ✅ |

---

## Best Practice Checklist

✅ **Separate prompts from business logic**
✅ **Use consistent directory structure**
✅ **Version your prompts**
✅ **Document what each prompt does**
✅ **Include examples in prompts**
✅ **Make prompts reusable**
✅ **Test prompts with real data**
✅ **Track prompt performance**
✅ **Plan for iteration**

---

## Common Anti-Patterns to Avoid

❌ **String literals scattered across codebase**
- Hard to find, easy to duplicate, painful to update

❌ **No versioning strategy**
- Can't track what changed or roll back

❌ **Prompts in database too early**
- Premature optimization, adds complexity

❌ **One giant prompt file**
- Hard to maintain, merge conflicts

❌ **No separation between system/user content**
- Prone to prompt injection

❌ **Hardcoded user context in prompts**
- Makes prompts not reusable