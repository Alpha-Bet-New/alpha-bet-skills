
# Document Creation Rules

## Purpose
Defines the standard process, structure, and quality requirements for creating new documentation in the Alpha-Bet project. This ensures consistency, prevents conflicts, and maintains alignment with project goals.

## When to Use
- **Before creating any new documentation** - Follow this process for all docs
- **When restructuring existing docs** - Validate against these rules
- **During documentation reviews** - Check compliance with standards
- **Onboarding documentation writers** - Learn the doc creation process

## Called By
- Developers/writers creating new documentation
- AI agents generating documentation
- Documentation reviewers validating new docs
- Team leads planning documentation structure

## Calls
- All existing documentation (for context review)
- [docs/ROADMAP.md](ROADMAP.md) - Alignment check
- [docs/ARCHITECTURE.md](ARCHITECTURE.md) - Technical alignment
- [docs/DECISIONS.md](DECISIONS.md) - Decision alignment

---

## Core Principles

### 🚨 CRITICAL: Read Everything, Assume Nothing

**Two Non-Negotiable Rules:**

1. **READ ALL MATERIALS IN FULL**
   - No skimming, no summarizing, no "getting the gist"
   - Read every word of every document
   - Understand context, relationships, and dependencies
   - Take comprehensive notes

2. **MAKE NO ASSUMPTIONS**
   - When in doubt → **ASK**
   - When unclear → **ASK**
   - When choosing between options → **ASK**
   - When you think you know → **ASK ANYWAY**

**Why These Rules Exist:**
- Prevents conflicts and redundancy
- Ensures alignment with strategy and roadmap
- Maintains consistency across all documentation
- Respects existing decisions and architecture
- Saves time by avoiding rework

**Failure to Follow These Rules Will Result In:**
- ❌ Duplicate or conflicting content
- ❌ Misalignment with project strategy
- ❌ Wasted effort and rework
- ❌ Confusion for readers
- ❌ Damage to documentation credibility

---

## Document Creation Process

### Phase 1: Pre-Creation (Research & Planning)

#### Step 1: Read All Existing Documentation IN FULL

**CRITICAL: Before writing anything**, you MUST thoroughly read ALL existing documentation **in its entirety**:

1. **[README.md](../README.md)** - Project overview and current status
2. **[docs/ROADMAP.md](ROADMAP.md)** - Development phases and priorities
3. **[docs/ARCHITECTURE.md](ARCHITECTURE.md)** - Technical design principles
4. **[docs/DECISIONS.md](DECISIONS.md)** - Decision log and open questions
5. **[docs/API_STRATEGY.md](API_STRATEGY.md)** - Provider integration strategy
6. **[docs/0_rules_docCreator.md](0_rules_docCreator.md)** - This document (document creation rules)
7. **All other docs** - Check `/docs` directory and read ALL related content

**Reading Requirements:**
- ⚠️ **Read documents IN FULL** - No skimming, no summaries, no assumptions
- ⚠️ **Read EVERY section** - Don't skip Table of Contents, examples, or appendices
- ⚠️ **Take notes** - Document key points, dependencies, and potential conflicts
- ⚠️ **Understand relationships** - How docs reference and depend on each other

**Why This Matters:**
- Avoid duplicating existing content
- Understand full context and relationships
- Identify genuine gaps vs perceived gaps
- Ensure consistency with existing docs
- Prevent conflicts with established patterns
- Respect decisions already made

**⚠️ MAKE NO ASSUMPTIONS**
- If you're unsure about something → **Ask clarifying questions**
- If terminology is unclear → **Ask for definitions**
- If scope overlaps with existing docs → **Ask which doc should contain what**
- If alignment is questionable → **Ask for validation**
- If requirements are ambiguous → **Ask for specifics**

**When to Ask Questions:**
- ❓ Unclear requirements or scope
- ❓ Potential conflicts with existing docs
- ❓ Ambiguous terminology
- ❓ Uncertain about alignment with strategy/roadmap
- ❓ Multiple valid approaches exist
- ❓ Trade-offs need stakeholder input
- ❓ Any doubt about the right approach

**Better to ask than to assume wrong!**

---

#### Step 2: Validate the Need for New Document

Ask yourself:

**Does this content belong in an existing document?**
- If yes → Update existing doc instead of creating new one
- If no → Proceed to next check

**Does this create redundancy or conflict?**
- Check for overlapping content
- Ensure it doesn't contradict existing docs
- Verify it fills a genuine gap

**Is this temporary or permanent?**
- Temporary → Consider using comments, issues, or notes instead
- Permanent → Proceed with creating document

**Alignment Questions:**
- ✅ Does this align with the roadmap? ([ROADMAP.md](ROADMAP.md))
- ✅ Does this support our trading strategy?
- ✅ Does this follow our architectural principles? ([ARCHITECTURE.md](ARCHITECTURE.md))
- ✅ Is this consistent with past decisions? ([DECISIONS.md](DECISIONS.md))

**If any answer is "No"**, consult with the team before proceeding.

---

#### Step 3: Define Document Scope

Clearly define:

1. **Purpose** - What problem does this doc solve?
2. **Audience** - Who will read this?
3. **Scope** - What's included (and excluded)?
4. **Relationship** - How does it fit with other docs?

**Document Naming Convention:**
- Use descriptive, lowercase names with hyphens
- Prefix with numbers only for ordering (e.g., `0_rules_docCreator.md`)
- Use `.md` extension (Markdown)
- Examples:
  - ✅ `API_STRATEGY.md`
  - ✅ `deployment-guide.md`
  - ✅ `0_rules_docCreator.md`
  - ❌ `doc1.md` (not descriptive)
  - ❌ `API Strategy.md` (spaces)

---

### Phase 2: Document Structure (Required Sections)

Every document **MUST** include these sections at the top:

```markdown
# Document Title

## Purpose
[1-3 sentences: What this document is for and what problem it solves]

## When to Use
- **Scenario 1** - Description
- **Scenario 2** - Description
- **Scenario 3** - Description
[List specific situations when someone should reference this document]

## Called By
- [List who/what references this document]
- Other documents (with links)
- Team roles
- AI agents
- Tools/scripts

## Calls
- [List what this document references]
- Other documents (with links)
- External resources
- APIs/Tools

---

[Main document content begins here]
```

---

### Phase 3: Content Guidelines

#### Writing Style

**Be Clear and Concise:**
- Use simple, direct language
- Avoid jargon unless necessary (define when used)
- Short sentences and paragraphs
- Active voice over passive

**Be Specific:**
- Include concrete examples
- Provide code samples where relevant
- Link to related resources
- Use bullet points and tables for clarity

**Be Actionable:**
- Focus on "how" not just "what"
- Include steps, not just descriptions
- Add checklists for processes
- Specify owners/timelines when relevant

#### Structure Best Practices

**Use Headings Hierarchically:**
```markdown
# H1 - Document Title (only one per doc)
## H2 - Major sections
### H3 - Subsections
#### H4 - Detailed points
```

**Include Navigation Aids:**
- Table of Contents for long docs (>5 sections)
- Cross-references to related docs
- "Back to top" links for very long docs

**Use Formatting Effectively:**
- **Bold** for emphasis
- `code` for technical terms, commands, file names
- > Blockquotes for important notes
- Lists for steps or options
- Tables for comparisons

#### Code Examples

When including code:

```markdown
```python
# Include language identifier for syntax highlighting
# Add comments to explain non-obvious code
# Keep examples concise but complete
def example_function():
    return "Clear and annotated"
\```
```

**Code Example Checklist:**
- [ ] Language specified
- [ ] Comments explain key parts
- [ ] Complete enough to be useful
- [ ] Follows project conventions

---

### Phase 4: Validation & Quality Checks

Before finalizing the document, complete this checklist:

#### Structure Validation
- [ ] Includes all required metadata sections (Purpose, When to Use, Called By, Calls)
- [ ] Has clear, descriptive title
- [ ] Uses proper heading hierarchy (H1 > H2 > H3)
- [ ] Table of Contents included (if >5 major sections)
- [ ] Proper Markdown formatting throughout

#### Content Validation
- [ ] Purpose is clear and concise (1-3 sentences)
- [ ] "When to Use" scenarios are specific and actionable
- [ ] All cross-references use proper links
- [ ] No broken links
- [ ] Code examples include language identifiers
- [ ] Technical terms are defined or linked to definitions

#### Consistency Checks
- [ ] Read all existing docs for context
- [ ] No duplicate content with existing docs
- [ ] No conflicts with existing docs
- [ ] Consistent terminology with other docs
- [ ] Follows naming conventions

#### Alignment Checks
- [ ] Aligns with [ROADMAP.md](ROADMAP.md) phases and priorities
- [ ] Supports the systematic trading strategy (data-driven, repeatable)
- [ ] Follows [ARCHITECTURE.md](ARCHITECTURE.md) principles
- [ ] Consistent with [DECISIONS.md](DECISIONS.md) decisions
- [ ] Doesn't contradict core philosophy (flexible, scalable, systematic)

#### Cross-Reference Updates
- [ ] Update [README.md](../README.md) if adding major doc
- [ ] Update related docs' "Calls" sections to reference this doc
- [ ] Add this doc to related docs' "Called By" sections
- [ ] Update any navigation/index documents

#### Final Quality Checks
- [ ] Spell-checked
- [ ] Grammar-checked
- [ ] Read through as if you're the target audience
- [ ] All links tested
- [ ] Version history added at bottom

---

### Phase 5: Publishing & Integration

#### Git Workflow

1. **Create document in proper location:**
   - Major docs → `/docs/`
   - Specialized docs → `/docs/[category]/`
   - Root-level → Only for README.md

2. **Commit with descriptive message:**
```bash
git add docs/new-document.md
git commit -m "Add [document name] documentation

[Brief description of what the doc covers]
[Why it was needed]

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

3. **Update related documents in same commit:**
   - Add cross-references
   - Update "Calls" and "Called By" sections

4. **Push to repository:**
```bash
git push origin main
```

#### Announcement & Communication

- Notify team of new documentation
- Share in relevant Slack/Discord channels
- Add to team meeting agenda if significant
- Update any documentation indexes

---

## Document Maintenance

### When to Update Documents

**Regular Updates:**
- After major decisions (update DECISIONS.md)
- When architecture changes (update ARCHITECTURE.md)
- Phase transitions (update ROADMAP.md)
- API changes (update API_STRATEGY.md)

**Immediate Updates:**
- When information becomes outdated
- When links break
- When conflicts are discovered
- When team asks questions not answered in docs

### Version History

Every document should include a version history at the bottom:

```markdown
## Version History

- **v2.0 (YYYY-MM-DD):** Major restructure with metadata headers
- **v1.1 (YYYY-MM-DD):** Added code examples section
- **v1.0 (YYYY-MM-DD):** Initial document created
```

---

## Special Document Types

### Decision Documents

When documenting decisions:
- Add to [DECISIONS.md](DECISIONS.md) decision log table
- Include date, decision, rationale, status, owner
- Document alternatives considered
- Link to supporting documents

### API Documentation

When documenting APIs:
- Follow patterns in [API_STRATEGY.md](API_STRATEGY.md)
- Include code examples
- Document error handling
- Provide test examples

### Architecture Documents

When documenting architecture:
- Follow principles in [ARCHITECTURE.md](ARCHITECTURE.md)
- Include diagrams (ASCII or images)
- Show code patterns
- Explain trade-offs

### Process Documents

When documenting processes:
- Use numbered steps
- Include checklists
- Provide examples
- Define success criteria

---

## Anti-Patterns to Avoid

### ❌ Don't Create These Documents

**Redundant Documents:**
- If it duplicates existing content → Update existing doc instead

**Overly Specific Documents:**
- "How to Fix Bug #123" → Use issue tracker instead
- "Meeting Notes 2024-11-20" → Use wiki/notes system instead

**Temporary Documents:**
- "TODO List" → Use project management tool instead
- "Scratch Notes" → Keep in personal notes

**Orphaned Documents:**
- Not linked from any other doc → Integrate or delete
- No clear purpose → Rethink necessity

### ❌ Don't Write Like This

**Vague Purpose:**
```markdown
❌ ## Purpose
This document is about APIs.
```

**Better:**
```markdown
✅ ## Purpose
Defines the strategy for integrating with sportsbook APIs, including provider priorities, error handling patterns, and testing approaches.
```

**Unclear "When to Use":**
```markdown
❌ ## When to Use
- When working with APIs
```

**Better:**
```markdown
✅ ## When to Use
- **Selecting providers to integrate** - Check priority rankings and pros/cons
- **Implementing provider adapters** - Reference design patterns and code examples
- **Handling API errors** - Consult error handling strategies
```

**Missing Cross-References:**
```markdown
❌ See the architecture document for more details.
```

**Better:**
```markdown
✅ See [ARCHITECTURE.md](ARCHITECTURE.md) for detailed design patterns.
```

---

## Examples of Good Documents

Study these as templates:

1. **[ARCHITECTURE.md](ARCHITECTURE.md)** - Comprehensive technical doc with code examples
2. **[API_STRATEGY.md](API_STRATEGY.md)** - Strategy doc with priorities and patterns
3. **[DECISIONS.md](DECISIONS.md)** - Decision tracking with structured questions
4. **[ROADMAP.md](ROADMAP.md)** - Timeline doc with clear phases and priorities

---

## Pre-Submission Checklist

Before submitting your new document, verify:

### Documentation Standards
- [ ] All required metadata sections present (Purpose, When to Use, Called By, Calls)
- [ ] Proper Markdown formatting
- [ ] Clear, descriptive title
- [ ] Appropriate file naming (lowercase, hyphens, .md)
- [ ] Version history section at bottom

### Content Quality
- [ ] Purpose is clear and specific (1-3 sentences)
- [ ] "When to Use" scenarios are actionable
- [ ] All cross-references use proper links
- [ ] No broken links
- [ ] Code examples properly formatted with language identifiers
- [ ] Spell-checked and grammar-checked

### Context & Consistency
- [ ] **Read all existing documentation** for full context
- [ ] No duplicate content
- [ ] No conflicts with existing docs
- [ ] Consistent terminology
- [ ] Proper cross-referencing

### Alignment & Strategy
- [ ] Aligns with [ROADMAP.md](ROADMAP.md) - Supports current or future phases
- [ ] Supports systematic trading strategy - Data-driven, repeatable, scalable
- [ ] Follows [ARCHITECTURE.md](ARCHITECTURE.md) - Technical principles maintained
- [ ] Consistent with [DECISIONS.md](DECISIONS.md) - No contradicting past decisions
- [ ] Upholds core philosophy - Flexible, systematic, scalable architecture

### Integration
- [ ] Updated [README.md](../README.md) if major document
- [ ] Updated related docs' "Calls" sections
- [ ] Updated related docs' "Called By" sections
- [ ] Git commit message is descriptive

### Final Verification
- [ ] **Does NOT violate any architectural principles**
- [ ] **Does NOT create conflicts with roadmap**
- [ ] **Does NOT contradict trading strategy**
- [ ] **Fills a genuine gap** in documentation
- [ ] **Provides clear value** to readers

---

## Emergency: Document Conflicts

If you discover your new document conflicts with existing docs:

### Step 1: Identify the Conflict
- Which document(s) conflict?
- What specific information conflicts?
- Is it a direct contradiction or just overlap?

### Step 2: Determine Resolution
- Is existing doc correct? → Update your new doc
- Is new doc correct? → Update existing doc
- Both partially correct? → Reconcile and update both

### Step 3: Escalate if Needed
- Architectural conflicts → Discuss with team lead
- Strategic conflicts → Raise in planning meeting
- Decision conflicts → Document in DECISIONS.md

### Step 4: Update All Affected Docs
- Never leave conflicting information
- Update all docs to be consistent
- Note the resolution in commit message

---

## Quick Reference: Document Creation Workflow

```
1. ☑️ Read all existing docs
2. ☑️ Validate need for new doc
3. ☑️ Define scope and audience
4. ☑️ Create with required structure
5. ☑️ Write clear, concise content
6. ☑️ Add code examples (if relevant)
7. ☑️ Run all validation checks
8. ☑️ Update cross-references
9. ☑️ Commit with descriptive message
10. ☑️ Announce to team
```

---

## Questions?

If you're unsure about documentation:
- Review existing docs as examples
- Ask team lead for guidance
- Consult [DECISIONS.md](DECISIONS.md) for past context
- Check [ROADMAP.md](ROADMAP.md) for strategic alignment

**Remember:** Good documentation is clear, consistent, and connected to the rest of the project.

---

## Version History

- **v1.1 (2024-11-20):** Added explicit "Read Everything, Assume Nothing" core principles section with emphasis on full document reading and asking clarifying questions
- **v1.0 (2024-11-20):** Initial document creation rules established