# Papers2Code-updates

A content repository that automatically notifies other repositories to rebuild when new markdown files are pushed.

## Repository Structure

```
Papers2Code-updates/
├── _posts/                          # Directory for blog content
│   └── sample-update.md            # Sample post with YAML frontmatter
├── .github/
│   └── workflows/
│       └── trigger-website-build.yml  # Automation workflow
└── README.md
```

## Features

- **Automatic Notifications**: When markdown files in `_posts/` are pushed, the workflow automatically triggers rebuilds in configured target repositories
- **YAML Frontmatter Support**: Blog posts include structured metadata (title, date, author, categories, tags, description)
- **Manual Triggering**: Workflow can be manually triggered via GitHub Actions UI
- **Multi-Repository Support**: Can notify multiple target repositories simultaneously

## Setup Instructions

### 1. Configure Repository Secrets

Add the following secrets to this repository (Settings → Secrets and variables → Actions):

#### `DISPATCH_TOKEN`
A GitHub Personal Access Token with `repo` scope:
1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Generate new token with `repo` scope
3. Copy the token and add it as a repository secret

#### `TARGET_REPOSITORIES`
Comma-separated list of repositories to notify (format: `owner/repo`):
```
owner1/repo1,owner2/repo2,owner3/repo3
```

### 2. Configure Target Repositories

Each target repository should have a workflow that listens for the `repository_dispatch` event:

```yaml
name: Rebuild on Content Update

on:
  repository_dispatch:
    types: [content-update]

jobs:
  rebuild:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Rebuild site
        run: |
          echo "Content updated in ${{ github.event.client_payload.repository }}"
          # Add your build commands here
```

## Usage

### Adding New Content

1. Create a new markdown file in the `_posts/` directory:

```markdown
---
title: "Your Post Title"
date: 2025-10-28
author: "Author Name"
categories: [category1, category2]
tags: [tag1, tag2]
description: "Brief description of your post"
---

# Your Content Here

Write your blog post content using Markdown...
```

2. Commit and push to the `main` or `master` branch
3. The workflow will automatically trigger and notify all configured target repositories

### Manual Triggering

You can manually trigger the workflow:
1. Go to Actions tab in this repository
2. Select "Trigger Website Build" workflow
3. Click "Run workflow"

## Workflow Details

The `trigger-website-build.yml` workflow:

- **Triggers on**: 
  - Push to `main` or `master` branches
  - Changes to markdown files in `_posts/` directory
  - Manual trigger via `workflow_dispatch`

- **Actions**:
  1. Checks out the repository
  2. Identifies changed markdown files
  3. Sends repository dispatch events to all configured target repositories
  4. Provides a summary of notified repositories and changed files

- **Event Payload**: Target repositories receive:
  ```json
  {
    "event_type": "content-update",
    "client_payload": {
      "repository": "source-repo-name",
      "ref": "refs/heads/main",
      "sha": "commit-sha"
    }
  }
  ```

## Example Post Structure

See [`_posts/sample-update.md`](_posts/sample-update.md) for a complete example with proper YAML frontmatter structure.

## Troubleshooting

### Workflow not triggering
- Ensure changes are pushed to `main` or `master` branch
- Verify that changed files are in `_posts/` directory with `.md` or `.markdown` extension

### Target repositories not rebuilding
- Verify `DISPATCH_TOKEN` has correct permissions (`repo` scope)
- Check `TARGET_REPOSITORIES` format is correct (`owner/repo,owner/repo`)
- Ensure target repositories have workflows listening for `repository_dispatch` events with type `content-update`
- Check target repository workflow logs for errors

### Checking workflow execution
- Go to Actions tab in this repository
- Click on "Trigger Website Build" to see workflow runs
- Review logs for detailed information about notifications sent

## License

This repository is part of the Papers2Code project.