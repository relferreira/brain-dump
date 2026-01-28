---
name: brain-dump
description: Capture URLs, text, and images into personal knowledge base. Use when user says /brain-dump, "brain dump", "save this", "capture this URL".
allowed-tools: Bash, Read, Write, Glob, WebFetch
---

# Brain Dump Skill

Capture URLs, text, and images into a personal knowledge base stored as Markdown files in `~/.brain-dump/`.

## Commands

```
/brain-dump              → List recent dumps
/brain-dump <url>        → Fetch, summarize, save URL
/brain-dump <image-path> → Describe and save image
/brain-dump <text>       → Summarize (if long) and save text
```

## Instructions

### Step 1: Detect Input Type

Parse the arguments after `/brain-dump`:

1. **No arguments** → List mode
2. **Starts with `http://` or `https://`** → URL mode
3. **File path ending in `.png/.jpg/.jpeg/.gif/.webp/.svg` (and file exists)** → Image mode
4. **Anything else** → Text mode

### Step 2: Ensure Directory Exists

Before any write operation, ensure the directories exist:
```bash
mkdir -p ~/.brain-dump/assets
```

### Step 3: Process Based on Mode

---

#### LIST MODE (no arguments)

1. Use Glob to find all `*.md` files in `~/.brain-dump/`
2. For each file (up to 10 most recent):
   - Read the frontmatter to extract date, tags, type
   - Extract the title (first `# ` heading)
3. Display a formatted list:
   ```
   Recent dumps:

   1. [2024-01-28] Article Title (url) #tag1 #tag2
   2. [2024-01-27] Quick Note (text) #notes
   ...
   ```
4. If no dumps exist, say: "No dumps yet. Use `/brain-dump <url>`, `/brain-dump <text>`, or `/brain-dump <image-path>` to get started."

---

#### URL MODE (starts with http:// or https://)

1. **Fetch content** using WebFetch with prompt:
   ```
   Extract the main content of this page. Provide:
   1. A clear, descriptive title (not the site name)
   2. A 2-4 sentence summary of the key points
   3. 2-3 relevant tags as a comma-separated list (lowercase, single words)

   Format your response as:
   TITLE: [title]
   SUMMARY: [summary]
   TAGS: [tag1, tag2, tag3]
   ```

2. **Parse the response** to extract title, summary, and tags

3. **Generate filename**:
   - Take the title
   - Convert to lowercase
   - Replace spaces and special chars with hyphens
   - Remove consecutive hyphens
   - Truncate to 50 characters max
   - Add `.md` extension
   - If file exists, append date: `filename-20240128.md`
   - If still exists, append counter: `filename-20240128-2.md`

4. **Create markdown file**:
   ```markdown
   ---
   date: YYYY-MM-DD
   source: [original URL]
   tags: [tag1, tag2, tag3]
   type: url
   ---

   # [Title]

   [Summary]

   ---
   Source: [original URL]
   ```

5. **Write file** to `~/.brain-dump/[filename].md`

6. **Confirm to user**:
   ```
   Saved: [title]
   File: ~/.brain-dump/[filename].md
   Tags: #tag1 #tag2 #tag3
   ```

**Error handling**: If WebFetch fails, offer to save as a simple bookmark:
```markdown
---
date: YYYY-MM-DD
source: [URL]
tags: [bookmark]
type: bookmark
---

# Bookmark: [URL domain]

URL saved for later review.

---
Source: [URL]
```

---

#### IMAGE MODE (image file path)

1. **Validate file exists** using Bash `test -f`

2. **Generate unique asset filename**:
   - Use format: `YYYYMMDD-HHMMSS-[original-filename]`
   - Copy to `~/.brain-dump/assets/`

3. **Read and describe image** using the Read tool (which handles images)

4. **Generate content** based on the image:
   - Create a descriptive title
   - Write a 2-4 sentence description
   - Extract 2-3 relevant tags

5. **Generate markdown filename** from the description (same rules as URL mode)

6. **Create markdown file**:
   ```markdown
   ---
   date: YYYY-MM-DD
   source: [original file path]
   tags: [tag1, tag2, tag3]
   type: image
   ---

   # [Descriptive Title]

   ![image](assets/[asset-filename])

   [Description of the image]

   ---
   Original: [original file path]
   ```

7. **Write file** and **confirm to user**

**Error handling**: If file doesn't exist, ask user to verify the path.

---

#### TEXT MODE (anything else)

1. **Analyze the text**:
   - If less than 500 characters: save as-is, generate title and tags
   - If 500+ characters: generate a 2-4 sentence summary

2. **Generate title**:
   - If text has a clear subject, use it
   - Otherwise, extract key phrase or use first few words

3. **Generate 2-3 tags** relevant to the content

4. **Generate filename** (same rules as URL mode)

5. **Create markdown file**:

   For short text (< 500 chars):
   ```markdown
   ---
   date: YYYY-MM-DD
   tags: [tag1, tag2, tag3]
   type: note
   ---

   # [Title]

   [Original text]
   ```

   For long text (>= 500 chars):
   ```markdown
   ---
   date: YYYY-MM-DD
   tags: [tag1, tag2, tag3]
   type: note
   ---

   # [Title]

   [Summary]

   ---

   ## Original Content

   [Full original text]
   ```

6. **Write file** and **confirm to user**

---

### Filename Generation Helper

To generate a valid filename:

1. Take the source string (title, description, or first words)
2. Convert to lowercase
3. Replace any character that isn't a-z, 0-9, or hyphen with a hyphen
4. Replace multiple consecutive hyphens with a single hyphen
5. Remove leading/trailing hyphens
6. Truncate to 50 characters (don't cut mid-word if possible)
7. Check if file exists in `~/.brain-dump/`:
   - If yes, append today's date: `name-20240128.md`
   - If that exists too, append counter: `name-20240128-2.md`, `name-20240128-3.md`, etc.

### Response Format

Always be concise. After successful save:
```
Saved: [title]
File: ~/.brain-dump/[filename].md
Tags: #tag1 #tag2 #tag3
```

For list mode, show a clean formatted list. For errors, be helpful and suggest fixes.
