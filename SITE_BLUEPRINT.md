# yuvaravii.github.io — Site Blueprint

> **Status:** Draft — awaiting RJ's approval before build  
> **Stack:** Hugo + PaperMod theme + GitHub Actions  
> **URL:** https://yuvaravii.github.io

---

## 1. Why PaperMod Over HugoBlox

| Criteria | PaperMod | HugoBlox |
|---|---|---|
| Complexity | Minimal, zero JS build tooling | Heavy, Tailwind + Node dependency |
| Speed | Fastest Hugo theme | Slower builds |
| Maintenance | Dead simple | Frequent breaking updates |
| Customization | Easy override system | Plugin ecosystem (overkill for personal site) |
| Dark mode | Built-in, system-aware | Built-in |
| Search | Fuse.js built-in | Built-in |

**Verdict:** You're an AI/ML engineer, not a frontend dev. PaperMod gives you a clean, fast, professional site with zero frontend babysitting. You write markdown, push, done.

---

## 2. Experience Timeline

This section lives at `/experience/` — a clean chronological view of your career.

| Period | Role | Company | Key Work |
|---|---|---|---|
| 2026 – Present | AI/ML Engineer | Technosmile Japan (India Office) | TSC Matching Algorithm — RF surrogate optimization, NDCG evaluation, MLflow experiment tracking, GitHub Actions CI/CD. AI Avatar generation project. India business development. |
| Prior | Data Analyst | Merkle / Dentsu | Data analysis, Python automation, reporting. (Details to be filled — pull from your ATS resume) |
| Education | AI/ML Bootcamp | 100x School (Harkirat Singh) | Ongoing — deep learning fundamentals, transformers, NLP |
| Education | (Degree) | (University — fill in) | (Fill in your degree details) |

**Site implementation:** PaperMod doesn't have a native "timeline" layout, so we'll use a single page (`content/experience.md`) with a styled markdown table or a simple custom shortcode. Clean and scannable.

**What you need to fill in:** Your Merkle/Dentsu dates, your university/degree, and any other prior roles. Pull from your ATS resume (`Ravishankar_J_Resume_ATS.docx`).

---

## 3. Site Architecture

```
yuvaravii.github.io/
├── content/
│   ├── experience.md             # Career timeline page
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions auto-deploy
├── archetypes/
│   ├── default.md              # Default frontmatter template
│   ├── blog.md                 # Blog post template
│   └── projects.md             # Project showcase template
├── content/
│   ├── _index.md               # Homepage content
│   ├── about.md                # About/profile page
│   ├── blog/
│   │   ├── _index.md           # Blog listing page
│   │   └── (your posts here)
│   ├── projects/
│   │   ├── _index.md           # Projects listing page
│   │   └── (your project pages here)
│   └── frameworks/
│       ├── _index.md           # Frameworks/mental-models listing
│       └── (your frameworks here)
├── static/
│   ├── images/                 # Site images, project screenshots
│   └── resume.pdf              # Optional downloadable resume
├── layouts/                    # Custom layout overrides (if needed)
├── hugo.toml                   # Site configuration
└── README.md                   # Repo README
```

---

## 3. Content Sections

### 3.1 Homepage (Profile Mode)
PaperMod's "profile mode" — shows your name, a one-liner tagline, social links (GitHub, LinkedIn, X, email), and a profile image. Clean, no clutter. Visitors instantly know who you are.

### 3.2 Blog (`/blog/`)
Your technical writing. Categories:
- **Deep Dives** — Technical breakdowns of ML concepts, architectures, papers
- **Frameworks** — Mental models, thinking tools, decision frameworks
- **Build Logs** — What you built, why, what you learned
- **Thoughts** — Raw ideas, observations, opinions on AI/ML industry

Each post has: title, date, tags, categories, summary, reading time (auto-generated).

### 3.3 Projects (`/projects/`)
Each project gets its own page with:
- **Problem** — What problem does this solve?
- **Approach** — What technique/architecture did you use?
- **Stack** — Tools, libraries, frameworks
- **Results** — Metrics, outcomes, learnings
- **Links** — GitHub repo, demo, paper

### 3.4 Frameworks (`/frameworks/`)
Your original thinking. Mental models, decision frameworks, principles you operate by. This is what separates you from every other "I also do ML" portfolio.

### 3.5 About (`/about`)
Who you are. What you're building toward. Your engineering philosophy. Not a resume dump — a narrative.

---

## 4. Theme Configuration Highlights

```toml
# hugo.toml — key settings

baseURL = "https://yuvaravii.github.io/"
languageCode = "en-us"
title = "Yuvaravii"  # Change to your preferred display name
theme = "PaperMod"

[params]
  env = "production"
  description = "AI/ML Engineer — building, writing, thinking."
  author = "Yuvaravii"
  ShowReadingTime = true
  ShowShareButtons = false
  ShowPostNavLinks = true
  ShowBreadCrumbs = true
  ShowCodeCopyButtons = true
  ShowToc = true
  TocOpen = false
  defaultTheme = "auto"  # auto = follows system preference

  [params.homeInfoParams]
    Title = "Hey, I'm Yuvaravii"
    Content = "AI/ML Engineer. I build things, break things, and write about what I learn."

  [[params.socialIcons]]
    name = "github"
    url = "https://github.com/yuvaravii"
  [[params.socialIcons]]
    name = "linkedin"
    url = "https://linkedin.com/in/YOUR_LINKEDIN"
  [[params.socialIcons]]
    name = "email"
    url = "mailto:gceravishankar@gmail.com"

[menu]
  [[menu.main]]
    identifier = "blog"
    name = "Blog"
    url = "/blog/"
    weight = 10
  [[menu.main]]
    identifier = "projects"
    name = "Projects"
    url = "/projects/"
    weight = 20
  [[menu.main]]
    identifier = "frameworks"
    name = "Frameworks"
    url = "/frameworks/"
    weight = 30
  [[menu.main]]
    identifier = "experience"
    name = "Experience"
    url = "/experience/"
    weight = 35
  [[menu.main]]
    identifier = "about"
    name = "About"
    url = "/about/"
    weight = 40
  [[menu.main]]
    identifier = "search"
    name = "Search"
    url = "/search/"
    weight = 50
```

---

## 5. GitHub Actions — Auto-Deploy Pipeline

Every push to `main` → Hugo builds → deploys to GitHub Pages. Zero manual steps.

```yaml
# .github/workflows/deploy.yml
name: Deploy Hugo to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true
      - name: Build
        run: hugo --minify
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

## 6. Content Workflow

Your daily workflow after setup:

1. Write a `.md` file in the right folder (`content/blog/`, `content/projects/`, etc.)
2. Use the archetype frontmatter (auto-generated with `hugo new blog/my-post.md`)
3. `git add . && git commit -m "new post: title" && git push`
4. GitHub Actions builds and deploys automatically
5. Live in ~60 seconds

---

## 7. First Content — Launch Checklist

You need at minimum these pieces before the site is worth sharing:

| # | Content | Priority | Status |
|---|---|---|---|
| 1 | About page — who you are, what you're building | **Must have** | Not started |
| 2 | 1 project page — your best ML project | **Must have** | Not started |
| 3 | 1 blog post — a technical deep-dive or framework | **Must have** | Not started |
| 4 | Profile image | **Must have** | Not started |
| 5 | 2nd project page | Nice to have | Not started |
| 6 | 2nd blog post | Nice to have | Not started |

**Rule:** Don't share the site URL publicly until items 1-4 are done. An empty portfolio is worse than no portfolio.

---

## 8. What I'll Build When You Approve

Once you say go, I will:

1. Initialize the Hugo site in this repo
2. Install PaperMod as a git submodule
3. Create `hugo.toml` with full configuration
4. Set up the GitHub Actions workflow
5. Create all content archetypes (blog, project, framework templates)
6. Create placeholder pages for all sections
7. Create the About page scaffold
8. Test the build locally

**You will then need to:**
- Add your profile image
- Fill in the About page with your actual story
- Write your first project page
- Write your first blog post
- Update LinkedIn URL in config
- Enable GitHub Pages in repo settings (Settings → Pages → Source: GitHub Actions)

---

## 9. Future Enhancements (Not Now)

- Custom domain (yuvaravii.com or similar)
- Analytics (GoatCounter — privacy-respecting, free)
- Comments (giscus — GitHub-backed, no signup needed)
- RSS feed (Hugo generates this automatically)
- Newsletter integration
- Jupyter notebook rendering for ML content

---

*Blueprint by your project assistant. Approve to proceed with build.*
