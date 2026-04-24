---
name: api-docs-html
description: >
  Generate a beautiful, professional single-file HTML API documentation page inspired by Mintlify, ReadMe.io, and Scalar. Use this skill whenever the user wants to create, generate, or produce API documentation as an HTML file. Triggers include: "generate API docs", "create documentation for my API", "make a docs page", "document this API", "create HTML documentation", "gerar documentação da API", "criar docs HTML para API". Works from any source: OpenAPI/Swagger YAML or JSON specs, PDF documentation, plain text descriptions, or raw endpoint lists. Always use this skill — even for quick or simple API doc requests — because the output quality and structure far exceeds what ad-hoc HTML generation produces.
---

# API Docs HTML Skill

Generates a stunning, single-file HTML API documentation page inspired by Mintlify, ReadMe.io, and Scalar. The output is a self-contained `.html` file with no external dependencies — everything is inlined (CSS, JS, fonts via CDN).

## Input Sources

The skill handles any of these inputs:
- **OpenAPI/Swagger** — YAML or JSON spec (uploaded file or pasted content)
- **PDF** — existing documentation PDF (read with pdf-reading skill if needed)
- **Plain text / markdown** — endpoint descriptions, curl examples, etc.
- **Mixed** — combine multiple sources

## Step-by-Step Workflow

### Step 1 — Read the Source

**If input is a Swagger/OpenAPI file:**
- Parse YAML/JSON to extract: `info`, `servers`, `paths`, `components/schemas`, `securitySchemes`
- Build internal model: list of endpoints grouped by tag, schemas, auth info

**If input is a PDF:**
- Read the skill at `/mnt/skills/public/pdf-reading/SKILL.md` first
- Extract: endpoint names, methods, paths, parameters, request/response bodies, auth, descriptions

**If input is plain text or description:**
- Extract all endpoints, methods, parameters, descriptions, and examples the user provided
- If info is incomplete, infer reasonable structure and note assumptions in a comment

### Step 2 — Plan the Documentation Structure

Before generating, mentally organize:

```
Sidebar:
  - Introduction / Overview
  - Authentication
  - [Tag Group 1]
    - GET /endpoint-a
    - POST /endpoint-b
  - [Tag Group 2]
    - ...
  - Error Codes
  - Models / Schemas (if present)

Main content per endpoint:
  - Method badge + path
  - Description
  - Authentication requirement
  - Parameters (path, query, header, body)
  - Request body schema + example
  - Response codes + body schemas + examples
  - Code samples (curl, Python, JS)
```

### Step 3 — Generate the HTML

Generate a complete, self-contained HTML file following the design system below.

---

## Design System

### Visual Style

Aim for a **dark, modern, developer-focused aesthetic** inspired by Scalar and Mintlify dark mode:
- Dark sidebar (`#0f1117`) with light text
- Main content area with slightly lighter background (`#161b27`)
- Cards/panels with subtle borders (`#1e2535`)
- Method badges with distinctive colors (see below)
- Clean sans-serif typography via Google Fonts (e.g., `Inter`, `JetBrains Mono` for code)

### Color Palette (CSS Variables)

```css
:root {
  --bg-main: #0f1117;
  --bg-content: #161b27;
  --bg-card: #1a2133;
  --bg-code: #0d1117;
  --border: #1e2d45;
  --border-subtle: #1a2535;
  --text-primary: #e2e8f0;
  --text-secondary: #94a3b8;
  --text-muted: #64748b;
  --accent: #3b82f6;
  --accent-hover: #2563eb;

  /* Method badges */
  --get:    #10b981;  /* green  */
  --post:   #3b82f6;  /* blue   */
  --put:    #f59e0b;  /* amber  */
  --patch:  #8b5cf6;  /* purple */
  --delete: #ef4444;  /* red    */
  --head:   #06b6d4;  /* cyan   */
  --options:#64748b;  /* slate  */

  --font-sans: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace;
  --radius: 8px;
  --radius-sm: 4px;
}
```

### Layout Structure

```
┌─────────────────────────────────────────────────────────────┐
│  TOPBAR  [Logo + API name]          [version badge] [search]│
├──────────────┬──────────────────────────────────────────────┤
│              │                                              │
│   SIDEBAR    │              CONTENT AREA                    │
│   (260px)    │                                              │
│              │   Endpoint card                              │
│  • Overview  │   ┌──────────────────────────────────────┐   │
│  • Auth      │   │ GET  /users/{id}                     │   │
│  ▼ Users     │   │ Description...                       │   │
│    GET /users│   │                                      │   │
│    POST/users│   │ Parameters | Body | Responses        │   │
│  ▼ Orders    │   └──────────────────────────────────────┘   │
│    ...       │                                              │
└──────────────┴──────────────────────────────────────────────┘
```

### Required UI Components

**1. Sidebar Navigation**
- Fixed left sidebar, 260px wide
- API name + version at top
- Collapsible tag groups with arrow toggle
- Active link highlight with left border accent
- Smooth scroll to section on click

**2. Method Badge**
```html
<span class="method-badge method-get">GET</span>
<span class="method-badge method-post">POST</span>
<!-- etc -->
```
Pill-shaped, uppercase, monospace font, colored background.

**3. Endpoint Card**
Each endpoint gets a card with:
- Header: method badge + path (monospace) + short summary
- Description paragraph
- Tabbed sections: Parameters / Request Body / Responses
- Code examples block (curl by default; add Python/JS if space allows)

**4. Parameter Table**
```
| Name       | In    | Type    | Required | Description         |
|------------|-------|---------|----------|---------------------|
| id         | path  | string  | ✓        | User identifier     |
| limit      | query | integer |          | Max results (1-100) |
```

**5. Code Blocks**
- Syntax highlighted (use Prism.js from CDN or hand-roll with CSS classes)
- Copy button (clipboard icon, JS onClick)
- Language label (curl / json / python)

**6. Response Section**
- Status code badge (200 green, 4xx yellow, 5xx red)
- Schema display (collapsible JSON tree or table)
- Example response in code block

**7. Authentication Section**
- Dedicated section before endpoint groups
- Explain the auth scheme (Bearer, API Key, OAuth2, etc.)
- Show example header/parameter

**8. Search**
- Simple JS filter on sidebar links
- Input at top of sidebar or in topbar

### Code Example Generation

For every endpoint, auto-generate a curl example:

```bash
curl -X {METHOD} '{baseUrl}{path}' \
  -H 'Authorization: Bearer {token}' \
  -H 'Content-Type: application/json' \
  -d '{
    "key": "value"
  }'
```

Replace placeholders with actual parameter names from the spec.

---

## HTML File Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{API Name} — API Reference</title>
  <!-- Google Fonts -->
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
  <!-- Prism.js for syntax highlight (optional but recommended) -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/themes/prism-tomorrow.min.css">
  <style>
    /* All CSS here — no external stylesheets */
    /* Use CSS variables from design system above */
  </style>
</head>
<body>
  <!-- Topbar -->
  <header class="topbar">...</header>

  <div class="layout">
    <!-- Sidebar -->
    <nav class="sidebar">
      <div class="sidebar-search">...</div>
      <ul class="nav-list">
        <!-- Dynamically generated nav items -->
      </ul>
    </nav>

    <!-- Main content -->
    <main class="content">
      <!-- #overview section -->
      <!-- #authentication section -->
      <!-- #tag-group sections with endpoint cards -->
      <!-- #models section (if schemas present) -->
      <!-- #errors section -->
    </main>
  </div>

  <!-- Prism.js -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/prism.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-bash.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-json.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-python.min.js"></script>
  <script>
    /* All JS here:
       - Sidebar toggle (collapsible groups)
       - Search filter
       - Copy button logic
       - Active section highlight on scroll (IntersectionObserver)
       - Smooth scroll
    */
  </script>
</body>
</html>
```

---

## Quality Checklist

Before saving the file, verify:
- [ ] All endpoints from the source are documented
- [ ] Every endpoint has: method, path, description, parameters, at least one response
- [ ] curl example generated for every endpoint
- [ ] Sidebar links all work (href="#section-id" matching actual IDs)
- [ ] Copy buttons functional
- [ ] Sidebar search filters correctly
- [ ] Page looks good at 1280px width
- [ ] No broken references to external files (must be 100% self-contained)
- [ ] Code blocks have correct syntax highlighting class (`language-bash`, `language-json`, etc.)

---

## Output

Save the final file as:
```
/mnt/user-data/outputs/{api-name}-docs.html
```

Where `{api-name}` is derived from the API title (lowercase, hyphenated). Then call `present_files` to deliver it to the user.

After presenting, give a short summary:
- Number of endpoints documented
- Tags/groups identified
- Auth method detected
- Any assumptions made (if source was incomplete)
