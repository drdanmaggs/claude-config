# Development Philosophy
- Plan before coding - outline approach first
- Ask questions rather than guess
- Simple and boring beats clever - these are real businesses, not tech demos
- Explain trade-offs explicitly - I need to make informed decisions

## My Context
- I typically use: Next.js 15 (App Router), React, Tailwind CSS, TypeScript, Vercel, Supabase
- I prefer practical solutions over "best practices"

## Workflows
1. Ask clarifying questions before implementing
2. Show me the plan and wait for approval
3. Implement in small, testable chunks
4. Run tests after changes
5. Commit frequently with clear messages
6. Keep me in the loop - no silent work for >5 minutes

## Global Documentation
- **Tailwind CSS 4:** `~/.claude/docs/tailwind4guide.md` - Implementation patterns, design tokens, migration
- **Design Token Philosophy:** `~/.claude/docs/design-tokens.md` - Semantic naming, consistency principles, when to break rules
- **Catalyst Components:** `~/.claude/docs/catalyst_master.md` - We use Catalyst Components across all codebases
- **AI Streaming:** `~/.claude/docs/ai-streaming-vercel-sdk.md` - Vercel AI SDK streaming pattern for Next.js + Anthropic
- **MCP Server Configuration Guide:** `~/.claude/docs/MCP-Server-Configuration-Guide.md` - How to configure MCP servers

## Project Documentation
- Check for project-specific CLAUDE.md in root directory
- Check project's /docs/ folder for project-specific guides
- Reference existing patterns before creating new ones

## Git Standards - CRITICAL RULES

### Branch Management
**ALWAYS create a new branch before any code changes:**
1. Check current branch: `git branch --show-current`
2. If on `main` or `master`, STOP
3. Create feature branch: `git checkout -b feature/descriptive-name` or `git checkout -b fix/descriptive-name`
4. Only then proceed with changes

**NEVER commit directly to main/master**

### Branch Naming Convention
- Features: `feature/short-description`
- Fixes: `fix/bug-description`
- Docs: `docs/what-changed`
- Refactors: `refactor/what-improved`

Use lowercase with hyphens, be specific but concise.

### Commits
- Conventional Commits: `type: description`
  - Types: feat, fix, docs, style, refactor, test, chore
- Explain WHY in commit body when not obvious
- Push to feature branches, I'll merge
- **NEVER commit or push without explicit permission**

## Next.js Architecture Rules

### Server vs Client Components - CRITICAL
- **DEFAULT**: All components are Server Components (no 'use client' directive)
- **Root layout (app/layout.tsx) MUST remain Server Component** - NEVER add 'use client' to root layout
- Only mark components as Client Components when they REQUIRE:
  - User interaction (onClick, onChange, onSubmit)
  - React hooks (useState, useEffect, useContext)
  - Browser APIs (localStorage, window, document)
  - Real-time updates

### Pattern: Keep Server, Extract Client
When tempted to add 'use client' to a component:
1. Identify the specific interactive piece that needs it
2. Extract ONLY that piece into a separate Client Component
3. Import that Client Component into your Server Component
4. Keep the parent as a Server Component

Example:
```tsx
// ❌ WRONG - Entire page becomes client
'use client'
export default function Page() {
  const [count, setCount] = useState(0)
  return <div>Static content... <button onClick={() => setCount(count + 1)}>{count}</button></div>
}

// ✅ RIGHT - Extract interactive part
// page.tsx - Server Component
import Counter from './Counter'
export default function Page() {
  return <div>Static content... <Counter /></div>
}

// Counter.tsx - Client Component (separate file)
'use client'
export default function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

## Code Standards
- TypeScript strict mode
- Functional components with hooks (React)
- Tailwind for styling (no CSS modules)
- Keep components under 200 lines
- Colocate tests with source files
- **Use design tokens/semantic classes over hardcoded values**
- **Never use opacity modifiers in components** (e.g., bg-black/20) - define proper tokens instead

## Catalyst UI Component Organization

### Philosophy
- Catalyst components are **source code you own**, not a library dependency
- Copy components into `/components/` and customize them directly
- They're a starting point that evolves into your own component system
- Don't isolate Catalyst in `/ui/` or `/catalyst/` subdirectories

### File Structure (Next.js App Router)
```
/components/
  ├── button.tsx           ← Catalyst base (customize as needed)
  ├── badge.tsx            ← Catalyst base
  ├── input.tsx            ← Catalyst base
  ├── heading.tsx          ← Catalyst base
  └── app-navbar.tsx       ← Custom component (uses Catalyst)
```

### Import Convention
- Use `@/components/button`, `@/components/badge` etc.
- No subdirectory prefixes (e.g., NOT `@/components/ui/button`)

### Custom Components
For app-specific components as projects grow:
- **Simple custom components**: Keep in `/components/` alongside Catalyst
- **Complex features**: Consider `/components/features/` subdirectory
- **Layout compositions**: Consider `/components/layouts/` subdirectory
- **Start simple**: Don't organize prematurely - refactor when `/components/` feels cluttered

### Usage Pattern
```tsx
// Import Catalyst components directly
import { Button } from '@/components/button'
import { Heading } from '@/components/heading'

// Customize Catalyst components by editing their source
// Open components/button.tsx and modify styling/behavior
```

## Project Management
- Break large tasks into 5-10 smaller tasks
- Challenge my perfectionism - aim for 80% not 100%
- Call me out if I'm seeking perfection as procrastination

## Never
- Read node_modules/, dist/, .git/, .next/
- Access .env* or secrets/
- Use sudo or rm -rf
- Make assumptions about medical/health content (I review all of that)
- Create PRs without my explicit approval
- Commit or push without explicit permission
- Commit directly to main/master branches
- **Add 'use client' to app/layout.tsx or other layout files**
- **Use 'use client' as first solution - explore server-first options first**
- **Fetch data in Client Components when Server Components could handle it**
- **Put API keys/secrets in Client Components** (they're exposed to browser)

## Always
- Create a new branch before making code changes
- Show me diffs before committing
- Run tests before marking tasks complete
- Reference my other projects when relevant patterns exist
- Ask before making assumptions about project requirements
- **Prefer Server Components by default**
- **Keep data fetching in Server Components when possible**
- **Extract minimal interactive pieces into Client Components**
