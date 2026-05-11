# Wild Petal Collective

Static marketing website for Wild Petal Collective, designed for GitHub Pages.

## Pages

- `index.html` — home page
- `design.html` — design studio page with intake forms
- `talismans.html` — talismans page with waitlist form
- `about.html` — about page
- `contact.html` — contact page

## Structure

```text
wild-petal-website/
  index.html
  design.html
  talismans.html
  about.html
  contact.html
  styles.css
  CNAME
  assets/
    home/
    talismans/
  copy/
    home.md
    design.md
    talismans.md
```

## Local Preview

Run a simple local server from the project root:

```bash
python3 -m http.server 8000
```

Then open `http://127.0.0.1:8000/`.

## Forms

- `design.html` submits to a Google Form endpoint.
- `talismans.html` submits to a Google Form endpoint.
- If the connected Google Forms change, update the form `action` URLs and any required `entry.*` field IDs in the HTML.

## GitHub Pages

1. Push the repository to GitHub.
2. In GitHub, open `Settings` -> `Pages`.
3. Publish from the `main` branch and the repository root.
4. Wait for the site to deploy.

## Custom Domain

This repo includes a `CNAME` file for:

`www.wildpetalcollective.com`
