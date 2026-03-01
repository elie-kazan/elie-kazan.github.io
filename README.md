# eliekazan.github.io

Personal blog & landing page built with [Hugo](https://gohugo.io/) and hosted on GitHub Pages.

## Quick Start

### 1. Install Hugo

```bash
# macOS
brew install hugo

# Windows
choco install hugo-extended

# Linux (Debian/Ubuntu)
sudo apt install hugo
```

### 2. Run Locally

```bash
hugo server -D
```

Visit `http://localhost:1313` to see your site.

### 3. Create a New Blog Post

```bash
hugo new posts/my-new-post.md
```

This creates a new Markdown file in `content/posts/`. Edit it, set `draft: false` when ready, and push.

### 4. Deploy

Just push to `main`. The GitHub Action builds and deploys automatically.

```bash
git add .
git commit -m "New blog post"
git push
```

## Project Structure

```
├── archetypes/         # Templates for new content
├── content/
│   └── posts/          # Blog posts (Markdown)
├── layouts/
│   ├── _default/       # Base template, single post, list page
│   ├── partials/       # Nav, footer (shared components)
│   └── index.html      # Landing page
├── static/
│   └── css/style.css   # All styles
├── .github/workflows/  # Auto-deploy action
└── hugo.toml           # Site config
```

## Customization

- **Edit landing page content**: `layouts/index.html`
- **Change styles**: `static/css/style.css`
- **Update personal info**: `hugo.toml` (email, GitHub, LinkedIn)
- **Add blog post emoji**: Set `emoji: "🔍"` in post front matter
