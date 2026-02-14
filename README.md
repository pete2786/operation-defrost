# Operation Defrost

Civic engagement tool that helps people contact their federal representatives about issues that matter. Generates personalized email templates tailored to different political perspectives so you can communicate effectively regardless of who represents you.

**Live at [operationdefrost.app](https://operationdefrost.app)**

## Current Issues

- **ICE Accountability in Minneapolis** — Demanding justice for Renee Macklin Good and Alex Pretti, an end to racial profiling, humane detention conditions, and community reparations
- **Digital Privacy & Government Surveillance** — Pushing for GDPR-style privacy protections against phone tracking, facial recognition, and government surveillance overreach
- **Protect the Boundary Waters from Mining** — Opposing Congress lifting the 20-year mining ban near the BWCA for a Chilean-owned mining company

## How It Works

1. Pick an issue
2. Choose which talking points to include in your message
3. Select a tone that matches your representative's politics
4. Enter your ZIP code to find your reps
5. Copy the generated email and paste it into their official contact form

Everything runs in the browser. No data is collected or stored.

## Architecture

No build tools, no frameworks, no dependencies to install. Pure vanilla HTML, CSS, and JavaScript with Tailwind CDN.

```
index.html                          # Entire app (SPA with hash router)
data/
  issues.json                       # Issue index
  issues/{id}.json                  # Full issue data (content blocks, tones, meta)
issue/{id}/index.html               # Social sharing stubs (see below)
images/                             # Self-hosted issue images
CNAME                               # Custom domain config
```

Deployed via GitHub Pages from `main`. Push to main = deploy.

## Adding a New Issue

### 1. Create the issue JSON

Create `data/issues/{your-issue-id}.json`:

```json
{
  "id": "your-issue-id",
  "title": "Issue Title",
  "subtitle": "Short subtitle shown on the issue page",
  "urgentBanner": {
    "date": "February 2026",
    "text": "Urgent context with <strong>HTML</strong> support."
  },
  "contentBlocks": [
    {
      "id": "block-id",
      "label": "Short label shown on the checkbox",
      "category": "category-name",
      "defaultSelected": true,
      "contextText": "Longer explanation shown below the checkbox.",
      "templateFragment": "Text inserted into the email when this block is selected."
    }
  ],
  "tones": [
    {
      "id": "tone-id",
      "label": "Tone Name",
      "description": "Brief description of the tone",
      "color": "blue",
      "subject": "Email subject line",
      "intro": "Opening paragraph.\n\n{selected_blocks}\n\nTransition to asks:",
      "asks": [
        "First demand",
        "Second demand"
      ],
      "closing": "Closing paragraph.\n\nThank you."
    }
  ],
  "meta": {
    "ogTitle": "Operation Defrost - Issue Title",
    "ogDescription": "Description for social media previews.",
    "ogImage": "https://operationdefrost.app/images/your-image.jpg",
    "ogImageWidth": 1200,
    "ogImageHeight": 800,
    "shareText": "Text used when sharing on social media"
  }
}
```

Key details:
- `{selected_blocks}` in the tone intro is replaced with the concatenated `templateFragment` values of all selected content blocks
- `urgentBanner` can be `null` if there's no urgent context
- `defaultSelected` controls which blocks are checked by default
- `color` should be a Tailwind color name (blue, red, green, purple, etc.)

### 2. Add to the issue index

Add an entry to `data/issues.json`:

```json
{
  "id": "your-issue-id",
  "title": "Issue Title",
  "summary": "One-sentence summary for the issue card on the home page.",
  "image": "images/your-image.jpg",
  "date": "2026-02-14",
  "status": "active"
}
```

### 3. Add the image

Save a 1200px wide JPG to `images/your-image.jpg`. This is used for both the issue card on the home page and the hero image on the issue detail page.

### 4. Create the social sharing stub

Social media crawlers (Facebook, Twitter, iMessage) don't execute JavaScript. Since this is a hash-routed SPA, they'd always see the home page meta tags. The stub gives each issue its own static HTML with correct OG tags.

Create `issue/{your-issue-id}/index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Operation Defrost - Issue Title</title>
    <meta name="title" content="Operation Defrost - Issue Title">
    <meta name="description" content="Description for social previews.">
    <meta property="og:type" content="website">
    <meta property="og:url" content="https://operationdefrost.app/issue/your-issue-id/">
    <meta property="og:title" content="Operation Defrost - Issue Title">
    <meta property="og:description" content="Description for social previews.">
    <meta property="og:image" content="https://operationdefrost.app/images/your-image.jpg">
    <meta property="og:image:width" content="1200">
    <meta property="og:image:height" content="800">
    <meta property="twitter:card" content="summary_large_image">
    <meta property="twitter:url" content="https://operationdefrost.app/issue/your-issue-id/">
    <meta property="twitter:title" content="Operation Defrost - Issue Title">
    <meta property="twitter:description" content="Description for social previews.">
    <meta property="twitter:image" content="https://operationdefrost.app/images/your-image.jpg">
    <link rel="canonical" href="https://operationdefrost.app/#/issue/your-issue-id">
    <script>window.location.replace('https://operationdefrost.app/#/issue/your-issue-id');</script>
</head>
<body>
    <p>Redirecting to <a href="https://operationdefrost.app/#/issue/your-issue-id">Operation Defrost - Issue Title</a>...</p>
</body>
</html>
```

### 5. Push to main

```bash
git add data/ images/ issue/
git commit -m "Add new issue: your-issue-id"
git push
```

GitHub Pages deploys automatically.

## Local Development

```bash
python3 -m http.server 8080
# Open http://localhost:8080
```

## Rep Lookup

Uses the [whoismyrepresentative.com](https://whoismyrepresentative.com) API via CORS proxy. The proxy list is in `index.html` in the `lookupRep()` function. If rep lookup breaks, it's usually because a proxy went down — test alternatives and update the list.

## Privacy

No analytics, no tracking, no server-side processing. All data stays in the user's browser.

## License

This project is open source. Issue images are sourced from Wikimedia Commons.
