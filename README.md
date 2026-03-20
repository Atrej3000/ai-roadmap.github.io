# GenAI Certificate Roadmap 2026

A single-file interactive HTML application mapping 45+ free and paid Generative AI certificates from Google, Microsoft, Anthropic, AWS, IBM, NVIDIA, Databricks, Hugging Face, Cisco, DeepLearning.AI, MIT, and Oxford — with AI-powered tools for personalized learning planning.

---

## Features

### Filtering & Discovery
- **Tier filter** — Tier 1 (Free + Badge), Tier 2 (Free, no badge), Tier 3 (Paid)
- **Role filter** — Developer, Manager, Researcher, Beginner
- **Topic filter** — GenAI, LLMs, Prompt Engineering, RAG, Agents, Fine-tuning, Ethics, Cloud AI, Strategy, MCP, Transformers
- **Time slider** — filter to courses completable within N hours
- **Search** — matches course title, provider, and topic keywords

### Per-Card Metadata
- **Difficulty / prerequisites** — color-coded: green = no prereqs, blue = Python needed, orange = ML background, purple = Cloud experience
- **Salary / ROI data** — annotated on paid certifications where data is available
- **Last verified date** — every card shows when availability was last confirmed
- **Topic tags** — clickable pills that activate topic filters
- **Next steps** — appears after marking a course complete, linking to 2 recommended follow-up courses

### Progress Tracking
- **Checkbox per course** — persists to `localStorage` across sessions
- **Progress bar** — shows X of N completed in the header
- **Clear progress** button

### Sharing
- **Shareable URL** — encodes completed course IDs as base64 in the URL hash (`#p=...`)
- Loading a shared URL shows a banner letting the recipient save or dismiss the imported progress
- **Print / PDF** — clean white `@media print` stylesheet strips UI chrome

### AI Tools (Anthropic API)
All five AI tools call `claude-sonnet-4-20250514` via the Anthropic `/v1/messages` endpoint. No API key configuration is required when running inside claude.ai artifacts; in standalone deployments, see the Configuration section below.

| Tool | What it does |
|------|-------------|
| **Find My Path** | 3-step decision tree (role → time → goal) → scored recommendations from the full course catalog |
| **Study Plan** | Input hours/week + target date → week-by-week schedule of uncompleted courses |
| **CV Matcher** | Paste your CV → Claude returns the 5 certificates that best address your skill gaps, with reasons |
| **Job Analyzer** | Paste a job description → Claude identifies high/medium relevance certificates, highlights them in the grid |
| **Verify Availability** | Claude reviews all free courses and flags any that may have changed, been paywalled, or discontinued |

The CV Matcher and Job Analyzer highlight matching cards directly in the course grid after analysis.

---

## File Structure

This is a zero-dependency single HTML file. No build step, no npm, no framework.

```
free-ai-certificates-roadmap.html   # The entire application
README.md                           # This file
```

All JavaScript is vanilla ES5-compatible. All CSS uses custom properties. The only external dependency is Google Fonts (loaded via CDN), which degrades gracefully if offline.

---

## Configuration

### Running locally
Open `free-ai-certificates-roadmap.html` directly in any modern browser. All filtering, progress tracking, sharing, and the decision tree work offline.

### AI features in standalone deployment
The five AI tools call the Anthropic API directly from the browser. When running outside of claude.ai, you need to either:

**Option A — proxy (recommended for production):** Route `/v1/messages` calls through a backend proxy that injects your API key server-side. Update the fetch URL in the `callClaude()` function:

```js
// Replace this line in callClaude():
return fetch("https://api.anthropic.com/v1/messages", { ... })

// With your proxy endpoint:
return fetch("/api/claude", { ... })
```

**Option B — direct (local/dev only):** Inject your key directly. Find the `callClaude` function and add the header:

```js
headers: {
  "Content-Type": "application/json",
  "x-api-key": "sk-ant-...",
  "anthropic-version": "2023-06-01"
}
```

> Never expose an API key in client-side code in a publicly accessible deployment.

### Model
The application uses `claude-sonnet-4-20250514`. To change it, find and replace the model string in the `callClaude()` function.

---

## Course Data

All courses are defined in the `COURSES` array at the top of the script block. Each entry follows this schema:

```js
{
  id: "unique-id",          // Used for progress tracking and next-steps linking
  title: "Course Title",
  prov: "Provider Name",
  pc: "#hexcolor",          // Provider brand color for the card accent
  tier: 1,                  // 1 = Free+Badge, 2 = Free only, 3 = Paid
  h: 2,                     // Estimated hours to complete
  cred: "Credly Badge",     // Credential type description
  url: "https://...",       // Direct link to the course
  diff: "none",             // "none" | "python" | "ml" | "cloud"
  tp: ["genai","llms"],     // Topic tags (see TOPIC_LABELS for valid values)
  rl: ["dev","beginner"],   // Roles (dev | manager | researcher | beginner)
  sal: "~20% salary lift",  // Salary data string, or null
  vf: "Mar 2026",           // Last verified date
  sprint: true,             // Whether to include in sprint recommendations
  tip: "LinkedIn tip text", // Multiline string shown in the LinkedIn tip modal
  nx: ["other-id"],         // IDs of recommended next courses (shown when done)
  tg: ["Top Pick"],         // Extra badge labels on the card
  cost: "$150 exam"         // Cost string for paid courses, or undefined
}
```

To add a new course, append an entry to the `COURSES` array. No other changes required.

---

## Sharing URL Format

The share feature encodes progress as a URL hash:

```
https://your-domain.com/roadmap.html#p=WyJkYi1nZW5haSIsIm52aWRpYS1nZW5haSJd
```

The hash value is `btoa(JSON.stringify([...completedCourseIds]))`. It decodes to an array of course ID strings. The app reads this on load, imports the progress, and shows a banner giving the viewer the option to save it as their own.

---

## Providers Covered

| Provider | Tier | Credentials issued |
|----------|------|-------------------|
| Google Cloud | 1 | Google Skill Badges, Completion Badges |
| Microsoft / LinkedIn | 1 | LinkedIn Professional Certificates, Applied Skills Badges |
| Anthropic | 1 | Official Certificates |
| AWS | 1 | Credly Badges |
| IBM | 1 | IBM Credly Badges |
| Databricks | 1 + 3 | Digital Badges, Paid Certification |
| NVIDIA | 1 | DLI Certificates |
| Hugging Face | 1 | PDF Certificates |
| Cisco | 1 | Credly Digital Badges |
| DeepLearning.AI | 2 + 3 | Accomplishments (free), Coursera Certificates (paid) |
| Univ. of Helsinki | 2 | PDF Certificate |
| fast.ai | 2 | No certificate |
| AWS (paid exams) | 3 | Credly Badges |
| Microsoft (paid exams) | 3 | Microsoft Certified Badges |
| Google (paid exams) | 3 | Credly Badges |
| MIT | 3 | MIT Professional Certificate |
| Oxford | 3 | Oxford Certificate |

---

## Browser Support

Tested in Chrome 120+, Firefox 121+, Safari 17+, Edge 120+. Requires:
- CSS custom properties
- `localStorage`
- `navigator.clipboard` (for copy-to-clipboard; degrades gracefully)
- `fetch` (for AI features)
- `btoa` / `atob` (for share URL encoding)

No polyfills included. IE is not supported.

---

## License

MIT — free to use, modify, and redistribute. If you deploy this publicly, attribution appreciated but not required.

Course availability, pricing, and credential details change frequently. Verify current status directly with each provider before committing time or money.

---

## Changelog

**March 2026 — v1.0**
- Initial release with 45+ courses across 3 tiers
- All 16 features: filters, progress tracker, sharing, print, decision tree, study plan, CV matcher, job analyzer, auto-refresh, LinkedIn tips, salary data, prerequisites, topic tags, next steps, last verified dates, role-based filtering
