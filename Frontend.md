# **Frontend Engineer Assignment (No Code, High-Quality UI Focus)**

## **Evaluation Criteria**

* Frontend tech selection and reasoning (framework, state, data fetching)
* UI architecture (routing, component design, reusable patterns)
* API calling strategy (error handling, retries, abort, pagination)
* Browser-level caching + offline-friendly patterns
* Debugging + observability (logging, tracing, error boundaries)
* Security basics on client (token handling, safe downloads, XSS considerations)
* UX quality for async jobs (progress, partial results, resilience)

---

## **Problem 1: Video-to-Notes Platform (Frontend System Design)**

**Goal:** Upload video → job runs async → user sees status + outputs: Summary.md, highlights (timestamps), assets. [READ MORE ABOUT THE PROJECT](./Video-summary-platform.md)

**Your solution must include**

* **Screens:** Upload, Jobs list, Job detail (status/logs), Results (markdown + highlights)
* **UI states:** loading, queued, processing, success, failed, retry, partial output
* **API calling plan:** how you poll/stream job progress (polling vs SSE), abort on navigation
* **Caching:** what to cache in browser (job list, job detail, results), TTL strategy, invalidation
* **Debugging plan:** how you would debug “stuck processing” from frontend side (network logs, correlation id display)

**Your Solution for problem 1:**

Frontend System Design
Tech Stack
- Framework:React with Next.js for fast routing and SSR support.
- State Management:React Query for server state (job status, results) and local state with React hooks.
- UI Library:TailwindCSS + reusable component library for consistent design.

Screens
1. Upload Page
   - Video upload with progress indicator.
   - Shows validation errors (size/type).

2. Job List
   - Displays all processing jobs.
   - Shows status badges (Queued, Processing, Success, Failed).

3. Job Detail
   - Live status updates with logs and progress.

4. Results Page
   - Displays generated Summary.md
   - Highlights with clickable timestamps
   - Screenshot assets.

UI States
- Loading → spinner
- Queued → waiting indicator
- Processing → progress bar
- Success → results view
- Failed → retry button
- Partial results → show completed artifacts first.

API Strategy
- Upload via POST /videos
- Poll /jobs/{id} every few seconds for progress.
- Abort polling when user leaves page.

Browser Caching
- Cache job list and job details using React Query.
- TTL caching for results data.
- Invalidate cache after job completion.
Debugging Plan
- Display job ID and correlation ID in UI.
- Log network requests in browser console.
- If job stuck in processing, inspect job status API responses.


---

## **Problem 2: LinkedIn Automation Platform (Frontend System Design)**

**Goal:** Connect LinkedIn → persona setup → draft preview → approve → schedule → posting history. [READ MORE ABOUT THE PROJECT](./linkedin-automation.md)

**Your solution must include**

* **Screens:** Connect, Persona editor, Drafts (3 variants), Approval, Scheduler, Post history
* **Form UX:** persona inputs validation, topic input rules, guardrails for scheduling
* **API calling:** draft generation request lifecycle, optimistic UI vs strict confirmation
* **Caching:** drafts caching, schedule list caching, refetch triggers after approval/post
* **Debugging:** how you surface posting failures to user and capture details for support

**Your Solution for problem 2:**

LinkedIn Automation Platform – Frontend Design

Screens
1. Connect LinkedIn
   - OAuth connection flow.
2. Persona Editor
   - Inputs for tone, audience, writing style.
3. Draft Generator
   - Shows 3 generated drafts.
4. Approval Screen
   - User selects or edits draft.
5. Scheduler
   - Select date/time for posting.
6. Post History
   - Shows published posts and status.

Form UX
- Validate persona inputs before generation.
- Show character limit for LinkedIn posts.
- Prevent scheduling posts in the past.

API Calling Strategy
- POST /generate-draft
- POST /approve
- POST /schedule
- Use optimistic UI updates when approving drafts.

Caching
- Cache drafts to avoid regeneration.
- Cache schedule list and post history.
- Refetch data after publishing.

Debugging
- Display error messages for failed LinkedIn posting.
- Show request ID to help backend debugging.

---

## **Problem 3: DOCX Template → Bulk Generator (Frontend System Design)**

**Goal:** Upload template → review fields → single generate → bulk via CSV → ZIP download + per-row report. [READ MORE ABOUT THE PROJECT](./docs-template-output-generation.md)

**Your solution must include**

* **Screens:** Template upload, Field review/editor, Single fill form, Bulk upload, Bulk run status, Report table, Downloads
* **Field UI:** field types (text/number/date), required/default, inline validation
* **Bulk UX:** CSV upload constraints, mapping UI (optional), progress + partial success
* **Browser caching:** template metadata caching, field schema caching, bulk report pagination caching
* **Downloads:** safe download UX (signed URL flow assumed), progress indicator

**Your Solution for problem 3:**

DOCX Template → Bulk Generator (Frontend Design)

Screens
1. Template Upload
2. Field Detection Review
3. Single Fill Form
4. Bulk CSV Upload
5. Bulk Run Status
6. Download Results

Field UI
- Editable fields detected from template.
- Field types supported: text, number, date.
- Validation rules (required fields, format validation).

Bulk Upload UX
- CSV validation before upload.
- Show mapping preview between CSV columns and template fields.
- Progress bar for generation status.

Browser Caching
- Cache template metadata and field schema.
- Cache bulk run results for pagination.

Downloads
- Use signed URLs for secure downloads.
- Show download progress indicator.


---

## **Problem 4: Character-Based Video Series Generator (Frontend System Design)**

**Goal:** Define characters once → create episode from story → view episode package (script/scenes/assets/render plan). [READ MORE ABOUT THE PROJECT](./char-based-video-generation.md)

**Your solution must include**

* **Screens:** Character library, Relationship editor, Episode creator, Episode detail (scenes), Asset gallery
* **Consistency UX:** show “locked character profile” per episode, version badges
* **API calling:** long-running generation job UI (progress, resume)
* **Caching:** character library caching, episode package caching, asset thumbnails caching

**Your Solution for problem 4:**

Character-Based Video Series Generator (Frontend Design)

Screens
1. Character Library
2. Relationship Editor
3. Episode Creator
4. Episode Detail (Scenes)
5. Asset Gallery

Consistency UX
- Character profiles locked per episode to maintain story consistency.
- Version badges show character updates.

API Calling
- Episode generation runs as async job.
- UI shows job progress and status updates.

Caching
- Cache character library data.
- Cache episode packages and asset thumbnails.


---

## **Cross-Cutting** 

Answer these in **bullet points** (max 1 page total):

1. **Frontend stack choice**

  - Framework: Next.js (React)
  - State: React Query
  - Router: Next.js routing
  - UI Kit: TailwindCSS
  - Chosen for performance, modular components, and good developer experience.

2. **API layer design**

* Fetch/Axios choice, typed client generation (OpenAPI), error normalization, retries, request dedupe, abort controllers.
  - Use Axios for API requests.
  - Central API client with error normalization.
  - Implement retry logic and abort controllers for cancelled requests.

3. **Browser caching plan**

* What you cache (GET responses, derived state), where (memory, IndexedDB, localStorage), TTL/invalidation rules.
* How you handle “job status updates” without stale UI.
  `
  - Cache GET responses with React Query.
  - Use memory cache for job status.
  - Use IndexedDB or localStorage for lightweight persistence.

4. **Debugging & observability**

* Error boundaries, client-side logging approach, correlation id propagation, “report a problem” payload.
* How you would debug: slow uploads, failed downloads, intermittent 500s.
  - Implement error boundaries in React.
  - Log API failures to monitoring service.
  - Display correlation IDs for debugging support issues.

5. **Security basics**

* Token storage approach, CSRF considerations (if cookies), XSS avoidance for markdown rendering, safe file download patterns.
  - Store tokens in secure HTTP-only cookies when possible.
  - Sanitize markdown rendering to prevent XSS.
  - Use signed URLs for secure file downloads.
