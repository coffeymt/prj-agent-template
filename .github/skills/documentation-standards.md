---
name: Documentation Standards
description: 'Defines how workspace documentation in assets/ and docs/ is written, structured, and maintained.'
applyTo: "assets/**/*.md, docs/**/*.md, **/README.md"
---

# Documentation Standards Skill

## Overview
Use this skill when creating or updating workspace documentation, especially under `assets/`.
Prioritize clarity, scanability, and maintenance over narrative writing.

## Assets Directory Structure
- Use top-level files in `assets/` for major topics such as architecture, onboarding, and runbooks.
- Keep each document self-contained and focused on exactly one topic.
- Prefer a flat structure with descriptive filenames over deep subdirectory nesting.
- Use explicit names that reveal scope (example: `CLIENT_ONBOARDING_RUNBOOK.md`).

## Writing Standards
- Be concise. Every sentence must add new, useful information.
- Lead with the most important information first (inverted pyramid).
- Prefer bullet points and tables for structured content.
- Use concrete examples instead of abstract descriptions.
- Remove filler phrases such as `In order to`, `It should be noted that`, and `As previously mentioned`.
- Avoid redundancy. If guidance exists elsewhere, link to it instead of repeating it.

## Document Section Structure
- Start with one H1 title that states what the document covers.
- Add an `Overview` section with 1 to 3 sentences of essential context.
- Organize body content as H2 sections by topic, not by timeline.
- Write each section so it can be read independently.

## Update Rules
- Update the relevant existing section in place; do not append updates to the bottom.
- Remove stale content instead of marking it as deprecated.
- Preserve existing section order unless reordering clearly improves clarity.
- Add cross-references to related docs using relative links.

## Anti-Patterns
- Do not add "living document" disclaimers.
- Do not add changelog sections inside docs; rely on git history.
- Do not commit TODOs, placeholders, or incomplete stubs.
- Do not use screenshots for textual instructions; use real text and code blocks.

## Review Checklist
- Scope is single-topic and self-contained.
- Overview is 1 to 3 sentences and captures essentials.
- Sections are topical H2s and independently readable.
- Redundant or stale content has been removed.
- Cross-references use relative links.
- No anti-patterns are present.
