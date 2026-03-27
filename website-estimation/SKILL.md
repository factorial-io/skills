---
name: website-estimation
description: |
  Guided workflow for creating relative website project estimations. Use this skill whenever the user wants to estimate, scope, or price a website project — whether they say "estimation", "Schätzung", "Aufwandsschätzung", "quote", "proposal", "Angebot", "scope a project", "how long will this take", or similar. Also trigger when the user mentions analyzing wireframes or designs for estimation purposes, creating a component catalog, writing a rebriefing document, or comparing estimations against reference projects. Works for any CMS or framework (Drupal, WordPress, Next.js, etc.) and any project size.
---

# Website Project Estimation

A guided, step-by-step process for creating professional website project estimations using relative estimation methodology.

## Core Methodology: Relative Estimation

Instead of guessing hours directly, find the smallest meaningful task and use it as a yardstick. Everything else is estimated as a multiple: "this feels like 3× the reference task." This removes the psychological pressure of committing to absolute numbers and produces more consistent results across estimators.

**How it works:**
1. Identify the simplest task in the project (e.g., a basic text component, a static page template)
2. Assign it a reference value in days (typically 0.5d per discipline)
3. Estimate everything else as a multiple of that reference
4. The reference value converts relative units to actual days

**Why this works better than direct hour estimation:** People are bad at guessing absolute durations but surprisingly good at comparing relative complexity. "Is this twice as hard as that?" is a much easier question than "How many hours will this take?"

## The Guided Process

This estimation workflow has 7 phases. Work through them in order, but adapt to whatever information is already available. Each phase ends with a checkpoint where you confirm findings with the user before moving on.

### Phase 1: Briefing Collection

Gather all available project information before estimating anything. Sources include emails, briefing documents, RFPs, meeting notes, and existing specifications.

**What to capture:**
- Project type (rebuild, redesign, new build, migration)
- Client and stakeholders (who delivers design, who is the technical contact)
- Business goals and target audience
- Known technical constraints (CMS preference, hosting, integrations)
- Existing content and assets to migrate
- Accessibility requirements (WCAG level, legal requirements like BFSG)
- Multilingual requirements (which languages, translation workflow)
- Timeline and budget range if known

**Where to look:**
- Check email (via MCP if available) for briefing threads
- Check project management tools (Jira, Linear) for existing tickets
- Ask the user what documents or information they already have

**Checkpoint:** Summarize what you've gathered and ask: "Is this complete, or is there more context I should know about?"

### Phase 2: Design Analysis & Component Catalog

Analyze available designs to build a comprehensive component inventory. Designs may come from Figma (via MCP), exported images (JPG/PNG), PDFs, or verbal descriptions.

**Process:**
1. Go through every page/screen systematically
2. Identify each unique UI component (not every instance — just unique patterns)
3. Give each component an ID (e.g., C01, C02) and a descriptive name
4. Note which pages use which components
5. Rate complexity using this framework:

| Complexity | Characteristics | Typical multiple |
|---|---|---|
| Simple | Pure markup/styling, no JS logic, no state | 1× reference |
| Medium | JavaScript interaction (accordions, tabs, sliders), responsive complexity, or complex markup structures | 2× reference |
| Complex | Multi-step forms, rich interactivity, external data, custom business logic | 4–6× reference |

**What to look for beyond components:**
- Content types (what structured data exists — products, articles, jobs, events)
- Dynamic listings and filter logic (which pages show filtered/sorted content)
- Authentication and protected areas (login flows, registration, role-based access)
- External integrations (APIs, feeds, third-party services)
- Forms and their backend requirements

**Checkpoint:** Present the component catalog grouped by complexity. Ask: "Does this look complete? Any components I missed or miscategorized?"

### Phase 3: Technical Approach

Define the technical architecture because it directly impacts estimation. The choice of CMS, component architecture, and page-building approach determines where effort falls.

**Key decisions to discuss with the user:**
- CMS / Framework (Drupal, WordPress, Next.js, headless, etc.)
- Component architecture (e.g., Drupal SDC, React components, Twig templates, custom blocks)
- Page building approach — does the client assemble pages, or does the dev team build templates?
- Hosting model (managed, self-hosted, platform-as-a-service)

**Architecture impact on estimation:**
- If components are directly placeable by editors (e.g., SDC in Drupal Canvas, Gutenberg blocks), there is no separate BE effort per component — components are frontend-only
- If the client builds pages in a visual editor, page template estimates drop significantly — you only estimate content types, views, and component development
- If using a headless approach, factor in API layer effort separately

**Checkpoint:** Confirm the technical approach with the user: "Based on what we know, I'd suggest [approach]. This means [impact on estimation]. Agreed?"

### Phase 4: Build the Estimation Structure

Now build the hierarchical estimation. The structure follows this pattern:

```
Epic (major project area)
  Feature (deliverable unit of work)
    Effort per discipline (FE, BE, optionally UX, UI)
```

**Standard epic structure for website projects:**

1. **Infrastruktur & Setup** — Project scaffolding, CI/CD, CMS installation, theme/design system foundation, content architecture, multilingual setup
2. **Components** — The full component library, grouped by complexity (simple, medium, complex). Each component gets its own line with FE effort.
3. **Forms & Integrations** — Backend logic for forms (contact, registration, newsletter), external system integrations (APIs, feeds, CRM, ATS), consent management
4. **Content Types & Views** — Data model definitions, dynamic listings with filter/sort logic, search functionality. Only the data structures and query logic — not page layout (that's the editor's job if using a page builder).
5. **Protected Areas / Portals** — Login flows, registration, role-based access, gated content sections
6. **Authentication & User Management** — Auth system, roles, permissions, password reset
7. **Content Migration** — Moving existing content from old to new system
8. **Standards & SEO** — Analytics, meta tags, structured data, performance optimization, accessibility baseline, URL redirects

This structure is a starting point. Adapt it to the project — skip epics that don't apply, add new ones if needed.

**Setting the reference:**
- Pick the simplest component or task as the reference
- Assign a day value per discipline (default: 0.5d)
- Estimate everything as a multiple

**Discipline split:**
- **FE** (Frontend): HTML/CSS/JS, component implementation, responsive behavior, interactions
- **BE** (Backend): CMS configuration, content types, views/queries, integrations, business logic, authentication
- **UX** and **UI** are often delivered by the client's design agency — if so, note it but don't estimate

**Governance (PM, PO, QA):** Add as a percentage surcharge on the total. Typical range: 10–20% depending on project complexity and client needs. Don't estimate governance per feature.

**Checkpoint:** Present the draft estimation structure (epics + features + descriptions, without numbers yet) and confirm the scope before adding effort values.

### Phase 5: Open Questions & Assumptions

Every estimation has unknowns. Make them explicit rather than hiding them in inflated numbers.

**Common question areas:**
- Scope boundaries (what's in, what's out)
- Integration complexity (is there API documentation? access to test systems?)
- Content volume (how many products, articles, pages to migrate?)
- Design delivery timeline and completeness
- Third-party dependencies (hosting provider, external services)
- Accessibility scope (which level, who tests?)

**Process:**
1. List all assumptions you've made during estimation
2. For each assumption, formulate a clear question
3. Create a document (in the client's language) with numbered questions
4. Group by topic for readability

**Output:** A clean markdown or email-ready document with all open questions. Use the same language the client communicates in (typically German for DACH projects).

**Checkpoint:** Review questions with the user before sending to the client.

### Phase 6: Refinement

After receiving answers to open questions, update the estimation:

1. Adjust scope (add/remove features based on client answers)
2. Update assumptions in the document header
3. Recalculate affected estimates
4. Present the delta: "These changes add X days / remove Y days, new total: Z"

This phase may repeat multiple times as scope evolves.

### Phase 7: Output Documents

Produce the final deliverables. There are three standard outputs:

#### 7a. Estimation Tree (importable format)

The primary estimation document, formatted for import into project management / estimation tools.

Read `references/tree-format.md` for the exact format specification and examples.

Key rules: no numbering, no markdown formatting (bold etc.), `- ` as separator between label and description, period as decimal separator, no empty lines between items.

#### 7b. Rebriefing Document

A professional document (docx and/or markdown) that summarizes:
- Project overview and technical approach
- Confirmed assumptions
- Epic descriptions (without cost numbers)
- Scope exclusions
- Next steps

This document serves as a shared understanding between all parties. It contains no pricing — that goes in the estimation tree or a separate commercial proposal.

If creating a docx, use the `docx` skill for professional formatting.

#### 7c. Estimation Comparison

Compare the current estimation against reference projects to validate reasonableness. Useful metrics:
- Total effort (days/hours/EUR)
- Effort distribution across epics (as %)
- Component count and average effort per component
- Governance percentage
- Rate and role mix

When comparing, account for structural differences: different projects may split work differently (one developer role vs. FE/BE split), use different day lengths (6h vs. 8h productive), or include different scope items.

## Practical Tips

**Productive hours per day:** Many agencies use 6 productive hours per day (not 8) to account for meetings, communication, and context switching. Clarify this with the user — it affects how day-based estimates convert to hours and costs.

**Rate cards:** If the user has a rate card, use it. Typical roles: developer senior, developer mid-level, consultant, UX designer, UI designer. Different roles may apply to different features (simple components → mid-level, integrations → senior).

**When UX/UI is delivered externally:** Note this in the estimation header. You may still want a small consulting/sparring budget for design review and technical feasibility feedback.

**Multilingual multiplier:** Multilingual setup is a fixed cost (CMS configuration, translation workflow). The per-content translation effort is usually the client's responsibility. Don't multiply every component estimate by the number of languages.

**Canvas / Page Builder projects:** When the client assembles pages in a visual editor, your estimation focuses on components, content types, and views — not page templates. This is a common pattern that significantly reduces BE effort per component.
