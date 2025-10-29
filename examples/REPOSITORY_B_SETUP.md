# Setting Up Repository B: Quick Reference

This is a quick reference guide for setting up a target repository (Repository B) to receive content updates from Papers2Code-updates.

## Prerequisites

✅ Papers2Code-updates has `DISPATCH_TOKEN` and `TARGET_REPOSITORIES` configured  
✅ You have admin access to Repository B  
✅ You know what static site generator or framework Repository B uses  

## Step-by-Step Setup

### 1️⃣ Choose Your Workflow Template

Based on your site's technology, select the appropriate workflow template:

| Your Technology | Use This Template |
|----------------|-------------------|
| Jekyll | `target-repository-workflows/jekyll-github-pages.yml` |
| Hugo | `target-repository-workflows/hugo-example.yml` |
| Next.js | `target-repository-workflows/nextjs-example.yml` |
| Custom/Other | `target-repository-workflows/basic-example.yml` |
| Multiple Sources | `target-repository-workflows/advanced-multi-source.yml` |

### 2️⃣ Copy Workflow to Repository B

```bash
# In your Repository B
mkdir -p .github/workflows
cp [chosen-template].yml .github/workflows/content-update.yml
```

Or manually create `.github/workflows/content-update.yml` and paste the template content.

### 3️⃣ Customize the Workflow

Edit the workflow file to match your repository structure:

#### Update content paths:
```yaml
# Change this:
cp -r _posts/* ../content/posts/

# To match your structure:
cp -r _posts/* ../src/content/blog/
```

#### Update build commands:
```yaml
# Change this:
run: npm run build

# To your build command:
run: bundle exec jekyll build
# or: hugo --minify
# or: python build.py
```

#### Update deployment:
```yaml
# Configure your deployment method
# GitHub Pages, Vercel, Netlify, etc.
```

### 4️⃣ Set Workflow Permissions

In Repository B, go to:
**Settings → Actions → General → Workflow permissions**

Select:
- ✅ **Read and write permissions** (if deploying to GitHub Pages or committing files)
- OR ✅ **Read repository contents** (if only building and deploying elsewhere)

### 5️⃣ Test the Integration

#### Option A: Manual test in Repository B
```bash
# Go to Actions tab
# Select "Rebuild on Content Update" workflow
# Click "Run workflow"
# Check logs for any errors
```

#### Option B: End-to-end test
```bash
# In Papers2Code-updates repository:
# 1. Create a test post in _posts/
# 2. Commit and push
# 3. Check Actions tab in both repos
# 4. Verify site rebuilds with new content
```

---

## Minimal Required Workflow

If you're setting up manually, here's the absolute minimum needed:

```yaml
name: Rebuild on Content Update

on:
  repository_dispatch:
    types: [content-update]  # Must match!

jobs:
  rebuild:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Get content
        run: |
          REPO="${{ github.event.client_payload.repository }}"
          SHA="${{ github.event.client_payload.sha }}"
          git clone https://github.com/$REPO.git source
          cd source && git checkout $SHA
          cp -r _posts/* ../[YOUR-CONTENT-DIR]/
      
      - name: Build
        run: [YOUR-BUILD-COMMAND]
      
      - name: Deploy
        run: [YOUR-DEPLOY-COMMAND]
```

---

## Common Structures for Repository B

### Jekyll Site
```
your-jekyll-site/
├── .github/workflows/content-update.yml  ← Add this
├── _config.yml
├── _posts/                               ← Content copied here
├── _layouts/
└── Gemfile
```

### Hugo Site
```
your-hugo-site/
├── .github/workflows/content-update.yml  ← Add this
├── config.toml
├── content/
│   └── posts/                            ← Content copied here
└── themes/
```

### Next.js Site
```
your-nextjs-site/
├── .github/workflows/content-update.yml  ← Add this
├── package.json
├── pages/
├── content/
│   └── posts/                            ← Content copied here
└── public/
```

### Custom Site
```
your-site/
├── .github/workflows/content-update.yml  ← Add this
├── [your-content-directory]/             ← Content copied here
├── [your-build-files]
└── [your-config-files]
```

---

## What Data Does Repository B Receive?

When content is updated in Papers2Code-updates, Repository B receives:

```json
{
  "event_type": "content-update",
  "client_payload": {
    "repository": "RyanKim17920/Papers2Code-updates",
    "ref": "refs/heads/main",
    "sha": "abc123..."
  }
}
```

### Accessing in Your Workflow:

```yaml
${{ github.event.client_payload.repository }}  # Source repo name
${{ github.event.client_payload.ref }}         # Branch reference
${{ github.event.client_payload.sha }}         # Commit SHA (use this!)
```

**Important:** Always use the `sha` to checkout a specific commit, not just clone the main branch. This ensures you get the exact content that triggered the update.

---

## Troubleshooting

### ❌ Workflow not triggering

**Check:**
1. Is the workflow file at `.github/workflows/content-update.yml`?
2. Does it have `on.repository_dispatch.types: [content-update]`?
3. Is Repository B listed in Papers2Code-updates' `TARGET_REPOSITORIES` secret?
4. Does the `DISPATCH_TOKEN` have `repo` scope?

**Debug:**
- Check Papers2Code-updates Actions logs for API response codes
- Look for error messages when triggering dispatch

### ❌ Content not syncing

**Check:**
1. Are the source and destination paths correct?
2. Does the `_posts` directory exist in Papers2Code-updates?
3. Are you using the commit SHA to checkout?

**Debug:**
- Add `ls -la` commands to see what files exist
- Print the cloned directory structure

### ❌ Build failing

**Check:**
1. Are all dependencies installed?
2. Are build commands correct?
3. Is the content in the right location?

**Debug:**
- Run build locally after manually copying content
- Check build logs for specific error messages

---

## Next Steps

1. ✅ Set up the workflow in Repository B
2. ✅ Test with a sample content push
3. ✅ Verify your site rebuilds correctly
4. ✅ Configure automatic deployments (if not already)
5. ✅ Set up notifications for workflow failures (optional)

---

## Need More Help?

- 📖 [Integration Guide](../INTEGRATION_GUIDE.md) - Comprehensive guide with detailed explanations
- 📋 [Example Workflows](target-repository-workflows/) - More templates and examples
- 💬 Open an issue in Papers2Code-updates repository

---

## Summary Checklist

Use this to verify your setup:

**Repository A (Papers2Code-updates):**
- [ ] `DISPATCH_TOKEN` secret configured
- [ ] `TARGET_REPOSITORIES` includes Repository B
- [ ] Workflow triggers on `_posts/*.md` changes

**Repository B (Your Site):**
- [ ] Workflow file exists at `.github/workflows/content-update.yml`
- [ ] Event type is `content-update`
- [ ] Content paths match your structure
- [ ] Build commands are correct
- [ ] Deployment is configured
- [ ] Workflow permissions are set

**Testing:**
- [ ] Manual workflow run succeeds
- [ ] Content push triggers rebuild
- [ ] Site updates with new content
- [ ] Deployment completes successfully
