# Multi-Agent Workspace Orchestrator

 A VS Code–first application for running, monitoring, and intelligently coordinating multiple AI coding agents from one interface.

 The goal is to make **Codex, Cursor, Google Antigravity, Claude Code, GitHub Copilot, Cline, and other compatible agents** work together as a coordinated development team instead of requiring the developer to manually switch between terminals and applications.

 Modern VS Code already supports multiple agent sessions and third-party agents such as Claude and Codex, while MCP provides a practical interoperability layer for connecting agent tools.  Visual Studio Code+1

 > **Important:** The application should use official agent CLIs/APIs and supported authentication mechanisms. It must not scrape private credentials, bypass provider limits, or attempt to circumvent provider-enforced quotas.

---

 ## 1\. Vision

 The application acts as an **Agent Orchestrator / Agent Control Center**.

 Instead of the developer deciding:

 > "Which agent should I use for this task?"

 the application analyzes the task, repository, current workload, previous agent performance, available quota, cooldowns, and responsibilities, then recommends or automatically assigns the appropriate agent.

 Example:

```
Developer:
"Build the authentication system and connect it to the dashboard."

                    │
                    ▼
             ┌──────────────┐
             │ Task Analyzer│
             └──────┬───────┘
                    │
       ┌────────────┼─────────────┐
       ▼            ▼             ▼
   Backend        Frontend       Tests
       │            │             │
       ▼            ▼             ▼
     Codex        Cursor       Claude
       │            │             │
       └────────────┼─────────────┘
                    ▼
             Agent Coordinator
                    │
                    ▼
              Shared Workspace
```

---

 # 2\. Core Features

 ## Multi-Agent Support

 The application should support agents through adapters rather than hard-coding one provider.

 Initial adapters:

 - OpenAI Codex
- Cursor
- Google Antigravity
- Claude Code
- GitHub Copilot
- Cline
- Windsurf
- Other MCP-compatible agents
- Custom/local CLI agents

 The architecture should allow new agents to be added without modifying the orchestration engine.

 Some existing tools already demonstrate that multiple coding-agent CLIs can be launched from a unified VS Code environment, including Codex, Antigravity, Cursor, Claude Code and others.  Visual Studio Marketplace+1

---

 # 3\. Account Manager

 Provide a unified account-management screen.

```
AGENT ACCOUNTS

┌───────────────────────────────────────────────┐
│ Codex                                         │
│ ● Account: Personal                           │
│   Status: Connected                           │
│   Quota: 72% remaining*                       │
│   Reset: 4h 32m*                              │
│                                               │
│ [Switch Account] [Refresh] [Settings]        │
├───────────────────────────────────────────────┤
│ Cursor                                        │
│ ● Account: Work                               │
│   Status: Connected                           │
│   Usage: Available through provider API       │
│                                               │
│ [Switch Account] [Refresh] [Settings]        │
├───────────────────────────────────────────────┤
│ Antigravity                                   │
│ ● Account: Google                              │
│   Status: Connected                           │
│   Quota: Provider-reported status              │
│                                               │
│ [Switch Account] [Refresh] [Settings]        │
└───────────────────────────────────────────────┘
```

 ### Account requirements

 - Multiple accounts per provider
- Account aliases
- Active/inactive account
- Login/logout
- Account switching
- Account health
- Last successful authentication
- Provider-specific quota information
- Provider reset time when officially exposed
- Secure credential storage
- Never display access tokens or passwords

 ### Important quota rule

 Quota information must come from **officially exposed provider information, supported CLI output, APIs, or user-configured telemetry**.

 If a provider does not expose exact real-time quota information, the UI must clearly display:

```
Quota information unavailable from provider
```

 rather than inventing a number.

---

 # 4\. Real-Time Agent Dashboard

 The main dashboard should show every active agent.

```
┌──────────────────────────────────────────────────────────┐
│                 AGENT CONTROL CENTER                     │
├──────────────┬───────────┬───────────┬──────────────────┤
│ Agent        │ Status    │ Task      │ Usage            │
├──────────────┼───────────┼───────────┼──────────────────┤
│ Codex        │ 🟢 Working│ Backend   │ 72% available*  │
│ Cursor       │ 🟡 Waiting│ UI        │ Provider data    │
│ Antigravity  │ 🟢 Working│ Research  │ Provider data    │
│ Claude       │ 🔵 Review │ Tests     │ Provider data    │
│ Copilot      │ ⚪ Idle   │ —         │ Provider data    │
└──────────────┴───────────┴───────────┴──────────────────┘
```

 Each agent should expose:

 - Current task
- Current status
- Files being modified
- Current branch/worktree
- Last action
- Current session
- Runtime
- Estimated usage
- Provider-reported quota where available
- Cooldown
- Errors
- Permission requests
- Recent completed tasks

---

 # 5\. Permission Manager

 Provide a central permission system.

```
PERMISSION CENTER

Agent: Codex
Request:
"Execute npm install"

Risk:
MEDIUM

[Allow Once]
[Allow for Workspace]
[Allow Automatically]
[Deny]
```

 ## Permission modes

 ### Manual

 Every sensitive operation requires confirmation.

```
Always ask before:
- terminal commands
- file deletion
- dependency installation
- git push
- external network access
- database modification
```

 ### Automatic

 Automatically approve operations matching configured rules.

 Example:

```
permissions:
  terminal:
    enabled: true

  read_files:
    enabled: true

  write_files:
    enabled: true

  delete_files:
    enabled: false

  git_push:
    enabled: false

  package_install:
    enabled: true
```

 ### Smart

 Use risk classification.

```
LOW
├── Read source file
├── Search repository
└── Run formatter

MEDIUM
├── Modify source
├── Install package
└── Run build

HIGH
├── Delete files
├── Change secrets
├── Git push
└── Production deployment
```

 High-risk actions should require explicit approval unless the user deliberately enables an automation policy.

---

 # 6\. Agent Responsibility Manager

 The application maintains a responsibility map.

 Example:

```
responsibilities:

  frontend:
    primary: cursor
    backup:
      - claude
      - codex

  backend:
    primary: codex
    backup:
      - claude

  database:
    primary: codex
    backup:
      - claude

  testing:
    primary: claude
    backup:
      - codex

  research:
    primary: antigravity

  documentation:
    primary: copilot
```

 The assignment engine can then determine:

```
Task:
"Fix React dashboard layout"

Detected domain:
frontend

Preferred agent:
Cursor

Fallback:
Claude

Current Cursor state:
Available

Assignment:
Cursor
```

---

 # 7\. Intelligent Task Router

 The router analyzes incoming work using:

 - Keywords
- Repository structure
- File extensions
- Git history
- Current agent workload
- Previous task results
- Agent capabilities
- Agent responsibility
- Quota availability
- Cooldown
- Estimated task complexity
- Current project phase

 Example:

```
"Fix PostgreSQL migration and update API"

        ↓

Keywords:
postgresql
migration
API

        ↓

Domain:
backend/database

        ↓

Agent selection:
Codex

        ↓

Check:
Quota ✓
Cooldown ✓
Workload ✓
Permission policy ✓

        ↓

Assign task
```

---

 # 8\. Project Phase Management

 The orchestrator should understand that different agents may own different portions of a project.

 Example:

```
PROJECT: E-Commerce Platform

Phase 1
├── Architecture
│   └── Claude
│
├── Database
│   └── Codex
│
└── API
    └── Codex

Phase 2
├── Frontend
│   └── Cursor
│
├── UI/UX
│   └── Antigravity
│
└── Documentation
    └── Copilot

Phase 3
├── Integration
│   └── Codex
│
├── Testing
│   └── Claude
│
└── Review
    └── Multiple agents
```

 The user should be able to manually override any assignment.

---

 # 9\. Agent Memory

 The application should maintain project-level orchestration memory.

 Store:

```
Task
Agent
Files changed
Result
Tests
Failures
Duration
Estimated usage
Actual usage when available
Review result
Human feedback
```

 Example:

```
{
  "task": "Implement authentication API",
  "agent": "codex",
  "result": "completed",
  "files_changed": 14,
  "tests_passed": 27,
  "tests_failed": 0,
  "duration_seconds": 842
}
```

 This allows the router to learn historical suitability.

 For example:

```
Backend API tasks
Codex
Successful: 18
Failed: 2

Frontend tasks
Cursor
Successful: 22
Failed: 3

Testing tasks
Claude
Successful: 16
Failed: 1
```

 This history should inform routing, but the system should not pretend that historical success guarantees future success.

---

 # 10\. Quota Manager

 The quota manager has two modes.

 ## Manual Mode

 Developer chooses:

```
Agent: Codex
Maximum daily usage:
[ 70% ]

Agent: Cursor
Maximum usage:
[ 80% ]

Agent: Antigravity
Maximum usage:
[ 60% ]
```

 ## Automatic Mode

 The scheduler manages available capacity.

```
                QUOTA MANAGER

Codex          ███████████████░░░ 72%
Cursor         ███████████░░░░░░░ 58%
Antigravity    █████████████████░ 86%
Claude         █████████░░░░░░░░░ 47%

Current workload:
Codex        2 tasks
Cursor       1 task
Antigravity  0 tasks
Claude       1 task
```

 The scheduler considers:

```
availability
+
quota
+
cooldown
+
task priority
+
task complexity
+
agent responsibility
+
historical performance
=
assignment decision
```

---

 # 11\. Cooldown System

 Each agent can have a cooldown policy.

 Example:

```
cooldown:

  codex:
    after_task: 30s
    after_rate_limit: 10m

  cursor:
    after_task: 15s

  antigravity:
    after_rate_limit: 5m
```

 The scheduler should not repeatedly hammer an unavailable provider.

```
Codex
Status: COOLDOWN

Available again:
02:41

Scheduler:
Using Cursor for compatible task
```

---

 # 12\. Automatic Load Balancing

 Example:

```
Incoming tasks:

1. React component
2. API endpoint
3. Unit tests
4. Database migration
5. Documentation
```

 Router:

```
React component
        → Cursor

API endpoint
        → Codex

Unit tests
        → Claude

Database migration
        → Codex

Documentation
        → Copilot
```

 If Codex becomes unavailable:

```
Codex
↓
Unavailable

API task
↓
Fallback evaluation
↓
Claude
↓
Assignment
```

---

 # 13\. Real-Time Monitoring

 The system should continuously monitor:

```
Agent process
Terminal
Workspace
Git
Task state
Permission requests
Quota information
Provider errors
Cooldown
File changes
Test results
```

 Dashboard example:

```
LIVE

19:31:02  Cursor     modified src/components/Dashboard.tsx
19:31:06  Codex      running database migration
19:31:09  Claude     running test suite
19:31:12  Antigravity analyzing UI requirements
19:31:15  Codex      permission requested: npm install
```

---

 # 14\. File Ownership

 Avoid multiple agents editing the same files simultaneously.

```
FILE OWNERSHIP

src/frontend/
    Cursor

src/backend/
    Codex

tests/
    Claude

docs/
    Copilot
```

 If another agent requests a locked file:

```
File:
src/backend/auth.ts

Owner:
Codex

Requester:
Cursor

Action:
WAIT

Reason:
File currently locked by Codex
```

---

 # 15\. Git / Worktree Isolation

 Agents should preferably work in isolated Git worktrees.

```
repository
│
├── main
│
├── agent/codex/backend
├── agent/cursor/frontend
├── agent/claude/tests
└── agent/antigravity/research
```

 This reduces accidental conflicts when multiple agents operate simultaneously.

---

 # 16\. Agent Handoff

 Agents should be able to hand work to another agent.

 Example:

```
Cursor:

"Frontend implementation complete."

        ↓

Handoff

        ↓

Claude:

"Review UI implementation."

        ↓

Handoff

        ↓

Codex:

"Fix integration issues."

        ↓

Final reviewer
```

 The handoff record should contain:

```
{
  "from": "cursor",
  "to": "claude",
  "task": "Review dashboard",
  "files": [
    "src/components/Dashboard.tsx"
  ],
  "context": "Implementation completed; review responsive behavior."
}
```

---

 # 17\. Main UI

 Recommended VS Code sidebar:

```
┌──────────────────────────────┐
│ AGENT ORCHESTRATOR           │
├──────────────────────────────┤
│ 🟢 Dashboard                 │
│                              │
│ 🤖 Agents                    │
│   ├─ Codex                   │
│   ├─ Cursor                  │
│   ├─ Antigravity             │
│   ├─ Claude                  │
│   └─ Copilot                 │
│                              │
│ 📋 Tasks                     │
│                              │
│ 🧠 Responsibilities          │
│                              │
│ 📊 Quotas                    │
│                              │
│ 🔐 Permissions               │
│                              │
│ 🧠 Memory                    │
│                              │
│ 🔀 Handoffs                  │
│                              │
│ ⚙ Settings                   │
└──────────────────────────────┘
```

---

 # 18\. Dashboard Layout

```
┌─────────────────────────────────────────────────────────┐
│ AGENT ORCHESTRATOR                         ● LIVE        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Active Agents                                           │
│                                                         │
│ Codex       🟢 Backend       72% available*             │
│ Cursor      🟢 Frontend      Provider data              │
│ Claude      🟡 Testing       Provider data              │
│ Antigravity 🟢 Research      Provider data              │
│ Copilot     ⚪ Idle          Provider data              │
│                                                         │
├─────────────────────────────────────────────────────────┤
│ TASK QUEUE                                              │
│                                                         │
│ #41 Authentication API       → Codex                    │
│ #42 Dashboard UI             → Cursor                   │
│ #43 Test coverage            → Claude                   │
│ #44 UX analysis              → Antigravity              │
│                                                         │
├─────────────────────────────────────────────────────────┤
│ LIVE ACTIVITY                                           │
│                                                         │
│ Codex      Running migration                           │
│ Cursor     Editing Dashboard.tsx                       │
│ Claude     Running Jest                                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

 `*` Exact quota should only be shown when supported by the provider.

---

 # 19\. Architecture

 Recommended architecture:

```
                    VS CODE EXTENSION
                           │
                           ▼
                 ┌───────────────────┐
                 │   UI / Dashboard  │
                 └─────────┬─────────┘
                           │
                           ▼
                 ┌───────────────────┐
                 │ Orchestrator Core │
                 └─────────┬─────────┘
                           │
        ┌──────────────────┼───────────────────┐
        ▼                  ▼                   ▼
 Task Router        Quota Manager       Permission Manager
        │                  │                   │
        └──────────────────┼───────────────────┘
                           ▼
                 ┌───────────────────┐
                 │ Agent Registry    │
                 └─────────┬─────────┘
                           │
        ┌──────────┬───────┼────────┬──────────┐
        ▼          ▼       ▼        ▼          ▼
      Codex     Cursor   Claude   Antigravity Copilot
        │          │       │        │          │
        └──────────┴───────┴────────┴──────────┘
                           │
                           ▼
                     MCP / CLI / API
```

---

 # 20\. Suggested Technology Stack

 ### VS Code Extension

 - TypeScript
- VS Code Extension API
- Webview / React
- Node.js

 ### Orchestration Engine

 - TypeScript
- Event-driven architecture
- SQLite for local state
- JSON/YAML configuration

 ### Agent Communication

 Prefer:

 1. Official APIs
2. Official CLIs
3. MCP
4. Provider-supported integrations

 MCP is particularly useful because multiple current AI coding tools support MCP-based integrations.  GitHub+1

 ### Storage

```
~/.agent-orchestrator/
│
├── config/
├── accounts/
├── memory/
├── history/
├── metrics/
└── logs/
```

 Sensitive credentials should use the operating system's secure credential/keychain facilities rather than plaintext files.

---

 # 21\. Agent Adapter Interface

 Every provider should implement the same interface.

```
interface AgentAdapter {
  id: string;
  name: string;

  detect(): Promise<boolean>;

  authenticate(): Promise<void>;

  getAccounts(): Promise<Account[]>;

  getStatus(): Promise<AgentStatus>;

  getQuota(): Promise<QuotaInfo | null>;

  launch(task: AgentTask): Promise<AgentSession>;

  stop(sessionId: string): Promise<void>;

  getSession(sessionId: string): Promise<AgentSession>;

  getCapabilities(): Promise<AgentCapabilities>;

  requestPermission(
    request: PermissionRequest
  ): Promise<PermissionDecision>;
}
```

 This makes adding a new agent straightforward.

---

 # 22\. Task Router Interface

```
interface TaskRouter {
  analyze(task: string): Promise<TaskAnalysis>;

  selectAgent(
    task: TaskAnalysis,
    agents: AgentState[]
  ): Promise<AgentAssignment>;

  fallback(
    failedAssignment: AgentAssignment
  ): Promise<AgentAssignment>;

  learn(result: TaskResult): Promise<void>;
}
```

---

 # 23\. Example Configuration

```
orchestrator:
  mode: automatic

agents:

  codex:
    enabled: true
    priority: high
    responsibilities:
      - backend
      - database
      - api

  cursor:
    enabled: true
    priority: high
    responsibilities:
      - frontend
      - react
      - ui

  antigravity:
    enabled: true
    responsibilities:
      - research
      - ux

  claude:
    enabled: true
    responsibilities:
      - testing
      - review
      - architecture

permissions:
  mode: smart

  automatic:
    read_files: true
    write_files: true
    run_tests: true
    package_install: false
    git_push: false
    delete_files: false

scheduler:
  enabled: true
  parallel_tasks: true
  respect_file_locks: true
  respect_cooldowns: true
  use_project_history: true
```

---

 # 24\. Security Requirements

 Security is a first-class feature.

 The application must:

 - Never expose passwords or access tokens.
- Never bypass provider authentication.
- Never circumvent provider quotas or rate limits.
- Respect provider terms and API restrictions.
- Require explicit configuration for autonomous permissions.
- Provide an emergency **STOP ALL AGENTS** button.
- Log permission decisions.
- Log automated actions.
- Allow workspace-level restrictions.
- Prevent agents from modifying protected files without permission.
- Support read-only mode.
- Provide an audit log.

 Emergency control:

```
┌─────────────────────────────┐
│      🛑 STOP ALL AGENTS     │
│                             │
│ Stops every active session. │
└─────────────────────────────┘
```

---

 # 25\. Agent Selection Algorithm

 The initial scheduler can use a deterministic scoring model internally without exposing it as a "rating" to the user.

```
candidate agent
       │
       ├── capability match
       ├── responsibility match
       ├── availability
       ├── cooldown
       ├── quota availability
       ├── current workload
       ├── file ownership
       └── historical task context
                │
                ▼
          assignment decision
```

 Example:

```
const candidates = agents
  .filter(agent => agent.enabled)
  .filter(agent => agent.capabilities.includes(task.type))
  .filter(agent => !agent.cooldown.active)
  .filter(agent => !agent.quota.exhausted)
  .filter(agent => !hasFileConflict(agent, task));

const selected = scheduler.choose(candidates, task);
```

 The system should explain every automatic assignment:

```
Assigned to Codex because:

✓ Backend responsibility
✓ Available
✓ No file conflict
✓ Not in cooldown
✓ Required capabilities available

Fallback:
Claude
```

---

 # 26\. Automatic Mode

 When enabled:

```
User submits task
        ↓
Analyze keywords
        ↓
Analyze repository
        ↓
Check active agents
        ↓
Check responsibilities
        ↓
Check quota availability
        ↓
Check cooldown
        ↓
Check file ownership
        ↓
Select agent
        ↓
Launch
        ↓
Monitor
        ↓
Review result
        ↓
Run tests
        ↓
Request next agent if required
        ↓
Complete
```

---

 # 27\. Example End-to-End Workflow

 User:

```
"Build the first version of the payment dashboard."
```

 The orchestrator detects:

```
UI
Frontend
Backend
Database
Testing
```

 It creates:

```
TASK-001
├── UI
│   └── Cursor
│
├── Backend
│   └── Codex
│
├── Database
│   └── Codex
│
└── Testing
    └── Claude
```

 Execution:

```
Cursor ──────────────┐
                     │
Codex ───────────────┼──→ Integration
                     │
Claude ──────────────┘
                           │
                           ▼
                       Final review
```

 If Codex reaches a provider-enforced limit:

```
Codex
 ↓
Unavailable
 ↓
Scheduler checks fallback
 ↓
Claude
 ↓
Task continues
```

 The system records the event for future scheduling.

---

 # 28\. Observability

 The application should provide:

 ### Agent Timeline

```
09:01 Codex started
09:03 Codex edited auth.ts
09:05 Codex requested permission
09:06 Permission approved
09:12 Tests started
09:15 Tests passed
09:16 Task completed
```

 ### Usage History

```
Agent       Tasks     Runtime     Provider Usage*
Codex       21        4h 12m      Available
Cursor      18        3h 51m      Available
Claude      15        2h 48m      Available
```

 ### Audit Log

```
Timestamp
Agent
Action
Permission
Files
Result
```

---

 # 29\. Extensibility

 New agents should require only an adapter.

```
agents/
├── codex/
├── cursor/
├── claude/
├── antigravity/
├── copilot/
├── cline/
└── custom/
```

 Users should be able to configure:

```
customAgents:

  my-agent:
    command: my-agent
    capabilities:
      - frontend
      - backend
    permissionMode: manual
```

---

 # 30\. MVP

 The first release should focus on:

 1. VS Code extension
2. Agent registry
3. Codex adapter
4. Cursor adapter
5. Antigravity adapter
6. Claude adapter
7. Agent dashboard
8. Account switching
9. Real-time session monitoring
10. Manual permissions
11. Optional automatic permissions
12. Task routing
13. Agent responsibilities
14. File ownership
15. Basic task history
16. Cooldown manager
17. Manual/automatic scheduler
18. Emergency stop

 Quota monitoring should initially support only **provider-exposed quota/usage data**, with graceful "unknown" states where exact information is unavailable.

---

 # 31\. Future Versions

 ### V2

 - Advanced historical routing
- Automatic worktree creation
- Parallel agent execution
- Agent handoffs
- Test/review pipelines
- MCP server
- Advanced permission policies

 ### V3

 - Predictive workload scheduling
- Cross-project agent memory
- Team dashboards
- Remote agents
- Distributed workers
- Cost/usage analytics
- Advanced workflow designer

---

 # 32\. Product Principle

 The application should not attempt to replace the individual AI agents.

 Instead:

```
                    AGENT ORCHESTRATOR
                           │
          ┌────────────────┼────────────────┐
          │                │                │
       Codex             Cursor          Claude
          │                │                │
          └────────────────┼────────────────┘
                           │
                     Shared project
```

 The orchestrator is the **control plane**.

 The individual agents remain the **workers**.

 This makes the system easier to extend as new coding agents appear.

---

 # 33\. Success Criteria

 The application is successful when a developer can open VS Code, connect their available AI-agent accounts, and see:

```
┌─────────────────────────────────────────────┐
│             AI DEVELOPMENT TEAM             │
├─────────────────────────────────────────────┤
│                                             │
│ Codex        Backend        🟢 Working      │
│ Cursor       Frontend       🟢 Working      │
│ Claude       Testing        🟡 Waiting      │
│ Antigravity  Research       🟢 Working      │
│ Copilot      Documentation  ⚪ Idle         │
│                                             │
├─────────────────────────────────────────────┤
│ Scheduler: AUTO                             │
│ Permissions: SMART                          │
│ Quota Monitor: LIVE*                        │
│                                             │
│ Tasks: 7                                    │
│ Running: 3                                  │
│ Waiting: 2                                  │
│ Completed: 2                                │
└─────────────────────────────────────────────┘
```

 Then the developer can simply give the system a high-level objective, while the orchestrator determines **which compatible agent should perform each part, monitors execution, handles permissions according to policy, respects provider limits/cooldowns, records history, and coordinates handoffs**.

---

 ## 34\. Important Implementation Constraint

 "Real-time quota" must mean **real-time information actually available from each provider**. Different agents expose different usage information and may change their subscription, quota, CLI, and API behavior over time. The application should therefore use a provider adapter with a capability such as:

```
interface QuotaInfo {
  supported: boolean;
  remaining?: number;
  limit?: number;
  resetAt?: Date;
  unit?: "tokens" | "requests" | "credits" | "unknown";
  source: "provider_api" | "official_cli" | "user_configured" | "unknown";
  updatedAt: Date;
}
```

 Never manufacture a quota number when the provider does not expose one.

---

 ## 35\. Reference

 This design builds on the current direction of VS Code toward unified multi-agent development, third-party agent sessions, parallel subagents, custom agents, and MCP-based integrations.  Visual Studio Code+2

 The architecture is intentionally provider-neutral so that the project can continue adding agents without rebuilding the core orchestration system."# MultiAgentOchestra" 
