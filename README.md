# Wild Petal Collective — Static Marketing Website

This project is a static, mobile-responsive marketing website prototype for Wild Petal Collective, ready for GitHub Pages deployment.

## Project Structure

```text
wild-petal-website/
  index.html
  design.html
  talismans.html
  styles.css
  assets/
    images/
      .gitkeep
    mockups/
      .gitkeep
  copy/
    home.md
    design.md
    talismans.md
  README.md
```

## What Each File Does

- `index.html` — umbrella brand homepage
- `design.html` — plant styling and interiors services page + Design leads form
- `talismans.html` — non-technical wearable concept page + Talismans waitlist form
- `styles.css` — shared styling system and responsive layout
- `copy/*.md` — editable source copy for each page
- `assets/images/` — production website images
- `assets/mockups/` — visual references and draft mockups

## Deploy to GitHub Pages

1. Create a new GitHub repository named **WPC Website** under your `WPC_resonance` account.
2. Upload the full `wild-petal-website` contents to the repository root (not inside another nested folder).
3. In GitHub, go to **Settings → Pages**.
4. Under **Build and deployment**, set:
   - **Source**: `Deploy from a branch`
   - **Branch**: `main` (or `master`), folder `/ (root)`
5. Save, then wait for GitHub Pages to publish.
6. Open the generated Pages URL and verify navigation and forms.

## Configure Email Capture (Two Lists)

This site uses static HTML form posts with two separate endpoints:

- Design leads form endpoint is in `design.html`
- Talismans waitlist endpoint is in `talismans.html`

### Steps

1. Create two forms in your form provider account (example: Formspree):
   - `Design Leads`
   - `Talismans Waitlist`
2. Copy each generated form endpoint URL.
3. Replace placeholders:
   - `https://formspree.io/f/your-design-form-id` in `design.html`
   - `https://formspree.io/f/your-talismans-form-id` in `talismans.html`
4. In your provider automations:
   - Route Design submissions to your client follow-up workflow.
   - Route Talismans submissions to your broadcast/waitlist workflow.

## Replace Images

1. Add final images to `assets/images/`.
2. Keep filenames or update image `src` values in HTML pages.
3. Recommended export sizes:
   - Hero images: ~1800px wide JPG/WEBP
   - Card images: ~1000px wide JPG/WEBP
4. Compress images for speed.

## Edit Website Text

1. Edit source messaging in `copy/home.md`, `copy/design.md`, and `copy/talismans.md`.
2. Copy final text into corresponding HTML sections.
3. Commit changes and push to GitHub; Pages redeploys automatically.

## Compliance Summary

- Static files only (HTML/CSS)
- No backend code
- No database in the project
- No analytics/tracking scripts
- No API integrations beyond static form-post endpoints
- Talismans messaging remains non-medical and non-diagnostic

