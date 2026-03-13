# GenAI Assignment:

**Evaluation Criteria**

We will score your submission on:

* Clarity and practicality of architecture
* Robust JSON schema design
* Prompt quality (zero-shot, reliable, minimal hallucination risk)
* Handling of ambiguity + user review flow
* Bulk generation thinking (errors, naming, report)

## Problem 1: **Proposal for “Video-to-Notes”**

We have a local folder of long videos (3–4 hours each, 200MB+). Watching them fully is slow. We need an automated way to generate a “summary package” per video: **Summary.md** + highlight clips + screenshots, all organized per video. [READ MORE ABOUT THE PROJECT](./Video-summary-platform.md)

### **Task**

Prepare a **pre-processed solution proposal** comparing  **three approaches** **:**

1. **Online/Cloud-Based (Already Available Solutions)**
2. **Build Our Own Using LLM APIs (Hybrid: local media processing + cloud LLM)**
3. **Build Fully Offline Using Open-Source Models (Local transcription + local LLM + pipeline)**

No code required. We want a **clear, practical proposal** with architecture and tradeoffs.

### Your Solution for problem 1:

Proposal: Video-to-Notes Platform

Idea
The Video-to-Notes platform converts long educational videos into structured learning artifacts such as transcripts, summaries, highlights, and screenshots. The goal is to help users quickly understand key ideas without watching the entire video.

Workflow
1. User uploads a video.
2. The system extracts audio from the video.
3. A speech-to-text model generates a transcript.
4. The transcript is analyzed by a language model to produce:
   - concise summary
   - key highlights with timestamps
   - topic segmentation
5. Important frames are extracted as screenshots.

AI Models Used
- Speech-to-Text:Whisper or similar ASR model  
- Text Summarization: Large Language Model (LLM)  
- Highlight Detection: LLM + timestamp mapping

Output
The user receives a structured notes package containing:
- Full transcript
- Short summary
- Highlight timestamps
- Screenshots referencing key moments

This helps students and professionals review long videos quickly and efficiently.

## Problem 2: **Zero-Shot Prompt to generate 3 LinkedIn Post**

Design a **single zero-shot prompt** that takes a user’s persona configuration + a topic and generates **3 LinkedIn post drafts** in **3 distinct styles**, each aligned to the user’s voice and constraints. The output must be structured so the app can: show 3 drafts to the user. Assume we are consuming **OpenAI API / Gemini API** with **one prompt call** (no fine-tuning). Your prompt must reliably produce valid, structured output. [READ MORE ABOUT THE PROJECT](./linkedin-automation.md)

**TASK:** Write a prompt that can work.

### Your Solution for problem 2:

Prompt Design for LinkedIn Post Generation

Prompt
You are a professional LinkedIn content writer.

Generate 3 LinkedIn post drafts based on the following input.

Topic: {topic}
User Persona:
- Tone: professional but conversational
- Audience: technology professionals
- Style: insightful and engaging

Each post should:
- be between 120–180 words
- start with a strong hook
- include a clear idea or insight
- end with a question or call to action
- avoid emojis and excessive hashtags

Return the results as three separate drafts.

Safety
The system should filter outputs to prevent:
- spam-like promotional content
- misleading claims
- inappropriate or offensive language


## Problem 3: **Smart DOCX Template → Bulk DOCX/PDF Generator (Proposal + Prompt)**

Users have many Word documents that act like templates (offer letters, certificates, invoices, contracts). They repeatedly change only a few fields (name, date, amount, address, role, etc.). Doing this manually is slow and error-prone, especially in bulk. [READ MORE ABOUT THE PROJECT](./docs-template-output-generation.md)

We want a system that:

1. Converts an uploaded **DOCX** into a reusable **template** by identifying editable fields.
2. Supports **single generation** (form-fill → DOCX/PDF download).
3. Supports **bulk generation** via **Excel/Google Sheet** rows.

### **Task (No coding)**

Submit a **proposal** for building this system using GenAI (OpenAI/Gemini) for “template field detection” and “field schema generation”. We want a practical design, not code.

### Your Solution for problem 3:

Smart DOCX Template Field Detection

Idea
When a user uploads a DOCX template, the system automatically detects editable placeholders and converts them into structured fields that can be filled through a form or spreadsheet.

Approach
1. Parse the DOCX document and extract text blocks.
2. Detect placeholders such as:
   - {{name}}
   - {{date}}
   - {{amount}}
3. Use a language model to identify potential variable fields that are not explicitly marked.
4. Convert detected fields into a structured schema.

Output
Example detected fields:
- name
- address
- date
- invoice_amount

These fields are presented to the user for confirmation and editing.
This approach allows the platform to transform static templates into dynamic document generation workflows.


## Problem 4: Architecture Proposal for 5-Min Character Video Series Generator

We want to build a system that helps a user create a short video series (around **5 minutes per episode**) using predefined characters. The user defines characters (image + personality) and relationships once, then provides a short story/situation for an episode. The system outputs a complete episode package and optionally a final video. [READ MORE ABOUT THE PROJECT](./char-based-video-generation.md)

### **Task**

Create a **small, clear architecture proposal** (no code, no prompts) describing how you would design and build this system.

### Your Solution for problem 4:

Character-Based Video Series Generator

Idea
The platform allows users to define characters once and generate multiple episodes using those characters. Each episode is created from a story prompt.

Workflow
1. User defines characters with attributes:
   - name
   - personality
   - relationships
2. User submits an episode prompt.
3. A language model generates:
   - episode storyline
   - scene breakdown
   - character dialogue
4. Image or video generation models produce visual assets for each scene.

Output
Each episode produces a package containing:
- script
- scene plan
- character dialogue
- generated visual assets

This enables scalable creation of story-based video content using reusable characters.
