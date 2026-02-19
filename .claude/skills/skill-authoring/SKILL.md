---
name: skill-authoring
description: Comprehensive guide for authoring effective Claude agent skills. Covers core principles (conciseness, appropriate degrees of freedom, testing), skill structure (naming, descriptions, frontmatter), progressive disclosure patterns, workflows, content guidelines, common patterns, evaluation methods, anti-patterns, runtime environment, and quality checklists. Use when creating or improving skills, determining when skills should trigger, writing skill descriptions, organizing skill content, choosing appropriate levels of specificity, and validating completed skills.
---

# Skill Authoring Best Practices

Guide for writing effective skills that Claude can discover and use successfully.

## Core Principles

### Concise is Key

The context window is a shared resource. Skills compete with system prompt, conversation history, other skills' metadata, and user requests for space.

**Design principle:** Claude is already very smart. Only add context Claude doesn't already have. Challenge every piece of information:
- Does Claude really need this explanation?
- Can I assume Claude knows this?
- Does this paragraph justify its token cost?

**Strategy**: Use concise examples over verbose explanations. Assume knowledge of common concepts.

### Set Appropriate Degrees of Freedom

Match specificity to the task's fragility and variability:

| Level | Use When | Style | Example |
|-------|----------|-------|---------|
| **High** | Multiple valid approaches, context-dependent | Text-based instructions | Code review checklist with general principles |
| **Medium** | Preferred pattern exists, variation is acceptable | Pseudocode/scripts with parameters | Template with customizable sections |
| **Low** | Fragile/error-prone, consistency critical, specific sequence required | Specific scripts, few parameters | Exact database migration command |

**Analogy:** Claude is a robot exploring a path:
- **Narrow bridge with cliffs**: One safe way forward → provide specific guardrails (low freedom)
- **Open field with no hazards**: Many paths succeed → give direction and trust Claude (high freedom)

### Test with All Models

Skills effectiveness depends on the underlying model. Test with:
- **Claude Haiku** (fast, economical): Does the skill provide enough guidance?
- **Claude Sonnet** (balanced): Is it clear and efficient?
- **Claude Opus** (powerful reasoning): Does it avoid over-explaining?

What works perfectly for Opus may need more detail for Haiku.

## Skill Structure

### Naming Conventions

**Format requirements:**
- Maximum 64 characters
- Lowercase letters, numbers, and hyphens only
- No XML tags
- No reserved words (anthropic, claude)

**Preferred style:** Gerund form (verb + -ing) clearly describes the activity.

Good examples:
- `processing-pdfs`
- `analyzing-spreadsheets`
- `managing-databases`
- `writing-documentation`

Avoid: vague names (`helper`, `utils`), overly generic (`documents`, `data`), reserved words.

### Writing Effective Descriptions

The `description` field in YAML frontmatter enables skill discovery and triggers.

**Critical:** Write in third person. The description is injected into the system prompt; inconsistent point-of-view causes discovery problems.

✓ Good: "Processes Excel files and generates reports"
✗ Bad: "I can help you process Excel files"
✗ Bad: "You can use this to process Excel files"

**Be specific and include key terms:** Include both what the skill does AND specific triggers/contexts for when to use it. Example:

```yaml
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

### Structure

Minimum required:
```
skill-name/
├── SKILL.md (required)
│   ├── YAML frontmatter (required): name, description
│   └── Markdown body (required): instructions and guidance
└── Optional bundled resources:
    ├── scripts/      - Executable code (Python/Bash)
    ├── references/   - Documentation loaded as needed
    └── assets/       - Output files (templates, icons, fonts)
```

## Progressive Disclosure

Keep SKILL.md under 500 lines. Split content into separate files as complexity grows.

### Pattern 1: High-Level Guide with References

```markdown
# PDF Processing

## Quick start
Extract text with pdfplumber: [code]

## Advanced features
- **Form filling**: See [FORMS.md](FORMS.md)
- **API reference**: See [REFERENCE.md](REFERENCE.md)
- **Examples**: See [EXAMPLES.md](EXAMPLES.md)
```

Claude loads reference files only when needed.

### Pattern 2: Domain-Specific Organization

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md
    ├── sales.md
    ├── product.md
    └── marketing.md
```

Claude reads only the relevant domain file.

### Pattern 3: Conditional Details

```markdown
# DOCX Processing

## Creating documents
Use docx-js for new documents. See [DOCX-JS.md](DOCX-JS.md).

## Editing documents
For simple edits, modify XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
```

### Key Guidelines

- **Avoid deeply nested references:** Keep all references one level deep from SKILL.md for complete file reads
- **Add table of contents to long files:** For reference files >100 lines, include TOC so Claude sees the full scope
- **Don't duplicate content:** Information lives in either SKILL.md or references, not both. Prefer references for detailed information to keep SKILL.md lean

## Workflows and Content

### Use Workflows for Complex Tasks

Break complex operations into clear sequential steps. Provide checklists Claude can copy and check off:

```markdown
## Analysis Workflow

Copy this checklist and track progress:

```
- [ ] Step 1: Read source documents
- [ ] Step 2: Identify key themes
- [ ] Step 3: Cross-reference claims
- [ ] Step 4: Create structured summary
- [ ] Step 5: Verify citations
```

**Step 1: Read source documents**
[Details]

**Step 2: Identify key themes**
[Details]
```

Checklists prevent skipping critical validation steps.

### Implement Feedback Loops

**Pattern:** Run validator → fix errors → repeat

For skills with code:
```markdown
1. Make your edits...
2. **Validate**: `python validate.py`
3. If validation fails:
   - Review error messages
   - Fix issues
   - Run validation again
4. **Only proceed when validation passes**
```

For skills without code: Reference a style guide or checklist for manual validation.

### Content Guidelines

**Avoid time-sensitive information.** Use "old patterns" sections for deprecated approaches instead of date-based conditions.

**Use consistent terminology** throughout. Choose one term per concept:
- Always "API endpoint" (not "URL", "route", "path")
- Always "field" (not "box", "element", "control")
- Always "extract" (not "pull", "get", "retrieve")

## Common Patterns

### Template Pattern

Provide templates for output format. Adjust strictness to requirements:

**For strict requirements:**
```markdown
## Report structure

ALWAYS use this exact template:

```markdown
# [Title]
## Executive summary
[One paragraph]
## Key findings
- Finding 1
- Finding 2
## Recommendations
1. Recommendation
2. Recommendation
```
```

**For flexible guidance:**
```markdown
## Report structure

Use this sensible default, but adapt as needed:

```markdown
# [Title]
## Executive summary
[Overview]
## Key findings
[Adapt sections]
## Recommendations
[Tailor to context]
```
```

### Examples Pattern

Provide input/output pairs showing desired style and detail level:

```markdown
## Format Examples

**Example 1:**
Input: Added user authentication
Output:
```
feat(auth): implement JWT authentication

Add login endpoint and token validation
```

**Example 2:**
Input: Fixed date display bug
Output:
```
fix(reports): correct date formatting

Use UTC timestamps consistently
```
```

### Conditional Workflows

Guide through decision points:

```markdown
## Modification workflow

1. Determine type:
   **Creating new?** → Follow "Creation" below
   **Editing existing?** → Follow "Editing" below

2. Creation:
   - Use appropriate library
   - Build from scratch

3. Editing:
   - Unpack existing
   - Modify content
   - Validate
   - Repack
```

## Anti-Patterns to Avoid

**Use forward slashes** in all file paths (even on Windows):
- ✓ `scripts/helper.py`, `reference/guide.md`
- ✗ `scripts\helper.py`, `reference\guide.md`

**Avoid offering too many options.** Provide a default with escape hatch:
- ✗ Bad: "You can use pypdf, pdfplumber, PyMuPDF, pdf2image, or..."
- ✓ Good: "Use pdfplumber for text extraction. For scanned PDFs requiring OCR, use pdf2image with pytesseract instead."

**Solve, don't punt.** Scripts should handle errors explicitly rather than failing and asking Claude to figure it out. Justify all configuration values (no "voodoo constants").

**Provide utility scripts** for deterministic operations. Pre-made scripts offer:
- Higher reliability than generated code
- Token savings (no code in context)
- Time savings (no generation needed)
- Consistency across uses

Make clear whether Claude should execute or read scripts as reference.

**Create verifiable intermediate outputs** for complex operations. Use "plan-validate-execute":
1. Claude creates plan in structured format (JSON/YAML)
2. Validation script checks plan before execution
3. Only execute after validation passes

Example: For batch PDF updates, validate field changes before applying.

## Runtime Environment

Skills run in a code execution environment with filesystem access, bash commands, and code execution.

**How Claude accesses skills:**
1. Metadata (name + description) pre-loaded at startup
2. SKILL.md and files read on-demand via bash
3. Scripts executed efficiently without loading their full contents
4. Reference files don't consume tokens until read

**Implications for authoring:**
- **File paths matter:** Use forward slashes (`reference/guide.md`), not backslashes
- **Name descriptively:** Use `form_validation_rules.md`, not `doc2.md`
- **Organize for discovery:** Structure by domain/feature, not numbering
- **Bundle comprehensive resources:** Include complete docs, API specs, datasets—no context penalty until accessed
- **Prefer scripts for deterministic operations:** Write `validate_form.py` rather than asking Claude to generate validation code
- **Make execution intent clear:**
  - "Run `analyze_form.py` to extract fields" (execute)
  - "See `analyze_form.py` for the algorithm" (read as reference)

**For MCP tools:** Always use fully qualified names: `ServerName:tool_name`

Example: `BigQuery:bigquery_schema`, `GitHub:create_issue`

**Explicit about dependencies:** Don't assume packages are available.

```markdown
**Good:**
"Install required package: `pip install pypdf`

Then use it:
```python
from pypdf import PdfReader
```
"

**Avoid:**
"Use the pdf library to process the file."
```

## Evaluation and Iteration

### Build Evaluations First

Create evaluations BEFORE extensive documentation. This ensures your skill solves real problems, not imagined ones.

**Process:**
1. Identify gaps—run Claude on representative tasks without skill, document failures
2. Create evaluations—build 3+ scenarios testing these gaps
3. Establish baseline—measure Claude's performance without skill
4. Write minimal instructions—create just enough to pass evaluations
5. Iterate—execute evaluations, compare against baseline, refine

**Evaluation structure:**
```json
{
  "skills": ["pdf-processing"],
  "query": "Extract all text from PDF and save to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Reads PDF using appropriate library",
    "Extracts text from all pages",
    "Saves to output.txt in readable format"
  ]
}
```

### Develop Iteratively with Claude

Work with Claude A (creates skill) and Claude B (tests skill):

**Creating:**
1. Complete task without skill using normal prompting
2. Notice context you repeatedly provide
3. Ask Claude A: "Create a skill capturing this pattern"
4. Review for conciseness
5. Improve information architecture
6. Test with Claude B on similar tasks
7. Iterate on observations

**Key insight:** Claude models understand skill format natively. Simply ask Claude to create a skill.

### Observe How Claude Navigates Skills

Watch for:
- **Unexpected exploration paths:** Does Claude read files in unanticipated order? Implies structure isn't intuitive
- **Missed connections:** Does Claude fail to follow references? Links need to be more prominent
- **Overreliance:** Does Claude repeatedly read the same file? Consider moving that content to SKILL.md
- **Ignored content:** Does Claude never access a bundled file? It may be unnecessary or poorly signaled

Iterate based on observed behavior, not assumptions.

## Quality Checklist

Before sharing a skill:

**Core Quality**
- [ ] Description is specific with key terms
- [ ] Description includes what skill does AND when to use it
- [ ] SKILL.md body is under 500 lines
- [ ] Detailed content is in separate files (if needed)
- [ ] No time-sensitive information (or in "old patterns" section)
- [ ] Consistent terminology throughout
- [ ] Examples are concrete, not abstract
- [ ] File references are one level deep
- [ ] Progressive disclosure used appropriately
- [ ] Workflows have clear steps

**Code and Scripts**
- [ ] Scripts solve problems rather than punt to Claude
- [ ] Error handling is explicit and helpful
- [ ] All configuration values are justified
- [ ] Required packages listed and verified as available
- [ ] Scripts are well documented
- [ ] No Windows-style paths (all forward slashes)
- [ ] Validation/verification for critical operations
- [ ] Feedback loops for quality-critical tasks

**Testing**
- [ ] At least three evaluations created
- [ ] Tested with Haiku, Sonnet, and Opus
- [ ] Tested with real usage scenarios
- [ ] Team feedback incorporated (if applicable)

---

**Source:** [Skill Authoring Best Practices - Claude Platform Documentation](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices.md)
