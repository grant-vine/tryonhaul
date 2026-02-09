# Documentation Standards

> **Last Updated**: 2026-02-09  
> **Related**: [AGENTS.md](../../AGENTS.md) | [INDEX.md](../../INDEX.md)

---

## Purpose

This document defines how to create, organize, and maintain documentation for Try on Haul.

---

## Core Principles

### 1. Single Source of Truth

- **Each piece of information has ONE canonical location**
- Reference other documents with links, don't duplicate
- If you find yourself copying content, create a link instead

### 2. Keep Docs Current

- Update docs when making related code changes
- Delete docs that are no longer relevant
- Mark outdated sections explicitly

### 3. Write for Agents and Humans

- Clear, structured format
- Machine-parseable (Markdown)
- Human-readable prose

---

## File Naming

### Documentation Files

**UPPERCASE-KEBAB-CASE** with `.md` extension:

```
MVP-FEATURES.md
TECH-STACK.md
API-REFERENCE.md
```

### Code Files

**lowercase-kebab-case**:

```
user-service.ts
image-uploader.tsx
api-client.ts
```

### Directories

**lowercase-kebab-case**:

```
docs/
├── architecture/
├── user-stories/
├── research/
├── standards/
└── plans/
```

---

## Document Structure

Every documentation file must have:

### 1. Title (H1)

Clear, descriptive title matching the filename:

```markdown
# Tech Stack Decisions
```

### 2. Metadata Block

```markdown
> **Last Updated**: YYYY-MM-DD  
> **Related**: [Link1](./path) | [Link2](./path)
```

### 3. Purpose Statement

Brief description of what this document covers:

```markdown
## Purpose

This document describes the technology decisions for Try on Haul with rationale.
```

### 4. Content Sections (H2+)

Organized with clear headings.

### 5. Change Log (at end)

```markdown
## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial document | AI Agent |
```

---

## Document Template

```markdown
# Document Title

> **Last Updated**: YYYY-MM-DD  
> **Related**: [Related Doc 1](./path) | [Related Doc 2](./path)

---

## Purpose

Brief description of this document's purpose and scope.

---

## Section 1

Content...

---

## Section 2

Content...

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| YYYY-MM-DD | Initial document | Author Name |
```

---

## Formatting Guidelines

### Headings

- **H1 (#)**: Document title only (one per file)
- **H2 (##)**: Major sections
- **H3 (###)**: Subsections
- **H4 (####)**: Use sparingly, prefer prose

### Lists

**Use bullet lists for**:
- Unordered items
- Feature lists
- Quick references

**Use numbered lists for**:
1. Sequential steps
2. Ranked items
3. Processes with order

### Tables

Use tables for structured data:

```markdown
| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Data     | Data     | Data     |
```

### Code Blocks

Always specify language:

````markdown
```typescript
function example(): void {
  console.log('Hello');
}
```
````

### Inline Code

Use for:
- File names: `package.json`
- Function names: `generateTryOn()`
- Variables: `userId`
- Commands: `npm install`

### Links

**Relative links** for internal docs:

```markdown
[Tech Stack](./TECH-STACK.md)
[Parent Directory](../architecture/OVERVIEW.md)
```

**Absolute URLs** for external resources:

```markdown
[Next.js Docs](https://nextjs.org/docs)
```

---

## Cross-Referencing

### When to Reference vs. Duplicate

| Situation | Action |
|-----------|--------|
| Full details exist elsewhere | Link: "See [Document](./path)" |
| Summary needed for context | Brief summary + link for details |
| Information is foundational | Can repeat key points + link |

### Reference Patterns

**Full reference**:
```markdown
> **Full details**: [TECH-STACK.md](../architecture/TECH-STACK.md)
```

**Inline reference**:
```markdown
As defined in [MVP Features](./MVP-FEATURES.md), users can generate up to 5 images per day.
```

**Summary with reference**:
```markdown
We use CatVTON for AI generation due to its accuracy and speed.

> See [AI Models Research](../research/AI-MODELS.md) for full comparison.
```

---

## Writing Style

### Tone

- **Concise**: No unnecessary words
- **Direct**: State things clearly
- **Professional**: Avoid slang
- **Active voice**: "The system validates..." not "Validation is performed by..."

### Technical Terms

- Define acronyms on first use: "Progressive Web App (PWA)"
- Link to glossary for complex terms
- Avoid jargon when simpler words work

### Examples

Always include examples for:
- API requests/responses
- Configuration files
- Code patterns
- Complex processes

---

## Maintaining Documentation

### When to Update

- **Code changes**: Update related docs in same PR
- **Architecture changes**: Update overview docs first
- **New features**: Add to MVP features, requirements
- **Deprecations**: Mark as deprecated, link to replacement

### Marking Outdated Content

```markdown
> **TODO**: Update this section to reflect new auth flow
```

Or for larger sections:

```markdown
> **⚠️ OUTDATED**: This section describes the old implementation.
> See [New Feature Name](./path/to/new-doc.md) for current approach.
```

### Deleting Documentation

When content is no longer relevant:
1. Check for incoming links
2. Update or remove links
3. Delete the file
4. Update INDEX.md

---

## Document Categories

### Architecture (`/docs/architecture/`)

System design, technology decisions, data flow:
- OVERVIEW.md
- TECH-STACK.md
- DATA-FLOW.md

### User Stories (`/docs/user-stories/`)

Features, requirements, acceptance criteria:
- MVP-FEATURES.md
- REQUIREMENTS.md

### Research (`/docs/research/`)

Investigation, comparison, findings:
- AI-MODELS.md
- PROMPT-ENGINEERING.md

### Standards (`/docs/standards/`)

Guidelines, conventions, rules:
- DEV-STANDARDS.md
- DESIGN-STANDARDS.md
- DOC-STANDARDS.md

### Plans (`/docs/plans/`)

Roadmaps, implementation plans:
- IMPLEMENTATION-PLAN.md

---

## Quality Checklist

Before committing documentation:

- [ ] Title matches filename
- [ ] Metadata block present (Last Updated, Related)
- [ ] Purpose statement included
- [ ] All internal links work
- [ ] Code blocks have language specified
- [ ] Tables are properly formatted
- [ ] No duplicate information (links used instead)
- [ ] Change log updated
- [ ] INDEX.md updated (if new file)

---

## Common Mistakes

### ❌ Avoid

- Duplicating content across files
- Leaving broken links
- Missing language in code blocks
- Inconsistent heading levels
- Walls of text without structure
- Outdated content without warning

### ✅ Do

- Link to canonical sources
- Test all links before commit
- Always specify code language
- Use consistent heading hierarchy
- Break content with lists, tables, sections
- Mark outdated content clearly

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2026-02-09 | Initial documentation standards | AI Agent |
