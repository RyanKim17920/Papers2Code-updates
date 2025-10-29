# Frequently Asked Questions (FAQ)

## General Questions

### Q: What is this repository for?

**A:** This is a content repository that stores blog posts and papers as Markdown files. When you add or update content here, it automatically notifies other repositories (like your website) to rebuild and display the new content.

### Q: Why separate content from the website?

**A:** Separating content from the website has several benefits:
- ✅ Content authors don't need to know how the website works
- ✅ Multiple websites can use the same content
- ✅ Content is version-controlled independently
- ✅ Easier to change website technology without losing content
- ✅ Clear separation of concerns

### Q: Can multiple websites use the same content repository?

**A:** Yes! That's one of the main benefits. You can configure multiple target repositories in the `TARGET_REPOSITORIES` secret, and they'll all receive notifications when content updates.

---

## Setup Questions

### Q: Do I need to set up both Repository A and Repository B?

**A:** 
- **Repository A** (this repo): Only needs secrets configured (`DISPATCH_TOKEN` and `TARGET_REPOSITORIES`)
- **Repository B** (your website): Needs a workflow file added (`.github/workflows/content-update.yml`)

Both are required for the connection to work.

### Q: What if I don't have a Personal Access Token?

**A:** You'll need to create one:
1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Select the `repo` scope
4. Generate and copy the token
5. Add it as a secret in Repository A

See the [Integration Guide](INTEGRATION_GUIDE.md) for detailed steps.

### Q: Can I test this without affecting my production site?

**A:** Yes! You can:
1. Create a test/staging repository as your target
2. Add it to `TARGET_REPOSITORIES` along with your production site
3. Test changes there first
4. Once verified, the production site will also rebuild

Or use manual workflow triggers in Repository B to control when rebuilds happen.

### Q: Do I need to make Repository A public?

**A:** No, it can be private. However, if Repository A is private:
- Your `DISPATCH_TOKEN` must have access to it
- Repository B needs authentication to clone it (use a PAT in the clone step)

For public content, a public repository makes it easier for Repository B to fetch content.

---

## Technical Questions

### Q: How does Repository B know when content changes?

**A:** 
1. When you push changes to `_posts/` in Repository A, a GitHub Actions workflow triggers
2. This workflow calls GitHub's Repository Dispatch API
3. The API sends a `repository_dispatch` event to Repository B
4. Repository B has a workflow listening for this event type
5. When received, Repository B's workflow runs to fetch and rebuild

It's similar to a webhook, but handled entirely through GitHub's infrastructure.

### Q: What happens if Repository B is offline or broken?

**A:** The dispatch event will still be sent, but:
- If Repository B's workflow is broken, it will fail (check workflow logs)
- If Repository B is temporarily unavailable, GitHub will retry
- The dispatch event itself is fire-and-forget; Repository A doesn't wait for B to complete

**Best practice:** Monitor workflow runs in Repository B and set up notifications for failures.

### Q: Can I control which files trigger the rebuild?

**A:** Yes! In Repository A's workflow (`.github/workflows/trigger-website-build.yml`), you can modify the `paths` filter:

```yaml
on:
  push:
    branches:
      - main
    paths:
      - '_posts/**.md'          # Only trigger on posts
      - '!_posts/*draft*'       # Ignore drafts
```

### Q: How do I send additional data to Repository B?

**A:** Modify the `client_payload` in Repository A's workflow:

```yaml
curl ... -d '{
  "event_type":"content-update",
  "client_payload": {
    "repository":"'"$GITHUB_REPOSITORY"'",
    "ref":"'"$GITHUB_REF"'",
    "sha":"'"$GITHUB_SHA"'",
    "custom_field":"custom_value",    # Add custom data
    "changed_files":["file1.md"]      # Add file list
  }
}'
```

Then access in Repository B:
```yaml
${{ github.event.client_payload.custom_field }}
```

### Q: Can I trigger rebuilds manually?

**A:** Yes, both workflows support manual triggering:

**Repository A:**
- Go to Actions → "Trigger Website Build" → Run workflow

**Repository B:**
- Go to Actions → "Rebuild on Content Update" → Run workflow

---

## Content Questions

### Q: What format should my blog posts use?

**A:** Use Markdown with YAML frontmatter:

```markdown
---
title: "Your Post Title"
date: 2025-10-28
author: "Author Name"
categories: [category1, category2]
tags: [tag1, tag2]
description: "Brief description"
---

# Your content here

Write your post using Markdown...
```

See [`_posts/sample-update.md`](_posts/sample-update.md) for a complete example.

### Q: Can I include images in my posts?

**A:** Yes, but you need to handle image storage:

**Option 1:** Store images in Repository A
```markdown
![Image](../images/my-image.png)
```
And copy the `images/` folder in Repository B's workflow.

**Option 2:** Use external image hosting
```markdown
![Image](https://example.com/images/my-image.png)
```

**Option 3:** Use GitHub's asset hosting
```markdown
![Image](https://github.com/user/repo/raw/main/images/my-image.png)
```

### Q: Can I have draft posts that don't trigger rebuilds?

**A:** Yes, several approaches:

**Option 1:** Use a separate `_drafts/` folder
- Only files in `_posts/` trigger the workflow
- Move to `_posts/` when ready to publish

**Option 2:** Use frontmatter flag
```yaml
---
published: false
---
```
And filter in Repository B based on this flag.

**Option 3:** Use branches
- Create drafts in a `draft` branch
- Merge to `main` when ready to publish

---

## Workflow Questions

### Q: How long does it take for my website to update?

**A:** Typically 2-5 minutes:
1. Push to Repository A: ~10 seconds
2. Repository A workflow runs: ~30 seconds
3. Repository B receives event: ~instant
4. Repository B workflow runs: ~1-3 minutes (depends on build)
5. Deployment: ~30-60 seconds

Total: Usually under 5 minutes from push to live.

### Q: Can I see what changed before rebuilding?

**A:** Yes! Repository A sends the commit SHA, so you can:

```yaml
- name: Show changes
  run: |
    git clone https://github.com/${{ github.event.client_payload.repository }}.git source
    cd source
    git diff ${{ github.event.client_payload.sha }}~1 ${{ github.event.client_payload.sha }}
```

### Q: What if the workflow fails?

**A:** 
1. Check the workflow logs in the Actions tab
2. Common issues:
   - Missing secrets (Repository A)
   - Incorrect event type name
   - Authentication failures
   - Build errors (Repository B)
3. Fix the issue and either:
   - Re-run the workflow manually, or
   - Make a small change and push again

### Q: Can I deploy to multiple environments (staging, production)?

**A:** Yes! Here's how:

**Option 1:** Different target repositories
```
TARGET_REPOSITORIES=owner/site-staging,owner/site-production
```

**Option 2:** Same repo, different workflows
- Use different event types: `content-update-staging`, `content-update-production`
- Trigger based on branch: `main` → production, `develop` → staging

**Option 3:** Approval gates
```yaml
deploy-production:
  needs: deploy-staging
  environment: production  # Requires approval
```

---

## Security Questions

### Q: Is it safe to store content in a public repository?

**A:** If your content is meant to be public (like blog posts), yes. However:
- ✅ Don't commit secrets, API keys, or passwords
- ✅ Don't include sensitive personal information
- ✅ Be aware anyone can see the content
- ✅ Consider licenses for your content

### Q: Can someone maliciously trigger rebuilds of my site?

**A:** No, because:
- The `DISPATCH_TOKEN` is required to send dispatch events
- The token is stored securely as a repository secret
- Only repository collaborators can push to Repository A
- GitHub's authentication prevents unauthorized API calls

### Q: What permissions does the Personal Access Token need?

**A:** Only `repo` scope. This allows:
- ✅ Sending repository dispatch events
- ✅ Triggering workflows in target repositories

Do NOT grant additional unnecessary permissions.

### Q: Can I rotate the Personal Access Token?

**A:** Yes, and you should periodically:
1. Generate a new token
2. Update the `DISPATCH_TOKEN` secret in Repository A
3. Delete the old token
4. No changes needed in Repository B

---

## Customization Questions

### Q: Can I use a different directory than `_posts/`?

**A:** Yes! Edit the workflow in Repository A:

```yaml
on:
  push:
    branches:
      - main
    paths:
      - 'content/blog/**.md'  # Your custom path
```

And update the copy command in Repository B accordingly.

### Q: Can I transform content before building?

**A:** Yes! Add transformation steps in Repository B:

```yaml
- name: Transform content
  run: |
    # Convert frontmatter format
    python scripts/convert-frontmatter.py
    
    # Optimize images
    python scripts/optimize-images.py
    
    # Generate metadata
    python scripts/generate-metadata.py
```

### Q: Can I aggregate content from multiple repositories?

**A:** Yes! See the [advanced-multi-source.yml](examples/target-repository-workflows/advanced-multi-source.yml) example. Each source repository can send dispatch events, and your workflow can handle them differently based on the source.

---

## Debugging Questions

### Q: How do I know if the dispatch event was sent?

**A:** Check Repository A's workflow logs:
- Look for "Successfully triggered rebuild for {repo}" messages
- Check for HTTP status codes (201 = success)
- Review the API response in the logs

### Q: Repository B workflow didn't trigger. Why?

**A:** Common causes:
1. **Event type mismatch**: Ensure both use `content-update`
2. **Workflow file location**: Must be in `.github/workflows/`
3. **Syntax errors**: Validate YAML syntax
4. **Permissions**: Check workflow permissions in repo settings
5. **Default branch**: Workflow file must be on default branch

Debug steps:
```bash
# Check workflow file exists
ls .github/workflows/content-update.yml

# Validate YAML syntax
cat .github/workflows/content-update.yml | yaml-validator

# Check event type
grep "types:" .github/workflows/content-update.yml
```

### Q: How do I test without pushing to main?

**A:** Use manual workflow triggers:

1. Add `workflow_dispatch` to Repository A's workflow:
```yaml
on:
  push:
    # ...
  workflow_dispatch:  # Allows manual trigger
```

2. Trigger manually from Actions tab
3. This sends the dispatch without requiring a push

---

## Cost Questions

### Q: Does this cost money?

**A:** For public repositories:
- ✅ GitHub Actions: Free (2,000 minutes/month)
- ✅ GitHub API: Free (5,000 requests/hour)
- ✅ Repository hosting: Free

For private repositories:
- GitHub Actions: 2,000 minutes/month free, then $0.008/minute
- API limits are the same
- Repository hosting: Free for Pro accounts

Typical usage: ~1-2 minutes per content update, well within free tier.

### Q: What are the rate limits?

**A:** GitHub API rate limits:
- **Authenticated requests**: 5,000 per hour
- **Repository dispatch**: No specific limit beyond the general API limit

This workflow makes 1 API call per target repository per push. Unless you're pushing hundreds of times per hour, you won't hit the limit.

---

## Migration Questions

### Q: I already have a website with content. How do I migrate?

**A:**
1. Copy your existing content to this repository's `_posts/` folder
2. Ensure each file has proper YAML frontmatter
3. Set up the workflow in your existing website repository
4. Test with a new post to verify the integration
5. Optionally remove content from your website repo (keep in this repo only)

### Q: Can I migrate from WordPress/Medium/etc.?

**A:** Yes! You'll need to:
1. Export your content from the current platform
2. Convert to Markdown format (many tools available)
3. Add YAML frontmatter to each file
4. Import to this repository's `_posts/` folder

Tools that might help:
- `wordpress-export-to-markdown`
- `medium-to-markdown`
- `jekyll-import`

---

## Advanced Questions

### Q: Can I use this pattern for other types of content (not just blog posts)?

**A:** Absolutely! This pattern works for:
- 📄 Documentation updates
- 📊 Data files (JSON, YAML, CSV)
- 🎨 Design assets (if small)
- 📝 Any text-based content

Just adjust the `paths` filter and content directory accordingly.

### Q: Can Repository B push changes back to Repository A?

**A:** Yes, but you'd need to:
1. Give Repository B write access to Repository A (via PAT)
2. Add git commit/push steps in Repository B's workflow
3. Be careful to avoid infinite loops!

Example use case: Generating metadata or processing content that should be saved back.

### Q: Can I run custom scripts when content updates?

**A:** Yes! Add script execution steps in Repository B:

```yaml
- name: Run custom processing
  run: |
    python scripts/process-content.py
    node scripts/generate-sitemap.js
    bash scripts/notify-subscribers.sh
```

---

## Need More Help?

- 📖 [Integration Guide](INTEGRATION_GUIDE.md) - Comprehensive setup guide
- 📋 [Example Workflows](examples/target-repository-workflows/) - Ready-to-use templates
- 🗺️ [Connection Diagram](CONNECTION_DIAGRAM.md) - Visual explanation
- 💬 Open an issue in this repository

---

## Contributing

Found an issue or have a question not covered here? Please:
1. Check existing issues first
2. Open a new issue with details
3. We'll update this FAQ based on common questions
