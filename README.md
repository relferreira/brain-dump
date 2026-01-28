# brain-dump

A Claude Code skill for capturing URLs, text, and images into a personal knowledge base.

## Overview

Brain Dump is a simple way to save things you find interesting. When you come across a useful article, have a quick thought, or want to save an image for later, just use `/brain-dump` and it gets stored as a searchable Markdown file.

## Installation

1. Clone this repository
2. Add the skill to Claude Code:
   ```bash
   claude skill add /path/to/brain-dump
   ```

## Usage

```
/brain-dump                    # List your recent dumps
/brain-dump <url>              # Save and summarize a webpage
/brain-dump <image-path>       # Describe and save an image
/brain-dump <text>             # Save a note or thought
```

### Examples

**Save a URL:**
```
/brain-dump https://react.dev/learn/thinking-in-react
```
Fetches the page, generates a summary, extracts tags, and saves it.

**Save a quick note:**
```
/brain-dump Remember to check out that new testing library Sarah mentioned
```

**Save an image:**
```
/brain-dump ~/Screenshots/architecture-diagram.png
```
Copies the image, generates a description, and saves it with the reference.

**List recent dumps:**
```
/brain-dump
```
Shows your 10 most recent saves with dates and tags.

## How It Works

1. **Input Detection** - Automatically detects if you're saving a URL, image, or text
2. **AI Processing** - Generates titles, summaries, and relevant tags
3. **Markdown Storage** - Saves everything as clean Markdown with YAML frontmatter
4. **Smart Filenames** - Creates readable, kebab-case filenames from content

## Storage

All dumps are stored in `~/.brain-dump/`:

```
~/.brain-dump/
├── thinking-in-react-guide.md
├── testing-library-note.md
├── architecture-diagram.md
└── assets/
    └── 20240128-143022-architecture-diagram.png
```

### File Format

Each dump is a Markdown file with frontmatter:

```markdown
---
date: 2024-01-28
source: https://react.dev/learn/thinking-in-react
tags: [react, tutorial, components]
type: url
---

# Thinking in React Guide

A step-by-step guide to building React applications by breaking
UI into components, building a static version first, and then
adding interactivity through state management.

---
Source: https://react.dev/learn/thinking-in-react
```

## Types

| Type | Description |
|------|-------------|
| `url` | Webpage content with summary |
| `note` | Text notes (short or summarized) |
| `image` | Image with AI-generated description |
| `bookmark` | Failed URL fetch, saved for later |

## Requirements

- Claude Code CLI
- The skill uses these tools: `Bash`, `Read`, `Write`, `Glob`, `WebFetch`
