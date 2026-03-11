# Plan: AI-Powered Client Scoping Tool

> Source PRD: https://github.com/auditmos/doit/issues/1

## Architectural decisions

- **Architecture style**: Monorepo (auditmos/saas-on-cf template) — TanStack Start frontend + Hono API backend on Cloudflare Workers, Drizzle ORM + Neon Postgres
- **Data model**: Project → Messages, Spec, Tasks. Project identified by unique slug for public access.
- **Key entities**: Project, Message, Spec, Task
- **Auth**: Better Auth for admin only. Client pages are fully public (slug = access).
- **AI**: Claude API called from data-service worker. System prompts encode the 4-phase interview methodology (discovery → PRD → plan → tasks). No tool use.
- **GitHub integration**: Personal access token as Cloudflare Worker secret. Polls issue status on page load.
- **Domain**: *.auditmos.com (subdomain TBD)

---

## Phase 1: Project Creation & Client Chat

**User stories**: 1, 2, 3, 16, 17, 18

### What to build

Admin creates a new project via the dashboard, which generates a unique slug/link. Client opens the link in a browser — no login — and lands on a chat interface. Messages are sent to Claude API with system prompts that initiate the discovery interview. All messages (user + assistant) are persisted to Neon Postgres. When the client returns to the same link later, the full conversation history loads and the AI resumes where it left off.

Admin can view the conversation history for any project from the dashboard.

### Acceptance criteria

- [ ] Admin can create a new project and receive a unique shareable link
- [ ] Client opens the link and sees a chat interface with no login required
- [ ] Client sends a message and receives an AI response following the discovery interview structure
- [ ] All messages persist to the database
- [ ] Client closes browser, reopens the link, and sees full conversation history
- [ ] AI resumes the interview from where it left off
- [ ] Admin can view conversation history per project in the dashboard

---

## Phase 2: AI Phase Transitions & Spec Generation

**User stories**: 4, 19, 20, 21, 22, 23

### What to build

Extend the AI system prompts so Claude auto-transitions through four phases: discovery → requirements (PRD) → implementation plan → task breakdown. The AI signals each transition clearly to the client in plain language. When the final phase completes, the AI outputs a structured spec (markdown) and a task list. The system parses this output and stores the spec and individual tasks in the database, linked to the project. The project status updates to "review."

Handle long conversations by implementing a summarization strategy when approaching context limits — summarize earlier phases and inject the summary into the system prompt.

### Acceptance criteria

- [ ] AI transitions from discovery to requirements phase automatically when discovery is complete
- [ ] AI transitions from requirements to plan to task breakdown automatically
- [ ] Each transition is clearly communicated to the client in non-technical language
- [ ] Final AI output contains a structured markdown spec and a numbered task list
- [ ] Spec and tasks are parsed and stored in the database
- [ ] Project status updates to "review" when generation completes
- [ ] Conversations exceeding context limits are handled via summarization

---

## Phase 3: Admin Spec Review & Editing

**User stories**: 10, 11, 12

### What to build

Admin dashboard shows a list of all projects with their current status (interviewing, review, active, complete). Clicking a project in "review" status opens the generated spec in a markdown editor. Admin can edit the spec and the task list (add, remove, rename, reorder tasks). Admin approves/finalizes the spec, which updates the project status to "active" and makes the spec visible to the client.

### Acceptance criteria

- [ ] Dashboard lists all projects with status indicators
- [ ] Admin can open a project and see the generated spec in a markdown editor
- [ ] Admin can edit and save the spec
- [ ] Admin can edit the task list (add, remove, rename, reorder)
- [ ] Admin can finalize/approve the spec
- [ ] Project status transitions from "review" to "active" on approval
- [ ] Finalized spec and tasks become visible on the client page

---

## Phase 4: Client Spec & Task View

**User stories**: 5, 6, 7, 8, 9

### What to build

When a project is finalized (status "active" or "complete"), the client's link shows a spec & tracking view instead of the chat. The page renders the approved spec as formatted markdown at the top. Below it, a task list displays each task with a plain-language name and simple status indicator (done / not done). No login required — the link remains the only access method.

### Acceptance criteria

- [ ] Client link shows spec & tracking view when project is active/complete
- [ ] Spec renders correctly from markdown with readable formatting
- [ ] Task list displays with plain-language names and done/not-done status
- [ ] Page loads without any authentication
- [ ] Chat view is no longer accessible once spec is finalized (or is accessible as a secondary view)

---

## Phase 5: GitHub Sync

**User stories**: 13, 14, 15

### What to build

Admin selects a GitHub repo from their available repos (fetched via GitHub API using the stored personal access token). A "push to GitHub" action creates one GitHub issue per task in the selected repo, with the task title and description as issue title and body. The GitHub issue number and URL are stored against each task in the database. On the client tracking page, when the page loads, the system polls GitHub API for the current status of each linked issue and displays updated done/not-done status.

### Acceptance criteria

- [ ] Admin can select a GitHub repo from a list of available repos
- [ ] "Push to GitHub" creates one issue per task in the selected repo
- [ ] GitHub issue numbers and URLs are stored in the database per task
- [ ] Client tracking page polls GitHub API on load for issue status
- [ ] Task status on the client page reflects whether the GitHub issue is open or closed
- [ ] Admin can see GitHub issue links in the dashboard
