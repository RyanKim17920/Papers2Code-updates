# Target Repository Workflow Examples

This directory contains example GitHub Actions workflow files for target repositories (Repository B) that receive content updates from the Papers2Code-updates repository.

## Available Examples

### 1. `basic-example.yml`
**Use this if:** You want a simple, customizable starting point

A minimal workflow template that shows the essential steps:
- Receiving dispatch events
- Cloning content from source repository
- Placeholder steps for build and deployment

Perfect for custom build systems or as a learning example.

---

### 2. `jekyll-github-pages.yml`
**Use this if:** Your site uses Jekyll and GitHub Pages

A complete workflow for Jekyll sites that:
- Syncs markdown files to Jekyll's `_posts` directory
- Builds the Jekyll site using Ruby and Bundler
- Deploys automatically to GitHub Pages

**Requirements:**
- Jekyll site with `Gemfile` and `_config.yml`
- GitHub Pages enabled in repository settings

---

### 3. `hugo-example.yml`
**Use this if:** Your site uses Hugo static site generator

A complete workflow for Hugo sites that:
- Syncs markdown files to Hugo's `content/posts` directory
- Builds using Hugo (with extended version support)
- Deploys to GitHub Pages

**Requirements:**
- Hugo site with `config.toml` or `config.yaml`
- Hugo theme configured

---

### 4. `nextjs-example.yml`
**Use this if:** Your site uses Next.js

A flexible workflow for Next.js sites that:
- Syncs markdown files to Next.js content directory
- Builds the Next.js application
- Supports deployment to Vercel or GitHub Pages (static export)

**Requirements:**
- Next.js project with `package.json`
- For GitHub Pages: Static export configured (`next export`)
- For Vercel: Vercel token and project secrets configured

---

### 5. `advanced-multi-source.yml`
**Use this if:** You aggregate content from multiple repositories

An advanced workflow that:
- Identifies the source repository
- Routes content to different directories based on source
- Generates metadata and content indexes
- Supports multiple content categories

**Use cases:**
- Aggregating papers, tutorials, and blog posts from different repos
- Multi-author sites with separate content repositories
- Content hubs that combine multiple sources

---

## How to Use These Examples

### Step 1: Choose an Example
Select the example that best matches your site's technology stack.

### Step 2: Copy to Your Repository
Copy the chosen workflow file to your target repository at:
```
.github/workflows/content-update.yml
```

### Step 3: Customize
Edit the workflow file to match your specific needs:
- Update directory paths
- Modify build commands
- Configure deployment settings
- Add any custom processing steps

### Step 4: Configure Secrets (if needed)
Some workflows require secrets:

#### For Vercel deployment:
- `VERCEL_TOKEN`: Your Vercel authentication token
- `VERCEL_ORG_ID`: Your Vercel organization ID
- `VERCEL_PROJECT_ID`: Your Vercel project ID

#### For custom deployments:
- Add any deployment-specific tokens or credentials

### Step 5: Test
Manually trigger the workflow to test:
1. Go to Actions tab in your repository
2. Select the workflow
3. Click "Run workflow"

Or trigger from the source repository by pushing a change to `_posts/`.

---

## Common Customizations

### Changing Content Directories

If your site uses different directory structures, update the copy commands:

```yaml
# Instead of:
cp -r _posts/* ../content/posts/

# Use your structure:
cp -r _posts/* ../src/content/blog/
```

### Adding Content Validation

Add validation before building:

```yaml
- name: Validate content
  run: |
    npm install -g markdownlint-cli
    markdownlint content/**/*.md
    python scripts/validate-frontmatter.py
```

### Selective Content Sync

Only sync specific categories:

```yaml
- name: Sync tutorials only
  run: |
    for file in source-content/_posts/*.md; do
      if grep -q "categories: \[.*tutorials.*\]" "$file"; then
        cp "$file" content/posts/
      fi
    done
```

### Content Transformation

Transform markdown before building:

```yaml
- name: Transform content
  run: |
    # Convert Jekyll frontmatter to Hugo format
    python scripts/convert-frontmatter.py
    
    # Process images
    python scripts/optimize-images.py
```

---

## Workflow Permissions

Ensure your workflow has the necessary permissions:

### For GitHub Pages deployment:
```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

### For committing back to the repo:
```yaml
permissions:
  contents: write
```

### For creating releases:
```yaml
permissions:
  contents: write
```

---

## Troubleshooting

### Workflow doesn't trigger
- Verify the source repository has `DISPATCH_TOKEN` configured
- Check the event type matches: `types: [content-update]`
- Look at workflow logs in both repositories

### Build fails
- Check that dependencies are installed correctly
- Verify directory paths in copy commands
- Review build logs for specific error messages

### Deployment fails
- Verify deployment secrets are configured
- Check workflow permissions
- Ensure deployment target is accessible

---

## Testing Your Workflow

### Test locally first:
```bash
# Clone your target repo
git clone https://github.com/your-org/your-repo.git
cd your-repo

# Manually fetch content
git clone https://github.com/RyanKim17920/Papers2Code-updates.git source
cp -r source/_posts/* content/posts/

# Test your build
npm run build  # or your build command
```

### Test the full workflow:
1. Create a test markdown file in Papers2Code-updates
2. Push to trigger the workflow
3. Monitor workflow runs in both repositories
4. Verify content appears on your deployed site

---

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Repository Dispatch Events](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event)
- [Integration Guide](../../INTEGRATION_GUIDE.md) - Comprehensive setup guide

---

## Support

If you encounter issues:
1. Check the [Integration Guide](../../INTEGRATION_GUIDE.md)
2. Review workflow logs in GitHub Actions
3. Open an issue in the Papers2Code-updates repository
