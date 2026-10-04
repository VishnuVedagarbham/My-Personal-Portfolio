# Prompt for Windsurf — Portfolio Site Refactor

Paste everything below into Windsurf as one message. It's written to be self-contained.

---

## Context

This is my personal portfolio site (static HTML/CSS/JS, deployed via GitHub Pages). The repo root contains `index.html` plus ~30 other HTML files, a `css/styles.css`, a `js/main.js`, an `images/` folder, and a few PowerShell scripts (`add_contact_buttons.ps1`, `remove_contact_buttons.ps1`, `update_secondary_pages.ps1`) that were being used to bulk-patch HTML files by string-replacing text like `</body>`.

I need two things done, in this order, and I need you to **not** touch the visual design while doing either of them.

## Hard constraint: do not change the design

This is the single most important rule for this task. Do not change:
- The background: a fixed, full-viewport photo (`images/Vishnu Vedagarbham picture.jpg`) with a dark translucent navy overlay on top (`background: rgba(5, 13, 25, 0.85)` / `0.9` in `css/styles.css`, and a `linear-gradient(135deg, rgba(10,25,60,0.7) 0%, rgba(5,15,40,0.9) 100%)` variant on the hero section).
- The nav bar: fixed top, `rgba(10, 25, 49, 0.7)` background, `backdrop-filter: blur(5px)`, centered links with `gap: 30px`, active-link underline.
- The hero title gradient text (`linear-gradient(90deg, #7d2ae8 40%, #5b13b9 100%)` clipped to text), the `#5ed6ff` accent color used throughout for links/borders/hover states, and the `Inter` font.
- The "cool-btn-card" grid on the About section (icon + title + description + "Dig Deeper" link, in a card with hover lift).
- The list-card → detail-page click-through pattern used on Goals, Education, Work Experience, Projects, Certifications, and Extracurriculars.
- Any spacing, border-radius, button styles, or hover animations currently in `css/styles.css`.

Effectively: `css/styles.css` should end up doing the same visual job it does now — reused and cleaned up, not redesigned. If you need to consolidate the inline `<style>` blocks that are currently duplicated inside individual page files (see Problem 3 below) back into `css/styles.css`, that's expected and good, as long as the rendered output looks pixel-identical to what's live now at each page.

If at any point a refactor would change how something looks, stop and ask me instead of guessing.

## Part 1: Fix the architecture (move away from file-per-page)

### Problem: massive duplication across ~30 static HTML files

Right now every "detail" page (e.g. `work_experience_1.html`, `goals_2.html`, `education_3.html`, `certifications_4A.html`, `project_1.html`, etc.) is a hand-copied HTML file that repeats the entire page shell: `<head>`, an inline `<style>` block duplicating the body background/overlay rules, the "Return to Home" button, the content, and a "Contact Me" button. There's no single source of truth for content — content lives scattered across dozens of files.

Symptoms this has already caused, that I want fixed as part of the refactor:
1. **Inline `<style>` drift**: several subpages (e.g. `work_experience.html`, `goals.html`, `education.html`, `extracurriculars.html`, `certifications.html`, `contact.html`, `references.html`) have their own copy-pasted `<style>` block using `background: rgba(40, 20, 10, 0.5)` (a brownish overlay), while `css/styles.css` and other pages use `rgba(5, 13, 25, 0.85)`/`0.9` (navy). These have drifted out of sync from copy-pasting. There should be exactly one definition of the background/overlay treatment, in `css/styles.css`, and every page should use it.
2. **The PowerShell scripts** (`add_contact_buttons.ps1`, `remove_contact_buttons.ps1`, `update_secondary_pages.ps1`) exist because adding a UI element (a "Contact Me" button) to all pages meant string-replacing `</body>` across dozens of files by hand. This is not sustainable and should not be necessary after the refactor — a shared layout/partial should mean I add something once.
3. **A stray, unused `template.html` and `templates/secondary_template.html`** exist in the repo — scaffold files that were presumably meant to be copied for new pages. After the refactor these should either be deleted or turned into the actual real template the build uses (not a manually-copied starting point).

### What I want instead

Convert this into a **data-driven, templated build** — content and presentation should be separated, and adding/editing content should never again require creating a new HTML file by hand or running a PowerShell script.

You have two reasonable paths — pick whichever fits a plain HTML/CSS/JS + GitHub Pages site better, and tell me which you chose and why:

**Option A — Static site generator (preferred if low-friction to set up):** Use a lightweight generator like Eleventy (11ty) that can output plain static HTML from Nunjucks/Liquid templates + a JSON/YAML/JS data file, with a GitHub Actions workflow (or a simple `npm run build` I run manually before pushing) that builds to a `dist`/`docs` folder for GitHub Pages. Real, separate URLs per page are preserved (e.g. `/work-experience/`, `/projects/near-space-balloon/`), so nothing about the site's navigability changes — the difference is that pages are generated from one shared layout + a content file instead of hand-copied HTML.

**Option B — Single build-time template script:** A small Node (or Python) script that reads one content file (e.g. `content.json`) and a small set of `.html` templates (page shell, list-page template, detail-page template), and generates the final static HTML files into the repo. Simpler than a full SSG, but I do the "build" step manually whenever content changes.

Either way:
- There should be exactly **one** file that defines page chrome (nav, background/overlay div, "Return to Home" / "Contact Me" buttons, footer) — every page includes/extends it instead of repeating it.
- There should be exactly **one** content source (JSON, YAML, or JS data file) holding all the text for Goals, Education, Work Experience, Projects, Certifications, and Extracurriculars — including each list page's items and each detail page's body content. Adding a new work experience entry should mean adding one object to this file, not creating a new `.html` file.
- The list → detail click-through UX and URL structure should stay conceptually the same as today (a list page linking to individual detail pages), just generated instead of hand-maintained.
- Delete the PowerShell scripts and `template.html`/`templates/secondary_template.html` once the new build process replaces what they were doing.
- `js/main.js`'s existing behavior (profile image modal on click, smooth-scroll nav, scroll-based nav-link highlighting) should keep working unchanged.

## Part 2: Fix the content (once the architecture is sorted)

The content itself is stale/incomplete in several places. Please fix all of the following as part of migrating content into the new data file:

1. **Work Experience is missing my two most recent and most relevant jobs.** Currently `work_experience.html` only lists "Office Assistant" (UMN Housing) and "Student Custodial Worker." It's missing:
   - **Advanced AI Engineering Fellow, Handshake AI, June 2026 – Present** — I design evaluation prompts and blind-rank frontier LLMs on correctness, agent behavior, and code quality, and write technical justifications used as RLHF training data.
   - **Software Engineering Intern, Stealth Startup (FinTech), May 2026 – Aug 2026** — designed and unit-tested scalable user features for a cloud-hosted corporate banking app, built backend microservices for user lifecycle management, and drove business logic for onboarding/tax-engine modules. (Keep this generic — no company name, no specifics beyond what's in my resume — this is a stealth company under NDA.)
   Add both as new Work Experience entries, above the existing two, using my resume as the source of truth for wording.

2. **`work_experience_3.html` has the wrong content.** It's titled "Office Assistant / UMN Housing and Residential Life" but its bullet points are actually copy-pasted from `work_experience_1.html` (the Youngevity/DailyWithDoc internship — "Utilized AI Prompt Engineering... Leveraged AI tools including WindSurf, Bolt.new, and GenSpark... Designed social media graphics using Canva"). Please fix this data mismatch — figure out which bullets actually belong to the Office Assistant role versus the Youngevity role and correct both entries. If you can't tell from context, flag it to me rather than guessing.

3. **Projects page has a dead link and an unfinished placeholder.** `projects.html` lists:
   - "Data Visualizer" → links to `#` (dead link, no real project behind it). Remove this entry.
   - "Project 6" → literally still has the placeholder title "Project 6" and description "Click to view project details." Remove this entry, or if it refers to a real project, ask me what it should be.
   - It's also missing two real projects that are on my resume but not on the site at all: the **Near-Space Research Ballooning Payload System** (atmospheric research payload to 90,000 ft, PICOLO microcontroller firmware, RM-80 Geiger counter, 10,000+ rows of telemetry analyzed) and the **Spotify Sorter Extension & Web API Engine** (Java/JavaScript browser extension, OAuth, playlist sorting, tested on 1,000+ song playlists). Add both, using my resume's bullet points as the source content.
   - `project_1.html` (the "Portfolio Website" project detail page) is entirely unfilled placeholder text — "[Project description and details go here]," "Technology 1/2/3," "Feature 1/2/3." Either write real content describing this portfolio site as the project, or flag it to me.

4. **Certifications section is entirely placeholder.** All four entries in `certifications.html` (Backend Development, Frontend Development, Core Languages, Cloud Computing) say "Issuing Organization, Year" or similar unfilled text, and none of the detail sub-pages (`certifications_1.html` through `certifications_6A.html`) appear to have real content either. I don't currently have real certifications to list. **Remove the Certifications section from the site entirely** (nav card on the About grid, the list page, and all detail pages) rather than shipping placeholder content. If I add real certifications later I'll ask you to re-add the section.

5. **Extracurriculars section doesn't reflect my actual involvement.** `extracurriculars.html` currently lists generic placeholders ("Coding Club," "Honor Society") with empty detail pages (`extracurricular_1.html`, `extracurricular_2.html` are fully unfilled `[Activity/Role]` templates). My resume lists my actual extracurricular involvement:
   - **Club Leadership** — Executive Board Member across AWS Cloud Club, Honors Multicultural Network (HMN), Data Nexus, Hindu Student Association (HSA), South Indian Student Association (SISA), and Society of Optics and Photonics (SOP).
   - **Collegiate Cricket** — selected to represent UMN at the Midwest Regional Championship and National Collegiate Cricket League; active player for the UMN Cricket Club.
   - **Community Involvement** — mentored underprivileged students and coordinated healthcare resource allocation through volunteer work with the Panchajanya Foundation and Abhyudaya.
   Replace the current placeholder entries with these three, using my resume's language as the source.

6. **Goals page has some unfinished sections.** `goals_2.html` ("Professional Goals") has placeholder bullets (`[Action Item 1]`, `[Action Item 2]`, `[Action Item 3]`) alongside a real paragraph about my interest in AI/ML — remove the placeholder bullets and keep the real paragraph, or ask me for real action items to replace them. `education_2.html` has a "Primrose" entry with placeholder dates (`[Month Year] – [Month Year]`) and placeholder achievements — ask me for the real dates rather than guessing.

7. **`references.html`** links to a "DailyWithDoc Internship recommendation" PDF (which exists in `references/`) and an "Academic Recommendation from Dr. M. A. Syed" (no PDF present in the repo). Confirm both PDFs exist and are linked correctly; flag if one is missing.

## What to give me back

1. A short summary of which architecture option (A or B) you went with and why.
2. A list of every content decision you had to make (anything under "flag it to me" above) instead of silently guessing.
3. Confirmation that you visually diffed the rebuilt pages against the current live site (or at least eyeballed each page type — home, a list page, a detail page) to confirm nothing about the look changed.
