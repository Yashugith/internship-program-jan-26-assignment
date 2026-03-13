# Backend Developer

## **Evaluation Criteria**

* System design clarity (services, boundaries, async workflows)
* Schema design quality (entities, constraints, indexing, migrations order)
* Database + storage selection (why, tradeoffs, scaling path)
* Security (authN/authZ, secrets, data protection, safe downloads)
* Reliability (retries, idempotency, job state machine, observability)
* Cost + scalability thinking (MVP → v1)

## **Problem 1: Video-to-Notes Platform (Architecture + Schema)**

**Goal:** Upload video → async processing → outputs: transcript, **Summary.md**, highlights (timestamps), screenshot/clip references. [READ MORE ABOUT THE PROJECT](./Video-summary-platform.md)

* **System design:** API + worker(s) + storage + external AI/transcription boundaries
* **DB choice:** Postgres vs other (short justification)
* **Schema:** tables only (User, VideoAsset, Job, JobEvent, Artifact, Highlight)
* **Key constraints/indexes:** (unique keys, foreign keys, indexes you’d add)
* **Job lifecycle:** queued → processing → success/failed (+ retry/idempotency rule)
* **Storage layout:** how files/artifacts are stored + safe download strategy

**Your Solution for problem 1:**

System Design
The Video-to-Notes platform allows users to upload long videos and automatically generate structured outputs such as transcripts, summaries, highlights with timestamps, and screenshot references.
To ensure scalability and performance, the system uses asynchronous processing with worker services.

Architecture flow:
User → API Server → Object Storage → Job Queue → Worker → AI/Transcription Service → Database

Components:
1. API Server  
Handles user requests such as video upload, job status checking, and retrieving generated notes. It validates the upload and creates processing jobs.

3. Object Storage  
Video files and generated artifacts are stored in object storage such as AWS S3 or Google Cloud Storage. This avoids storing large files in the database.

5. Job Queue  
A queue system (Redis Queue / RabbitMQ) is used to handle background processing tasks and ensure reliable asynchronous execution.

7. Worker Services  
Workers consume jobs from the queue and perform processing tasks such as:
extracting audio
generating transcripts
summarizing text
detecting highlights
capturing screenshots

9. External AI / Transcription Services  
Workers call external services (such as Whisper or OpenAI APIs) to perform speech-to-text transcription and summarization.

Database Choice
PostgreSQL is used because it provides strong relational integrity, indexing support, and reliable transaction handling for structured metadata.
Large video files are stored in object storage, while only metadata and artifact references are stored in the database.

Database Schema
User  
- id (UUID, Primary Key)  
- email (Unique)  
- name  
- created_at  

VideoAsset  
- id (UUID, Primary Key)  
- user_id (Foreign Key → User.id)  
- file_url  
- duration  
- status (uploaded, processing, completed)  
- created_at  

Job  
- id (UUID, Primary Key)  
- video_id (Foreign Key → VideoAsset.id)  
- status (queued, processing, success, failed)  
- retry_count  
- created_at  
- completed_at  

JobEvent  
- id (UUID)  
- job_id (Foreign Key → Job.id)  
- event_type  
- message  
- created_at  

Artifact  
- id (UUID)  
- video_id (Foreign Key → VideoAsset.id)  
- artifact_type (transcript, summary, screenshot)  
- file_url  
- created_at  

Highlight  
- id (UUID)  
- video_id (Foreign Key → VideoAsset.id)  
- start_timestamp  
- end_timestamp  
- description  

Indexes are added on frequently queried fields such as user_id, video_id, and status.

Job Lifecycle
1. User uploads a video.
2. API stores the file in object storage.
3. Video metadata is saved in the VideoAsset table.
4. A processing job is created with status = queued.
5. The job is pushed to the queue.
6. A worker picks the job and updates status to processing.
7. Worker performs transcription, summary generation, highlight detection, and screenshot extraction.
8. Generated artifacts are stored in object storage.
9. Artifact references are saved in the database.
10. Job status is updated to success.
If processing fails, the system retries up to 3 times before marking the job as failed.

Storage Layout
Example structure in object storage:

videos/{user_id}/{video_id}.mp4
artifacts/{video_id}/transcript.txt  
artifacts/{video_id}/summary.md  
artifacts/{video_id}/screenshots/

Artifacts are delivered to users using signed URLs for secure downloads.

Security
- JWT authentication for API access  
- Signed URLs for secure file downloads  
- File type and size validation during upload  
- Rate limiting to prevent abuse

Reliability and Scalability
The platform is designed to scale horizontally by adding more worker instances. Queue-based processing ensures reliable job execution, and retry mechanisms prevent job loss. Object storage allows efficient handling of large video files while keeping the database lightweight.
*

## **Problem 2: LinkedIn Automation Platform (Backend Architecture)**

**Goal:** Connect LinkedIn → store persona → generate drafts (handled by GenAI team) → approve → schedule → auto-post + audit logs. [READ MORE ABOUT THE PROJECT](./linkedin-automation.md)

**Your solution must include**

* **System design:** OAuth flow, token storage/refresh, scheduler/worker design
* **Schema:** User, LinkedInAccount, Persona, Draft, Schedule, PostAttempt/PostLog
* **Security:** encrypt tokens at rest, least-privilege scopes, access control
* **Reliability:** retry posting, dedupe to prevent double-post, rate limiting
* **Prompt/config storage proposal:** how backend stores “prompt versions” or “config packs” provided by GenAI team (DB vs repo vs hybrid, versioning + rollback)

**Your Solution for problem 2:**

System Design
The LinkedIn Automation Platform allows users to connect their LinkedIn account, generate AI-assisted post drafts based on a persona, review the drafts, schedule posts, and automatically publish them while maintaining audit logs.

Architecture flow:
User → API Server → OAuth Service → Database → Job Scheduler → Worker → LinkedIn API

Components:
1. API Server  
Handles user actions such as connecting LinkedIn, managing personas, generating drafts, approving posts, and scheduling publishing tasks.

2. OAuth Integration  
Users connect their LinkedIn account using OAuth. Access tokens and refresh tokens are securely stored to allow scheduled posting.

3. Draft Generation Service  
When a user provides a topic, the backend calls the GenAI service which generates multiple post drafts based on the user's persona configuration.

4. Scheduler / Queue  
A scheduler service manages future publishing times. Jobs are pushed into a queue so workers can publish posts automatically.

5. Worker Services  
Workers process scheduled jobs, publish posts via LinkedIn API, and store results in audit logs.

Database Schema
User  
- id (UUID, Primary Key)  
- email  
- created_at  

LinkedInAccount  
- id (UUID)  
- user_id (Foreign Key → User.id)  
- access_token (encrypted)  
- refresh_token (encrypted)  
- expires_at  

Persona  
- id (UUID)  
- user_id (Foreign Key → User.id)  
- tone  
- audience  
- writing_style  

Draft  
- id (UUID)  
- user_id  
- persona_id  
- topic  
- content  
- status (generated, approved, scheduled)

Schedule  
- id (UUID)  
- draft_id  
- publish_time  
- status (pending, posted, failed)

PostLog / PostAttempt  
- id (UUID)  
- draft_id  
- status  
- linkedin_post_id  
- created_at

Indexes are added on user_id, publish_time, and status for efficient scheduling queries.
Security
- OAuth tokens are encrypted at rest.
- Access scopes follow the least-privilege principle.
- Only authorized users can access their LinkedIn accounts and drafts.

Reliability
- Scheduled posts are processed through a queue-based worker system.
- Retry logic prevents failed posts due to temporary API errors.
- Deduplication checks ensure the same post is not published twice.
- Rate limiting is applied to avoid LinkedIn API limits.

Prompt (Config Storage)
Prompt templates or configuration packs used by the GenAI team are stored in the database with version control.

Fields include:
- version
- prompt_template
- created_at
- status (active / deprecated)

This enables safe rollback if a prompt change introduces issues.

## **Problem 3: DOCX Template → Bulk DOCX/PDF Generator (Backend + Storage)**

**Goal:** Upload DOCX template → detect fields → single fill export → bulk fill via CSV/Sheet → ZIP + per-row report. [READ MORE ABOUT THE PROJECT](./docs-template-output-generation.md)

**Your solution must include**

* **System design:** template ingestion, field extraction service, bulk job worker, export service
* **Schema:** Template, TemplateVersion, TemplateField, BulkRun, BulkRow, Artifact, JobEvent
* **Bulk storage strategy:** where CSV input + generated docs + ZIP live, cleanup policy
* **Reliability:** partial success handling, per-row status, retries, resumable bulk run
* **Security:** template isolation per tenant/user, safe downloads, anti-path traversal

**Your Solution for problem 3:**

System Design
The DOCX Template → Bulk DOCX/PDF Generator allows users to upload a DOCX template, automatically detect editable fields, and generate documents either individually or in bulk using spreadsheet data.

Architecture flow:
User → API Server → Template Processing Service → Database → Job Queue → Worker → Storage

Components:
1. Template Ingestion Service  
When a DOCX file is uploaded, the system parses the document and extracts placeholders (e.g., {{name}}, {{date}}, {{amount}}). These fields are converted into a reusable template schema.

2. Field Detection (GenAI)  
GenAI services can assist in identifying possible editable fields and generating a structured field schema from the uploaded document.

3. Single Generation  
Users fill fields via a form interface. The system generates a single DOCX or PDF output.

4. Bulk Generation  
Users upload an Excel / Google Sheet. Each row represents one document instance. A background job generates documents for all rows.

5. Worker Services  
Workers process bulk generation jobs asynchronously to avoid blocking the API server.

Database Schema
Template  
- id (UUID, Primary Key)  
- user_id  
- template_name  
- created_at  

TemplateVersion  
- id (UUID)  
- template_id (Foreign Key → Template.id)  
- version_number  
- file_url  
- created_at  

TemplateField  
- id (UUID)  
- template_version_id  
- field_name  
- field_type  

BulkRun  
- id (UUID)  
- template_version_id  
- status (queued, processing, completed, failed)  
- created_at  

BulkRow  
- id (UUID)  
- bulk_run_id  
- row_data (JSON)  
- status (pending, success, failed)

Artifact  
- id (UUID)  
- bulk_row_id  
- file_url  
- created_at

JobEvent  
- id (UUID)  
- job_id  
- event_type  
- created_at

Indexes are added on template_id, bulk_run_id, and status.

Storage Strategy
Templates and generated documents are stored in object storage (AWS S3 / Cloud Storage).

Example layout:
templates/{template_id}/version_{n}.docx  
outputs/{bulk_run_id}/{row_id}.pdf  

Bulk outputs can optionally be packaged into a ZIP file for easy download.

Reliability
- Bulk generation runs as background jobs using queue workers.
- Each row has its own status so partial failures can be retried.
- Failed rows can be regenerated without restarting the entire job.

Security
- Templates and generated files are isolated per user or tenant.
- Signed URLs are used for secure downloads.
- Input validation prevents path traversal or malicious file uploads.


---

## **Problem 4: Character-Based Video Series Generator (Backend Architecture)**

**Goal:** Define characters once (image + traits + relationships). For each episode story → output episode package (script/scenes/assets plan/render plan), optionally render. [READ MORE ABOUT THE PROJECT](./char-based-video-generation.md)

**Your solution must include**

* **System design:** episodic pipeline as jobs, asset management, consistency strategy storage
* **Schema:** Character, Relationship, Episode, Scene, Asset, RenderJob, Artifact
* **Consistency data:** what gets persisted to keep character continuity across episodes
* **Storage:** images/audio/video assets, versioning, dedupe strategy
* **Security + cost controls:** quotas, rate limits, large asset constraints

**Your Solution for problem 4:**

System Design
The Character-Based Video Series Generator allows users to define characters once (including image, personality traits, and relationships) and then generate multiple short video episodes based on story prompts. Each episode produces a package containing script, scene breakdown, asset plan, and optionally a rendered video.

Architecture flow:
User → API Server → Story/Character Service → Job Queue → Worker → Media/GenAI Services → Storage → Database

Components:
1. Character Management  
Users define characters with attributes such as name, personality, appearance, and relationships. This data is reused across multiple episodes.

2. Episode Generation  
The user submits a story idea or scenario. The backend generates an episode plan including script, scene structure, and required assets using AI services.

3. Asset Generation  
Images, voice lines, or video clips are generated using external AI tools. These assets are stored and referenced for the episode.

4. Render Pipeline  
A render job combines scenes and assets to produce the final video (optional depending on workflow).

5. Worker Services  
Workers process generation and rendering tasks asynchronously to handle large workloads.

Database Schema
Character  
- id (UUID, Primary Key)  
- user_id  
- name  
- description  
- personality_traits  

Relationship  
- id (UUID)  
- character_id  
- related_character_id  
- relationship_type  

Episode  
- id (UUID)  
- user_id  
- title  
- story_prompt  
- status (queued, processing, completed)

Scene  
- id (UUID)  
- episode_id  
- scene_number  
- description  
- dialogue  

Asset  
- id (UUID)  
- episode_id  
- asset_type (image, audio, video)  
- file_url  

RenderJob  
- id (UUID)  
- episode_id  
- status (queued, rendering, completed, failed)

Artifact  
- id (UUID)  
- render_job_id  
- file_url

Indexes are added on episode_id, character_id, and status fields.

Consistency Strategy
Character definitions and relationships are stored persistently so that personality traits and relationships remain consistent across episodes. Each episode references the same character records.

Storage
Media assets such as images, audio, and generated video clips are stored in object storage (AWS S3 / Cloud Storage).

Example layout:
characters/{character_id}/images  
episodes/{episode_id}/assets  
renders/{episode_id}/final_video.mp4

Deduplication strategies can be used for reused assets.

Security and Cost Control
- API authentication ensures only authorized users generate episodes.
- Rate limits and quotas prevent excessive generation.
- File size limits and asset quotas help control infrastructure cost.


## Problem 5: Cross-Cutting

Answer briefly for the whole platform:

1. **Multi-tenancy:** user-level vs workspace-level, how you model it in schema
2. **AuthZ model:** RBAC or simple ownership rules, and where enforced (API + DB)
3. **Observability:** what you log for jobs + correlation ids + minimal metrics
4. **Data retention:** what to delete and when (inputs, artifacts, logs)
5. **Secrets & compliance:** token encryption, key management approach, PII handling

**Your Answer for problem 5:**

System Design
The Character-Based Video Series Generator allows users to define characters once (image, personality traits, relationships) and reuse them across multiple episodes. For each episode, the user provides a story prompt and the system generates a structured episode package including script, scene plan, assets, and optionally a rendered video.

Architecture flow:
User → API Server → Character & Episode Service → Job Queue → Worker → AI/Media Services → Storage → Database

Workers process generation tasks asynchronously to handle heavy media workloads.

Database Schema
Character  
- id (UUID)  
- user_id  
- name  
- traits  

Relationship  
- id (UUID)  
- character_id  
- related_character_id  
- relation_type  

Episode  
- id (UUID)  
- user_id  
- story_prompt  
- status  

Scene  
- id (UUID)  
- episode_id  
- scene_number  
- description  

Asset  
- id (UUID)  
- episode_id  
- asset_type (image, audio, video)  
- file_url  

RenderJob  
- id (UUID)  
- episode_id  
- status  

Artifact  
- id (UUID)  
- render_job_id  
- file_url  

Consistency Strategy
Character data and relationships are stored persistently so that all episodes reference the same character definitions, ensuring continuity across the video series.

Storage
Media assets and generated videos are stored in object storage.

Example layout:

episodes/{episode_id}/assets  
renders/{episode_id}/final_video.mp4

Security and Cost Control
- Authentication for API access  
- Rate limiting and user quotas  
- File size limits for media assets
