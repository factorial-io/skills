# Estimation Tree Format

This document defines the exact output format for the importable estimation tree. The format is designed for import into estimation/project management tools (e.g., Productive, Forecast, Scoro).

## Format Rules

1. **Indentation** = hierarchy. Use 2 spaces per level.
2. **No numbering** — no "1.1", "1.2", etc.
3. **No markdown formatting** — no bold (`**`), italic (`*`), or other markup
4. **Separator** between label and description: ` - ` (space dash space)
5. **Decimal separator**: period (0.5d, not 0,5d)
6. **No empty lines** between items
7. **Effort lines** are indented one level deeper than their feature, formatted as `- FE: Xd` or `- BE: Xd`
8. **Header** at the top: project name, reference value, technical approach, assumptions

## Structure

```
# Project Name — Estimation Type
Reference value: Xd (description of reference task)
[Additional context: technical approach, who delivers UX/UI, etc.]
Epics:
- Epic Name - Epic Desctiption
  - Feature Name - Short description of what this delivers
    - FE: Xd
    - BE: Yd
  - Another Feature - Description
    - FE: Xd
- Another Epic - Another Epid description
  - Feature - Description
    - BE: Xd
```

Supported department shortcuts:
- BE: Backend
- FE: Frontend
- PM: Project Management
- PO: Product Ownership
- UI: UI Design
- UX: UX Design
- MGMT: Management
- CONS: Consulting
- QA: Quality Assurance

## Example

```
# Acme Corp — Website Rebuild Estimation
Reference value: 0.5d (simplest component per discipline)
Technical approach: Drupal 11 with Canvas, SDC components.
UX/UI delivered by client agency.
Epics:
- Infrastruktur & Setup
  - CMS Setup & Configuration - Installation, managed hosting, CI/CD, staging/production
    - FE: 1.5d
    - BE: 2.5d
  - Theme & Design System Foundation - Base theme, design tokens, typography, grid, responsive breakpoints
    - FE: 2.5d
    - BE: 1d
  - Content Architecture - Content types, taxonomies, menu structure, URL patterns
    - BE: 4d
  - Multilingual DE/EN - i18n configuration, translation workflow, URL strategy, language switcher
    - FE: 1d
    - BE: 2.5d
- Components — Simple - Pure markup/styling components, no JS logic
  - C01 Hero Header Simple - Title + subtitle centered, breadcrumb
    - FE: 0.5d
  - C02 Teaser Card - Image + title + text + CTA button
    - FE: 0.5d
  - C03 Blockquote - Large quotation marks + quote text + attribution
    - FE: 0.5d
- Components — Medium - Components with JS interaction or complex markup
  - C10 Accordion / FAQ - Expandable list items with toggle animation
    - FE: 1d
  - C11 Main Navigation - Multi-level nav, mobile hamburger, responsive
    - FE: 3d
  - C12 Footer - Logo, multi-column sitemap, social icons, legal links
    - FE: 1.5d
- Components — Complex - Components with rich interactivity or custom logic
  - C20 Timeline - Year markers with alternating image+text blocks, galleries per entry
    - FE: 2.5d
  - C21 Registration Form - Multi-field form with validation, consent checkboxes, CAPTCHA
    - FE: 1.5d
- Forms & Integrations - Backend logic for forms and third-party connections
  - Contact Form Backend - Form processing, email delivery, spam protection
    - BE: 1.5d
  - External API Integration - REST/JSON feed import, cron sync, error handling
    - FE: 1d
    - BE: 4d
- Content Types & Views - Data models and dynamic listings
  - Content Type Product - Structured product data, variants, category assignment
    - BE: 2d
  - View Product Overview - Category filters, grid layout, AJAX filtering
    - FE: 2d
    - BE: 2.5d
- Authentication & User Management
  - User Auth System - Login flows, roles, permissions, access control
    - BE: 5d
- Content Migration
  - Blog Article Migration - Migrate existing articles from old CMS
    - BE: 2.5d
- Standards & SEO
  - Analytics Setup - GA4/Matomo, event tracking, consent-controlled
    - BE: 1d
  - SEO Basics - Meta tags, sitemap, robots.txt, canonical URLs, hreflang
    - BE: 2d
  - Accessibility WCAG 2.1 AA - Semantic HTML, ARIA, keyboard nav, contrast. Base effort, mostly in components.
    - FE: 1.5d
  - Redirects & URL Migration - 301 redirects from old URL structure
    - BE: 1d
```

## Totals

After creating the tree, always calculate and present:
- FE total (sum of all FE lines)
- BE total (sum of all BE lines)
- Grand total (FE + BE)
- Note that governance (PM, PO, QA) is added as a percentage on top

## Converting to Hours and Cost

When the user has a rate card:
- Clarify productive hours per day (typically 6h or 8h)
- Multiply days × hours/day to get hours
- Multiply hours × hourly rate to get cost
- Apply governance percentage as surcharge on the work total
- Different roles (senior vs. mid-level) may have different rates — assign roles based on feature complexity
