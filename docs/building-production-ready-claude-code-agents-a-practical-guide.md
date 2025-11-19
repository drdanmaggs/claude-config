# Building Production-Ready Claude Code Agents: A Practical Guide

**Claude Code agents transform development workflows when designed correctly.** The most effective agents follow the single-responsibility principle, maintain minimal tool access, and integrate into systematic workflows rather than replacing human judgment. Teams managing billions of tokens monthly reveal that success depends on treating agents as infrastructure—with proper testing, observability, and iterative refinement—rather than as autonomous employees. This guide distills battle-tested practices from enterprise deployments, official Anthropic guidance, and power users managing production systems.

Understanding agent architecture fundamentally changes how you approach development. The key insight: **agents excel at focused expertise with clear boundaries, not general-purpose autonomy**. When you create an agent, you're building a specialized tool that operates in isolated context, communicates through structured outputs, and benefits from progressive disclosure of information. The best implementations combine multiple focused agents through dynamic orchestration rather than building monolithic, all-knowing systems.

This philosophy emerges from production experience across companies like TELUS (100+ billion tokens monthly), Zapier (800+ internal agents), and Bridgewater (50-70% reduction in analysis time). These organizations discovered that agent effectiveness depends more on proper design patterns, context management, and systematic workflows than on raw model capability. Even with powerful models, poorly designed agents create overhead rather than value.

## The foundation: Understanding what agents actually are

Claude Code agents are specialized AI assistants defined through markdown files with YAML frontmatter. Each agent operates in its own isolated context window, inheriting or restricting tool access, and following custom instructions encoded in its system prompt. This architecture enables powerful capabilities: parallel execution, context isolation, specialized expertise, and systematic verification workflows.

**Agents differ fundamentally from projects, prompts, and MCP servers.** Prompts provide one-time instructions in your current conversation. Projects share knowledge and history across conversations (Team/Enterprise feature). MCP servers create secure gateways to external systems with authentication boundaries. Agents provide recurring workflows with isolated context and defined tool access. Each serves different purposes—understanding when to use which mechanism separates effective implementations from confused ones.

The technical structure follows a consistent pattern across all implementations. Every agent consists of YAML frontmatter specifying metadata (name, description, tools, model, permissions) followed by markdown containing the system prompt. Files live in `.claude/agents/` for project-specific agents or `~/.claude/agents/` for user-level agents. Project-level agents take precedence when names conflict, enabling team-wide consistency while allowing personal customization.

**Context isolation provides agents' most powerful capability.** When you invoke a subagent, it operates in a separate 200,000 token context window. Only the final summary returns to the main conversation, preventing detail pollution while preserving high-level focus. This architectural choice enables longer work sessions, cleaner conversations, and true parallel execution. Teams report this alone justifies agent adoption—maintaining strategic context while delegating tactical implementation.

Model selection significantly impacts both performance and costs. Claude offers three primary models: **Haiku 4.5 delivers 90% of Sonnet performance at 3x cost savings** ($1/$5 vs $3/$15 per million tokens), making it ideal for frequent subagents. Sonnet 4.5 balances reasoning and cost for most development tasks. Opus 4.1 provides deepest reasoning for critical architecture and security work but costs 5x more. Strategic model assignment—Opus for planning, Sonnet for implementation, Haiku for validation—extends subscription budgets significantly while maintaining quality.

## When to create agents versus using other approaches

**The decision tree for agent creation follows clear patterns based on task characteristics.** Start by asking whether this task requires one-time instructions with immediate context—if yes, use inline prompts in your main conversation. For knowledge that multiple agents or conversations need, create a Skill using Agent Skills progressive disclosure. Skills provide portable expertise (security review procedures, data analysis patterns) that loads dynamically when relevant rather than polluting every agent's context.

Create custom agents when you have specific recurring workflows requiring tool restrictions or isolated context. The canonical use cases: **code reviewers that only read files, specialized implementers with framework-specific knowledge, orchestrators that coordinate other agents, validators that enforce quality gates**. Each benefits from context isolation—reviewers don't clutter main conversation with detailed analysis, implementers work in parallel on different features, validators provide independent verification without bias from implementation context.

MCP servers serve a different purpose entirely: secure gateways to external systems requiring authentication, networking, or complex state management. When you need GitHub PR management, database queries, or cloud infrastructure control, build or configure MCP servers. When you need consistent coding patterns or specialized review processes, build agents. The distinction: **MCP servers provide capabilities, agents provide workflows**.

Enterprise experience reveals that fewer, well-designed agents outperform agent proliferation. Teams with 5-10 carefully crafted agents report better outcomes than teams with 30+ narrowly defined specialists. The overhead of agent management—maintaining prompts, debugging failures, updating documentation—scales linearly with agent count. Each agent should provide clear, distinct value that justifies its maintenance burden.

**The "Agent Skills first" principle prevents redundant agent creation.** Before building your fourth security-focused agent, create a Security Analysis Skill that all agents can load when needed. Skills cost less to maintain (one file vs multiple agent configs), improve consistency (same security checks everywhere), and enable progressive disclosure (only loads when relevant). This pattern scales better than specialized agents for cross-cutting concerns.

Consider whether dynamic orchestration solves your problem better than pre-defined agents. The main Claude agent can spawn Task() subagents dynamically—general-purpose clones that inherit context and work in parallel. For one-off complex tasks, this often outperforms rigid subagent workflows. Reserve custom agents for truly recurring patterns where specific tool restrictions or specialized prompts add consistent value.

## Architecture patterns that scale in production

**Agent architecture follows three primary patterns, each solving different problems.** Simple focused agents handle single responsibilities with minimal complexity: a code reviewer that only reads files and reports issues, a test writer that generates tests following project patterns, a documentation specialist that maintains API specs. These typically run 50-200 lines, use 3-5 tools, and complete work quickly. Their value comes from consistency and context isolation, not sophistication.

Orchestrator agents coordinate multiple subagents through structured workflows. These stay at the meta-level, never implementing directly. An implementation orchestrator might spawn a planner agent to analyze requirements, a backend developer to implement APIs, a frontend developer to build UI, then a validator to verify integration. The orchestrator maintains the overall task context while each specialist works in isolation. **Critical insight: orchestrators using Task() for dynamic spawning outperform pre-defined subagent lists.** Static delegation creates rigid workflows that don't adapt to actual needs.

Quality-gate agents implement verification workflows with explicit pass/fail criteria. A pre-commit agent checks that tests pass, linting succeeds, and commit messages follow conventions before allowing commits. An architecture reviewer validates that implementations match approved designs before merging. These agents provide **systematic validation that humans would skip under time pressure**, raising overall code quality through consistent enforcement.

The file structure directly impacts maintainability. For projects with 5-10 agents, flat structure works fine: `.claude/agents/code-reviewer.md`, `.claude/agents/test-writer.md`, etc. Beyond 15 agents, categorization helps: organize by function (development/, quality/, infrastructure/), by domain (frontend/, backend/, database/), or by workflow stage (planning/, implementation/, validation/). Some teams adopt plugin architectures where each plugin bundles related agents, commands, and skills.

**CLAUDE.md serves as the constitutional document** for all agent behavior in a project. This file, automatically loaded into context, defines coding standards, common commands, repository structure, testing approaches, and critical constraints. Production teams maintain 13KB files with strict token budgets per topic, treating documentation space like expensive advertising. The forcing function: if explaining a tool requires many tokens, the tool probably needs simplification. CLAUDE.md focuses on what agents get wrong, not comprehensive manuals.

Progressive disclosure through Agent Skills extends context management. Skills follow three-level loading: Level 1 loads just name and description into the system prompt at startup. When Claude needs that expertise, Level 2 loads the full SKILL.md content. Level 3 loads additional linked files on-demand. This architecture enables large knowledge bases without overwhelming context, similar to how humans reference manuals only when needed rather than memorizing everything.

## Writing effective agent prompts and system instructions

**Agent prompts follow a clear structure that consistently produces better results.** Start with role definition: "You are a [specific role] with expertise in [specific domain]." This grounds the agent's identity and triggers relevant training data. Follow with core responsibilities as a focused list—usually 3-7 clear items. Then provide a structured process: numbered steps showing how to approach tasks. Finally, specify output format so agents know what good responses look like.

The most effective prompts balance specificity with flexibility. Overly rigid prompts ("Always do exactly these 15 steps") break when encountering edge cases. Vague prompts ("Help with code") produce generic, unhelpful responses. The sweet spot: **clear principles, suggested process, explicit constraints, but room for agent judgment**. "Prefer functional patterns over classes. Check existing code for conventions. When unsure, ask before implementing" provides guidance without straitjacketing.

Real examples dramatically improve prompt effectiveness. Instead of "Review code for security issues," provide: "Check for SQL injection (example: unsanitized user input in database queries), XSS vulnerabilities (example: unescaped HTML in templates), authentication bypass (example: missing permission checks on sensitive endpoints)." Concrete examples calibrate the agent's understanding better than abstract descriptions.

**The description field determines when agents activate**—treat it as critical as the system prompt. Effective descriptions include keywords for pattern matching ("Use PROACTIVELY after code changes," "MUST BE USED for security-sensitive modifications") and explicit examples showing invocation context:

```markdown
description: |
  Expert code reviewer. Use PROACTIVELY after writing or modifying code.
  Examples:
  <example>
  Context: User has finished implementing authentication
  user: "I've completed the login system"
  assistant: "Let me use code-reviewer to verify the implementation"
  <commentary>Major feature completed, review needed</commentary>
  </example>
```

These structured examples, with \<example\>, \<commentary\>, and context tags, significantly improve auto-invocation rates.

Context protocol patterns enable agent coordination for complex workflows. Agents report completion to a context-manager agent using structured formats (JSON or markdown with consistent sections). The context manager maintains overall task state and coordinates between specialists. This explicit communication pattern prevents confusion when multiple agents contribute to the same feature.

**Negative constraints require special care.** Saying "Never use --force flag" leaves agents stuck when they believe that flag is required. Instead: "Don't use --force flag; prefer --safe-mode flag which provides same capability with safety checks." Always provide alternatives when restricting options. Similarly, avoid comprehensive prohibitions ("Don't modify any existing files") that paralyze useful work—specify what actions are allowed instead.

Anti-patterns in prompt engineering cause subtle but significant problems. The @-mention pattern ("For details see @path/to/docs.md") embeds entire files into context every run, wasting tokens and overwhelming Claude. Better: "For complex usage or specific errors, see path/to/docs.md for troubleshooting." Reference without embedding. Similarly, over-engineering prompts with enterprise patterns ("Use dependency injection, implement interfaces, write comprehensive test suites") adds unnecessary complexity to simple tasks—explicitly state complexity expectations when they differ from defaults.

## Tool permissions and the security model

**Tool access follows the principle of least privilege**—grant only tools each agent needs for its specific role. The default configuration inherits all tools from the main thread, including all connected MCP servers. This works for general-purpose agents but violates security best practices for specialized roles. Read-only agents (reviewers, analyzers, auditors) should explicitly restrict to `Read, Grep, Glob` and explicitly disallow `Write, Edit, Bash`.

The tool specification syntax supports both inclusion and exclusion. Include tools with comma-separated lists: `tools: Read, Write, Edit, Bash, Glob, Grep`. Exclude tools explicitly: `disallowedTools: Write, Edit, Bash`. The second pattern works better for read-only agents—ensures new tools don't accidentally grant write access. For Bash restrictions, use pattern matching: `Bash(git commit:*)` allows only git commit commands, blocking destructive operations.

**MCP tool inheritance requires careful consideration.** When you connect MCP servers (GitHub, databases, cloud providers), those tools become available to all agents by default. A database MCP server grants query execution; a GitHub MCP server enables PR creation and issue management. Agents with full tool access can use these without explicit permission. Production teams create tiered tool access: untrusted agents get minimal tools, validated implementers get standard tools plus specific MCPs, orchestrators get administrative tools.

Permission management operates at three levels: user-level (~/. claude/settings.json), project-level (.claude/settings.json), and runtime (--dangerously-skip-permissions flag). User settings apply across all projects, establishing personal baselines. Project settings override for team consistency. Runtime flags bypass all checks—**only safe in isolated containers without network access**. Production teams document allowed tools in checked-in settings.json so entire team follows consistent policies.

The /permissions command provides interactive tool management during development. Use it to discover available tools (lists all including MCP servers), add specific tools to allowlists, review current permissions, and test agent behavior with different configurations. This experimental workflow—try tools, observe behavior, refine permissions—produces better configurations than purely theoretical access planning.

Real-world permission mistakes create serious problems. The "YOLO mode" (--dangerously-skip-permissions) seems convenient but causes incidents: development database wipes during migration testing, unintended commits pushed to wrong branches, file deletions that bypass backups. Teams running YOLO mode mitigate risks through Docker containers (easily destroyed), dedicated development environments (separate from production), and aggressive git hygiene (frequent commits, feature branches, easy rollbacks).

**Hook-based permission enforcement provides runtime validation** beyond static tool lists. Hooks intercept specific operations (tool use, subagent stops, session ends) and can block, modify, or supplement agent actions. A pre-commit hook verifies tests pass before allowing git commits. A tool use hook validates parameters match expected patterns. This runtime enforcement catches permission issues that static configuration misses, especially when agents chain tools in unexpected ways.

## Testing strategies that catch problems before production

**Test-driven development with Claude Code produces dramatically better results** than implementation-first approaches. The workflow: first, ask Claude to write comprehensive tests based on requirements, explicitly telling it NOT to implement yet. Run the tests and verify they fail (confirming they actually test something). Commit the tests to establish ground truth. Then ask Claude to implement code that passes tests without modifying the tests themselves. This constraint prevents the common failure mode where agents change tests to match buggy implementations.

After 2-3 implementation iterations, introduce independent verification: use a separate subagent or fresh Claude instance to review the implementation against tests. Ask specifically: "Does this implementation overfit to these tests, or would it handle real-world variations?" Fresh context provides unbiased evaluation that catches subtle issues original implementer missed.

Visual testing revolutionizes UI development when properly configured. Give Claude screenshot capability through Puppeteer MCP server or manual screenshot sharing. Provide design mocks (drag-drop images or Figma links). Ask Claude to implement, take screenshots, and iterate until matching the mock. **The key insight: first iteration looks reasonable, but iterations 2-3 produce significantly better results.** The visual feedback loop works far better than text descriptions for layout, spacing, colors, and responsive behavior.

Component-level evaluation decomposes agent behavior into testable units. Break your agent into steps: router (choosing which skill to invoke), skill 1 (executing first capability), skill 2 (executing second capability), output formatter (structuring results). Create separate evaluators for each component. Test router on two axes: ability to choose correct skill given input, and ability to extract correct parameters. This granular testing isolates failures more effectively than end-to-end tests alone.

**Evaluation datasets must include diverse, realistic cases** beyond happy paths. Include in-scope queries the agent should handle successfully. Add out-of-scope questions (small talk, unrelated topics) to verify the agent recognizes boundaries. Include adversarial inputs attempting to bypass constraints or extract sensitive information. Add edge cases with unusual but valid combinations. Real production datasets provide the best test cases when available—synthetic data rarely captures actual complexity.

The three-agent Playwright testing workflow demonstrates specialized testing architecture. A planner agent explores the application autonomously, identifying user paths and edge cases like a QA engineer. A generator agent transforms plans into Playwright tests using semantic locators and proper waiting strategies. A healer agent replays failing tests when changes break them, inspects the current UI to find equivalent elements, and suggests fixes. This specialization produces better test coverage than monolithic testing agents.

Verification strategies vary by output type. **Rules-based feedback works for objective constraints**: email validity, parameter ranges, required fields, format specifications. Define clear rules, provide specific feedback when violated. Visual feedback suits UI work: screenshot outputs, compare to specifications, iterate until matching. LLM-as-judge handles subjective quality: tone appropriateness, style consistency, alignment with brand guidelines. Each verification method targets different quality dimensions.

Multi-agent verification provides unbiased review. After implementing a feature, clear context (or start a second Claude instance) and have that fresh agent review the first agent's work. The review agent hasn't invested in the implementation and catches issues the implementer rationalized away. Start a third instance to edit code based on review feedback, further reducing implementation bias. This separation-of-concerns pattern catches more issues than single-agent review loops.

## Production best practices from enterprise deployments

**Context management determines agent effectiveness at scale.** Fresh Claude sessions in monorepos consume ~20,000 tokens baseline (10% of 200K window). The remaining capacity fills rapidly with conversation history, file contents, and command outputs. Production teams monitor usage with /context command and establish clear patterns: /clear between distinct tasks, not /compact which is opaque and error-prone. For complex multi-day work, implement "document and clear" workflow—have Claude dump plan and progress to .md file, /clear state, then resume from the document in fresh session.

The CLAUDE.md maintenance pattern separates successful teams from struggling ones. Start small, document only what Claude consistently gets wrong. Use the # key during development to have Claude automatically add learnings to CLAUDE.md. Run the file through prompt improvers periodically to compress and clarify. Add emphasis ("IMPORTANT:", "YOU MUST:") to critical rules—Claude's attention mechanisms respond to these markers. Treat the document like expensive advertising space with strict token budgets per section.

**Token economics shift dramatically with Haiku 4.5** (released October 2025). At 90% of Sonnet's agentic performance but 3x cheaper and 2x faster, Haiku changes agent design patterns. Use Haiku for high-frequency operations: code review subagents that run after every change, exploration agents that scan codebases, validation agents that check outputs. Reserve Sonnet for complex implementation and Opus for critical architecture decisions. This strategic model assignment extends Pro subscription usage 3x while maintaining quality.

Planning workflows consistently outperform direct implementation. The "Explore, Plan, Code, Commit" pattern from Anthropic's internal usage: first, explore codebase and requirements; second, create detailed plan (use "think hard" keywords for extended reasoning); third, implement following plan; fourth, commit with clear messages. Teams that skip planning produce 3x more iteration cycles and lower-quality outcomes. For complex work, create plan documents or GitHub issues before implementation—external artifacts serve as reset points when context degrades.

Multi-instance workflows unlock parallel development at individual developer level. Use git worktrees to create lightweight working directories on different branches. Run separate Claude instances in different terminal tabs—one implements feature A, another feature B, a third runs tests. Each maintains independent context and works in parallel. This pattern, impossible with traditional single-thread development, enables throughput previously requiring multiple developers.

**GitHub Actions integration creates self-improving flywheels.** Run Claude Code headless in CI/CD to handle PR reviews, bug fixes, and routine maintenance. Capture full agent logs from these runs. Weekly, query logs to identify common mistakes and failure patterns. Update CLAUDE.md, CLI tools, and abstractions based on findings. This data-driven improvement cycle turns bug reports into systematic fixes—the bug pattern that affected five PRs gets permanently addressed rather than repeatedly hitting similar issues.

Hook-based workflow enforcement catches issues before they reach review. Implement block-at-submit hooks (not block-at-write which confuses agents) that validate quality gates: tests must pass, linting succeeds, commit messages follow format, no TODOs without issue numbers. These hooks create tight feedback loops—agents learn constraints through immediate feedback rather than later PR rejections. Critical distinction: let agents complete plans, then validate final results rather than interrupting mid-work.

Permission strategies balance safety with productivity. Start conservative (default review mode) for first projects. Gradually add common safe operations to allowlists: Edit, Read, Grep, Glob, specific git operations. Deny risky operations explicitly: database migrations, destructive git commands, network operations in sensitive environments. Use project-level settings.json checked into git for team consistency. Individual developers customize with local settings for personal preferences.

## Anti-patterns that degrade agent performance

**Complex slash command systems create anti-patterns**, despite seeming organized. When you force engineers to learn documented magic commands (like `/custom-project-create-api-endpoint`) to accomplish basic work, you've defeated Claude Code's natural language interface. These systems add cognitive overhead, drift from team memory, require maintenance, and often reflect poorly designed underlying tools. Better: create simple personal shortcuts, invest in intuitive CLAUDE.md documentation, simplify tooling so commands aren't necessary.

Custom subagent overuse creates two serious problems. First, gatekeeping context: specialized subagents operate in isolated contexts, hiding information from the main agent that might need it. Second, forcing human workflows: rigid delegation patterns (always use analyzer → implementer → tester) don't adapt to actual task needs. **Better alternative: use built-in Task() feature** to dynamically spawn general-purpose agent clones that inherit context. Let the main agent decide orchestration strategy rather than imposing pre-defined workflows.

Write-time hooks that block mid-plan confuse and frustrate agents. Interrupting during implementation breaks flow and leads to abandoned approaches. Block-at-submit hooks (wrapping git commit operations) work far better—agents complete their plans, then validation occurs before finalizing. This matches human workflows: you finish drafting before spellcheck, complete implementation before submitting PR, not constant interruptions during creative work.

**Context bloat from documentation patterns wastes tokens and degrades performance.** The @-mention anti-pattern embeds entire documentation files into context every run. A 50KB architecture doc added with @ consumes 50K tokens per session regardless of relevance. Better: "For complex FooBar usage or if you encounter FooBarError, see path/to/docs.md for troubleshooting." This reference pattern costs ~50 tokens and points to details when needed.

Negative-only constraints paralyze agents without providing alternatives. "Never use --force flag" leaves agents stuck when their training suggests that flag solves the problem. They'll either violate the constraint or give up entirely. Instead: "Don't use --force flag, prefer --merge-strategy=ours which provides same outcome with safety checks." Provide the better alternative explicitly. Similarly, specify what to do rather than comprehensive lists of prohibitions.

Over-engineering from training bias creates unnecessary complexity. Claude's training includes enterprise best practices that don't suit every context. For proof-of-concept work, explicitly state: "This is a prototype, NOT an enterprise application. Use the simplest approach that could work. Don't implement comprehensive error handling, don't create interface abstractions, don't write extensive test coverage until we validate the concept." Without this guidance, simple buttons get surrounded by unnecessary architecture.

The "lead-specialist" agent pattern (pre-defined orchestrator with rigid subagent delegation) performs worse than dynamic orchestration in practice. Static patterns (always use backend-architect → backend-developer → test-writer) don't adapt to varied tasks. Some tasks need security-first design, others need performance-first optimization, still others need rapid prototyping. Master-clone patterns with dynamic Task() spawning let agents choose appropriate orchestration rather than following human-imposed workflows.

## Practical patterns for different agent types

**Code reviewer agents require careful balance between thoroughness and noise.** The default "review everything" approach produces verbose reports flagging trivial issues. Better: focus exclusively on logic errors, security vulnerabilities, and functionality problems. Explicitly tell reviewers not to comment on variable naming, formatting, or style unless it impacts understanding. Use read-only tools (Read, Grep, Glob) to enforce non-modification. Include git diff in workflow to focus on changed lines rather than reviewing entire files.

Example configuration:
```markdown
---
name: code-reviewer
description: Expert code review. Use PROACTIVELY after code changes.
tools: Read, Grep, Glob, Bash(git diff:*)
model: haiku
---

You are a senior code reviewer ensuring correctness and security.

When invoked:
1. Run git diff to see recent changes
2. Focus only on modified lines
3. Check for: logic errors, security vulnerabilities, performance problems
4. Ignore: variable naming, formatting, style preferences

IMPORTANT: Only report bugs and security issues. Be concise.
Three sentence maximum per issue: what, why, how to fix.
```

**Explorer and scanner agents excel at codebase comprehension** for onboarding or feature planning. They systematically map structure, identify patterns, document conventions, and create navigational guides. Use read-only tools plus Bash for git operations (git log, git blame, git grep). Have them produce structured markdown outputs: architecture overview, key files and purposes, patterns and conventions, entry points, common workflows.

Implementation workflow: when starting work in unfamiliar codebase, invoke explorer to build mental model. Have it create maps of relevant modules, trace feature implementations end-to-end, identify similar features for pattern matching. This upfront exploration (5-10 minutes) saves hours of confused implementation. The Anthropic team reports this as a primary onboarding accelerator—new engineers use Claude to understand codebases in days rather than weeks.

**Implementer agents benefit from explicit framework and pattern guidance.** A Django implementer should document project-specific patterns: file locations (models in apps/*/models.py, views in apps/*/views.py), testing approach (pytest-django with factory patterns), ORM patterns (select_related usage, custom managers), API patterns (DRF with ViewSets and serializers). Include anti-patterns for the framework: don't query in loops (N+1), don't put logic in serializers, don't skip migrations.

Validator agents verify outputs against specifications through systematic checks. A test validator confirms: all functions have corresponding tests, test names describe what they verify, tests actually test behavior (not just call functions), edge cases have coverage, tests are independent. An API validator checks: all endpoints documented, authentication required where needed, input validation present, error responses follow conventions, rate limiting configured.

**Orchestrator agents stay at meta-level, never implement directly.** They analyze tasks to identify required domains (frontend, backend, database), determine dependencies between subtasks, spawn appropriate specialists using Task(), coordinate information flow between agents, and verify integration. Use Task tool only—orchestrators shouldn't read or write code. Their value comes from maintaining strategic context while specialists work in isolated tactical contexts.

Documentation specialists create maintainable documentation that developers actually read. They generate API specs from code (OpenAPI/Swagger), write realistic usage examples (not toy "hello world"), document common error scenarios with solutions, maintain changelogs with meaningful descriptions, and update troubleshooting guides based on recurring issues. Key anti-pattern: don't just copy docstrings. Document why decisions were made, what edge cases exist, what tradeoffs were considered.

## Workflows for testing, debugging, and maintenance

**The TDD workflow with Claude produces consistently better results** than implementation-first approaches. Start by having Claude write comprehensive tests based on requirements—explicitly state you're doing TDD to prevent it from creating mock implementations. Ask it to run tests and confirm they fail. Commit these failing tests. Then request implementation that passes tests without modifying them. Allow 2-3 iteration cycles for refinement. Finally, use independent subagent verification to ensure implementation doesn't overfit to test cases.

Debugging multi-step failures requires systematic decomposition. When an agent produces wrong outputs, break the task into components: understand the input, choose the right approach, execute the approach, format the output. Test each component independently. Did it misunderstand requirements (input problem), choose wrong tools (routing problem), use tools incorrectly (execution problem), or produce wrong format (output problem)? This decomposition identifies root causes faster than "try again" iteration.

**Course correction during development uses several mechanisms.** The Escape key interrupts agents during any phase—thinking, tool calls, or file edits. Double-tapping Escape jumps back in history to edit previous prompts. Asking Claude to undo changes often works better than manual reversion. The critical skill: intervening early when agents go wrong, before they compound initial mistakes with follow-on changes.

Context management debugging addresses the most common production issue. When agents become confused or repetitive, check context usage with /context command. If approaching 180K+ tokens (out of 200K), either /clear and resume with context document, or use /compact sparingly (prefer clear because compact is opaque). Create custom /catchup commands that reload only changed files after clearing—more efficient than manual @-mentions.

Observability patterns from enterprise deployments provide essential monitoring. Implement distributed tracing to track agent execution flows. Log all tool calls with parameters, outcomes, and timing. Record reasoning chains showing why agents made decisions. Group related operations by thread_id for multi-turn conversations. Set up alerts on unusual patterns: excessive retries, timeout spikes, error rate increases, cost anomalies.

**The data-driven improvement flywheel creates self-improving systems.** Run Claude Code in CI/CD capturing full logs. Weekly, query these logs for common failure patterns. Identify recurring mistakes: wrong tool selections, misinterpreted requirements, missed edge cases, performance issues. Update CLAUDE.md with corrections. Improve CLI tools based on usage patterns. Add hooks to prevent recurring errors. This systematic approach turns individual bugs into permanent fixes.

Evaluation pipelines in production provide continuous quality assessment. Run LLM judges on production outputs scoring relevance, correctness, and tone. Collect user feedback through thumbs up/down and follow-up queries (repeated question signals poor initial answer). Compare automated scores with user signals to calibrate judges. Set alert thresholds that trigger reviews when quality drops. Integrate evaluations into CI/CD to catch regressions before deployment.

Version control for agents themselves prevents subtle degradation. Check agent configurations into git like any code. Review changes to prompts, tool permissions, or model selections. Test agents before deploying to entire team. Monitor metrics before/after changes: task completion rate, iteration count, token usage, error rates. Roll back changes that degrade performance. This discipline treats agents as infrastructure requiring proper change management.

## Decision frameworks for confident agent creation

**The agent design decision tree provides systematic guidance.** First question: is this one-time work with immediate context? Use inline prompts. Second: do multiple agents need this expertise? Create a Skill (portable, reusable). Third: is this a recurring workflow with specific tool restrictions? Create custom agent. Fourth: does this need external system access with authentication? Create or configure MCP server. Fifth: is this shared team knowledge? Use Project (Team/Enterprise feature).

Model selection follows task complexity and frequency. **For critical planning and architecture: Opus 4.1** (deepest reasoning, highest cost, use sparingly). For standard implementation work: Sonnet 4.5 (balanced capability and cost, default choice). For high-frequency routine tasks: Haiku 4.5 (90% of Sonnet performance, 3x cheaper, 2x faster). For validation and parsing: Haiku 3.5 (fastest, cheapest). This strategic assignment can triple subscription value while maintaining quality.

Agent value assessment requires honest evaluation beyond initial excitement. High-value scenarios: maintenance and migrations (tech debt cleanup), codebase exploration and onboarding, test generation and review, parallel development streams, prototype to proof-of-concept, incident response and debugging, repetitive tasks with clear patterns. Low-value scenarios: tasks requiring deep business context not in codebase, novel algorithms without training data, highly creative architectural decisions, tasks where iteration exceeds 20 cycles (context degradation), complex CSS layouts (lacks primitives).

**Tool permission patterns follow role-based templates.** Read-only analyzers (reviewers, auditors, scanners): Read, Grep, Glob only, explicitly disallow Write/Edit/Bash. Research specialists (analysts, planners): add WebFetch, WebSearch for information gathering. Code implementers (developers, engineers): Read, Write, Edit, Bash, Glob, Grep with appropriate hooks. Documentation specialists: Read, Write, Edit, Glob, Grep, WebFetch, WebSearch. Orchestrators: Task tool only for spawning subagents, Read/Grep/Glob for context awareness.

The simple-versus-complex agent decision depends on initialization costs and usage frequency. Lightweight agents under 3K tokens initialize quickly, supporting frequent invocation and low overhead. Best for: code review, linting, simple validations, format checking. Medium-weight agents 10-15K tokens balance capability with efficiency. Suitable for: test strategy generation, documentation creation, refactoring guidance. Heavy agents 25K+ tokens provide deep analysis but cost more to initialize. Reserve for: architecture review, security auditing, complex refactoring, cross-cutting analysis.

Orchestration architecture choices profoundly impact effectiveness. Master-clone pattern (main agent dynamically spawns general Task() agents) adapts to varied situations, maintains context transparency, enables flexible coordination. Lead-specialist pattern (pre-defined orchestrator with rigid subagent list) forces human workflows, hides context in specialized agents, fails to adapt to task variations. **Production experience strongly favors master-clone for complex work**, reserving lead-specialist only for repetitive workflows with consistent structure.

## Creating your first agents: Step-by-step processes

**Start with a simple code reviewer to learn the fundamentals.** Create file `.claude/agents/code-reviewer.md` in your project. Add YAML frontmatter specifying name, description with explicit activation keywords, read-only tools, and Haiku model for efficiency. Write concise system prompt: role statement, numbered process steps, output format specification, anti-pattern avoidance. Test by making code changes and asking "Review my recent changes." Iterate based on output quality—too verbose means tighten constraints, too shallow means expand responsibilities.

Building a framework-specific implementer requires documenting patterns. For a Django developer agent, specify: file structure conventions (where models/views/serializers live), generator commands to use first (manage.py commands), testing patterns (pytest-django with factories), ORM anti-patterns (N+1 queries), authentication requirements, API conventions (DRF ViewSets). Test with representative tasks: "Add User profile model with bio field," "Create API endpoint for user search." Verify it follows existing patterns rather than inventing new approaches.

**Creating orchestrator agents starts with clear task decomposition.** Define domains your projects typically touch: frontend, backend, database, DevOps, security. Write orchestrator prompt focusing on analysis and coordination, not implementation. Give it only Task, Read, Grep, Glob tools. Include coordination protocol: analyze task to identify required domains, determine dependencies and parallelization opportunities, spawn specialists with clear instructions, integrate results. Test with multi-domain features to verify effective coordination.

Quality-gate validator creation emphasizes clear pass/fail criteria. For a pre-commit validator: specify checks (tests passing, linting clean, formatting applied), define how to verify each (run test suite, run linter, run formatter in check mode), determine blocking versus warning issues, format output clearly (passed checks, failed checks, how to fix). Integrate with hooks to run before commits. Test by intentionally creating violations and verifying it catches them.

Documentation specialist development benefits from examples. Provide samples of good documentation (clear purpose statements, realistic usage examples, troubleshooting sections). Specify format (API specs in OpenAPI, guides in markdown, examples in code comments). Define anti-patterns (copying docstrings verbatim, toy examples, ignoring edge cases). Test with undocumented modules and verify output helps actual users.

Iterative refinement follows predictable patterns. Initial version handles happy path but misses edge cases. Second version catches edge cases but becomes too verbose or strict. Third version balances thoroughness with usability. This cycle typically requires 5-10 test iterations. **Key practice: maintain test cases that failed previously** to verify improvements don't regress. Build evaluation sets from real usage examples, not imagined scenarios.

## Managing agents in production environments

**Production deployment starts with representative test sets.** Collect 20-50 realistic queries your agent should handle successfully. Include 5-10 out-of-scope queries to verify boundary recognition. Add 5-10 adversarial queries testing constraint adherence. Include edge cases with unusual but valid combinations. Run these tests before deployment. Establish baseline metrics: task completion rate, iteration count, token usage, response time.

Continuous evaluation provides ongoing quality assurance. Instrument production agents with logging: capture inputs, outputs, tool calls, timing, errors. Sample 10-20% of production runs through LLM judges scoring quality dimensions (relevance, correctness, completeness, tone). Collect user feedback (thumbs, follow-ups, abandonment). Correlate automated scores with user signals to validate judges. Set alert thresholds triggering review when metrics degrade.

**Context management at scale requires systematic practices.** Establish clear /clear policies: between distinct tasks, when context exceeds 150K tokens, when agent behavior becomes confused or repetitive. Create project-specific /catchup commands that reload only changed files after clearing. Document plan checkpoints in .md files for complex work spanning multiple sessions. Train team on "document and clear" workflow for week-long features.

Settings and configuration management ensures consistency. Store project-level settings in .claude/settings.json checked into git: tool allowlists, timeout values, default permissions, MCP configurations. Document customization in README.md. Use ~/.claude/settings.json for personal preferences that shouldn't affect team. Periodically audit allowed tools and permissions—remove unused grants, tighten restrictions on sensitive operations.

CLAUDE.md maintenance follows monthly cycles. Review agent logs identifying common mistakes. Add concise guidelines addressing recurring issues. Run file through prompt improver to compress and clarify. Add emphasis markers to critical rules. Test that changes improve behavior without causing new issues. Keep file under 15KB by removing outdated guidance. This living documentation continuously improves based on actual usage patterns.

**Hook evolution requires treating hooks as production code.** Version control all hooks in .claude/hooks/. Test hooks thoroughly—bad hooks cause mysterious failures. Keep hooks idempotent (safe to run multiple times). Validate JSON in hook configurations. Document hook purposes and maintenance procedures. Prefer non-blocking hint hooks over blocking enforcement hooks except for critical quality gates.

MCP server configuration management establishes tool availability. User-level MCPs (~/.claude/settings.json) provide personal tools across projects. Project-level MCPs (.claude/settings.json) add project-specific integrations. Checked-in MCPs (.mcp.json) ensure team consistency. Document which MCPs provide which capabilities in README.md. Regularly audit MCP tool grants—some MCPs expose dozens of tools, many unnecessary for your agents.

Agent deprecation and replacement follows systematic process. Document why agent no longer serves well: overlaps with other agents, maintenance burden too high, patterns changed making it obsolete. Identify replacement approach: merge with another agent, convert to Skill, simply remove. Communicate changes to team. Archive old agent with deprecation notice. Monitor for uses trying to invoke removed agent and provide migration guidance.

## Special considerations for medical professionals learning development

**Your medical background provides unique advantages in agent design.** Clinical thinking translates directly: differential diagnosis parallels debugging (systematic elimination), protocol adherence matches quality gates (verification checklists), evidence-based medicine aligns with evaluation-driven development (data validates approaches). Use this mental framework—you're not learning something foreign, you're applying familiar systematic thinking to different domains.

Medical documentation habits serve you well. Just as clinical notes follow structured formats (SOAP, APSO, HPI), agent prompts follow consistent patterns (role, responsibilities, process, output). Just as you document procedures for reproducibility, agent configurations capture decision-making. Just as you track patient outcomes to refine treatment, agent metrics guide improvements. The discipline of thorough documentation transfers directly.

**Start with agents that match clinical patterns you already know.** Code reviewers parallel peer review—systematic evaluation against standards. Test validators parallel quality assurance—verification before deployment. Documentation specialists parallel medical writing—clear communication of complex information. These familiar patterns provide comfortable entry points before tackling novel agent types.

Your existing agents demonstrate you've already mastered core concepts. A code reviewer that catches bugs parallels diagnostic pattern recognition. A codebase scanner that maps architecture parallels systematic examination. An implementation planner that structures work parallels treatment planning. You're not starting from zero—you're extending skills you've already developed through practical application.

Technical terminology often obscures simple concepts. "Context window" just means working memory—like how you can hold 7±2 facts in mind. "Token" means word-chunks—the unit of text processing. "Model" means the AI system—like specialist consultants. "Tools" means available actions—like procedures you can perform. "Hooks" means triggers—like lab flags that initiate protocols. Translating jargon to familiar concepts removes artificial barriers.

**The learning path follows natural progression.** First, use existing agents to accomplish work—build intuition through application. Second, modify existing agents to fit your needs—small, low-risk changes with immediate feedback. Third, create simple focused agents for clear use cases—build confidence with successes. Fourth, experiment with orchestration and coordination—tackle complexity incrementally. This matches medical training: observe, assist, perform under supervision, perform independently.

Common beginner mistakes reflect learning process, not capability limitations. Over-engineering agents (too complex, too many features) comes from not yet knowing what simplicity works. Under-specifying constraints (vague prompts, missing examples) reflects incomplete mental models. Tool over-permissioning (granting unnecessary access) shows unfamiliarity with security principles. All correct through experience—they're learning opportunities, not failures.

Pair programming with Claude accelerates learning safely. For new techniques, implement side-by-side with Claude: you write code, Claude reviews. You explain what you want, Claude implements, you evaluate. This collaborative approach builds skills while Claude handles details you haven't memorized yet. Over time, you'll internalize patterns and need less assistance—but the scaffolding prevents being overwhelmed during learning.

## Emerging patterns and future-looking considerations

**Agent ecosystems continue rapid evolution, with clear trends emerging.** The shift from static orchestration to dynamic coordination continues—fewer pre-defined agent workflows, more runtime decision-making. Progressive disclosure through Skills scales better than monolithic agents—knowledge loads when needed rather than polluting context. Hybrid model usage (Opus planning, Sonnet implementing, Haiku validating) optimizes cost-performance tradeoffs. These patterns will strengthen as models improve and tooling matures.

The "Jira ticket in, reviewed PR out" vision represents the aspirational endpoint. Currently achievable for well-scoped tickets in well-documented codebases. The gap: handling ambiguous requirements, navigating complex interdependencies, making architectural decisions with incomplete information. Future improvements will address these through better reasoning models, more sophisticated context management, enhanced tool integration. Track progress by testing agents on progressively complex tickets.

**Integration depth continues expanding beyond code.** Current state: agents handle implementation, testing, documentation, review. Near future: agents manage entire feature lifecycles (requirements analysis, implementation, deployment, monitoring). Longer term: agents maintain codebases autonomously (security patching, dependency updates, performance optimization, refactoring). Your current agent expertise positions you well as these capabilities mature—the fundamentals remain consistent.

MCP ecosystem growth provides expanding capabilities. Current count: 100+ MCP servers for databases, cloud platforms, monitoring systems, communication tools, data sources. Each new MCP adds capabilities to all your agents automatically. Watch for MCPs relevant to your domains: healthcare data sources, FHIR servers, medical device interfaces, clinical workflow systems. These will enable specialized medical software development workflows.

Evaluation and observability tools mature rapidly. Current generation: basic logging, simple metrics, manual evaluation. Next generation: automated evaluation pipelines, semantic monitoring, intelligent alerting, drift detection. These enable truly autonomous agent operation—systems that recognize and correct their own degradations. Invest in observability foundations now; sophisticated tooling will build on these basics.

The convergence toward agent-first development changes software engineering fundamentally. Current approach: human writes code, agent assists with specific tasks. Emerging approach: agent writes code, human provides requirements and reviews outputs. Future approach: agent manages entire development lifecycle, human focuses on product and architecture. **Your current investment in learning agent patterns positions you for this transition**—you're building skills that become more valuable as agents become more capable.

## Synthesis: Principles that transcend specific implementations

The most powerful agents share common characteristics transcending implementation details. **Single responsibility sharply defines what agents do and don't handle**—better to chain three focused agents than build one that tries everything. Clear boundaries prevent scope creep and confusion. When an agent's description requires multiple "and also" clauses, split it.

Minimal tool access reduces both security risks and agent confusion. Every additional tool increases decision space—more ways to approach problems means more ways to make wrong choices. Grant exactly the tools needed for the agent's responsibility. Read-only agents get Read/Grep/Glob. Implementers get Write/Edit. Orchestrators get Task. Resist granting additional tools "just in case"—add them when actual needs emerge.

**Context isolation enables sophisticated workflows impossible otherwise.** Subagents provide fresh perspectives, prevent detail pollution in main conversations, enable parallel execution, create verification independence. Teams that underutilize subagents because "they seem complicated" miss dramatic productivity gains. The complexity investment pays returns immediately through cleaner conversations and better outcomes.

Systematic verification catches issues humans skip under pressure. Automated quality gates enforce testing standards, code formatting, commit message conventions, documentation requirements. Independent review subagents provide unbiased evaluation. Visual feedback loops validate UI implementations. Rules-based checking verifies objective constraints. This multi-layered verification raises consistent quality bars.

Documentation as code ensures agents and teams stay aligned. CLAUDE.md lives in git alongside code, evolving as the project evolves. Agent configurations version control like any infrastructure. Team shares knowledge through checked-in settings rather than oral tradition. This documentation-as-artifact approach scales better than informal knowledge sharing.

**Iterative refinement based on actual usage produces better agents than upfront theorizing.** Start simple, deploy quickly, monitor carefully, refine based on real behavior. Maintain evaluation datasets capturing cases where agents failed. Test improvements against these cases. This empirical approach—hypothesis, test, observe, refine—matches scientific method applied to agent development.

The human-agent collaboration model emphasizes strengths of each. Humans provide: strategic thinking, requirement interpretation, creative problem-solving, ethical judgment, final accountability. Agents provide: systematic execution, comprehensive testing, parallel work capacity, consistent quality enforcement, tireless iteration. Effective collaboration maximizes both rather than replacing humans with agents.

Your journey from medical professional to developer using agents demonstrates these principles. You've built expertise through systematic application: using agents to accomplish real work, modifying configurations based on outcomes, creating new agents for emerging needs, developing judgment about what works. This practical, iterative, evidence-driven approach—familiar from medicine—succeeds in software development too. The specific techniques matter less than the systematic thinking they embody. Master the principles, and the implementations follow naturally.