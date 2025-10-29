# Connection Diagram: How Repositories Communicate

This document provides a visual explanation of how Papers2Code-updates (Repository A) connects to target websites (Repository B).

## Overview: The Repository Dispatch Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│                    GitHub Ecosystem                              │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  Repository A: Papers2Code-updates                     │    │
│  │  (Content Source)                                      │    │
│  │                                                         │    │
│  │  📝 _posts/                                            │    │
│  │     └── 2025-10-28-new-paper.md  ← New file added     │    │
│  │                                                         │    │
│  │  🔄 .github/workflows/trigger-website-build.yml       │    │
│  │     └── Detects change, sends notifications            │    │
│  └─────────────────────┬──────────────────────────────────┘    │
│                        │                                         │
│                        │ GitHub API Call                         │
│                        │ POST /repos/{owner}/{repo}/dispatches   │
│                        │                                         │
│                        ▼                                         │
│  ┌────────────────────────────────────────────────────────┐    │
│  │  Repository B: Your Website                            │    │
│  │  (Content Consumer)                                    │    │
│  │                                                         │    │
│  │  🔔 Receives: repository_dispatch event               │    │
│  │     Payload: {                                         │    │
│  │       repository: "RyanKim17920/Papers2Code-updates"   │    │
│  │       sha: "abc123..."                                 │    │
│  │     }                                                   │    │
│  │                                                         │    │
│  │  🔄 .github/workflows/content-update.yml              │    │
│  │     1. Fetches content from Repo A                     │    │
│  │     2. Copies to local content directory               │    │
│  │     3. Rebuilds website                                │    │
│  │     4. Deploys                                         │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Detailed Flow: Step by Step

### Phase 1: Content Update (Repository A)

```
User Action                  Repository A                     GitHub
─────────────────────────────────────────────────────────────────────

1. User writes post
   in _posts/
                        
2. git commit & push ──────▶ Receives push event
                             on main branch
                        
                        3. Workflow triggers
                           (trigger-website-build.yml)
                        
                        4. Identifies changed 
                           .md files in _posts/
                        
                        5. Reads TARGET_REPOSITORIES
                           secret: "owner/repo1,owner/repo2"
                        
                        6. For each target repo,
                           makes API call ─────────▶ GitHub API
                                                     validates token
                           
                           curl -X POST \             & delivers
                             /repos/owner/repo1/ ◀─── webhook
                             dispatches
```

### Phase 2: Content Consumption (Repository B)

```
GitHub                      Repository B                     Deployment
─────────────────────────────────────────────────────────────────────

GitHub API delivers ───────▶ 1. Receives 
repository_dispatch            repository_dispatch
event to Repository B          event
                           
                           2. Workflow triggers
                              (content-update.yml)
                           
                           3. Reads payload:
                              - source repo name
                              - commit SHA
                           
                           4. Clones Repo A
                              git clone [repo]
                              git checkout [sha]
                           
                           5. Copies markdown files
                              cp _posts/* content/
                           
                           6. Runs build
                              npm run build
                              (or jekyll, hugo, etc.)
                           
                           7. Deploys website ───────────────▶ GitHub Pages
                                                              Vercel
                                                              Netlify
                                                              etc.
```

---

## Data Flow: What Gets Transmitted?

### 1. Trigger Event (Push to Repo A)

```
Event Type: push
Trigger: Changes to _posts/*.md files
Branch: main or master
```

### 2. API Call (Repo A → GitHub → Repo B)

```http
POST https://api.github.com/repos/{target-owner}/{target-repo}/dispatches

Headers:
  Authorization: token ${DISPATCH_TOKEN}
  Accept: application/vnd.github.v3+json

Body:
{
  "event_type": "content-update",
  "client_payload": {
    "repository": "RyanKim17920/Papers2Code-updates",
    "ref": "refs/heads/main",
    "sha": "abc123def456789..."
  }
}
```

### 3. Received Event (In Repo B)

```yaml
# Accessible in workflow as:
github.event.client_payload.repository  # "RyanKim17920/Papers2Code-updates"
github.event.client_payload.ref         # "refs/heads/main"
github.event.client_payload.sha         # "abc123def456789..." ← Most important!
```

---

## Key Components Explained

### Component 1: Personal Access Token (PAT)

```
Purpose: Authenticates API calls from Repo A to Repo B
Required Scope: repo (full control of private repositories)
Storage: Repository secret DISPATCH_TOKEN in Repo A
Security: Never exposed in logs or code
```

**How it works:**
- Repo A cannot directly trigger workflows in Repo B
- GitHub API requires authentication to send dispatch events
- PAT acts as your identity to authorize the API call
- Must have sufficient permissions to trigger workflows in target repos

### Component 2: Repository Dispatch Event

```
What: A webhook-like notification system between repos
Type: "content-update" (custom, defined by us)
Payload: Flexible JSON data structure
Trigger: Workflow in target repo listening for this event type
```

**Why repository_dispatch?**
- ✅ Allows cross-repository communication
- ✅ Doesn't require direct repo access
- ✅ Asynchronous and reliable
- ✅ Supports custom payloads
- ✅ Standard GitHub Actions feature

### Component 3: Commit SHA

```
Purpose: Identifies exact version of content to fetch
Format: 40-character hexadecimal string (e.g., "abc123...")
Usage: git checkout ${SHA}
Why not branch?: Branch can change; SHA is immutable
```

**Example:**
```bash
# ❌ Bad: Branch might have new commits
git clone repo && git checkout main

# ✅ Good: SHA is immutable, exact version
git clone repo && git checkout abc123def456789
```

---

## Security & Permissions

### Required Permissions in Repository A

```
Repository Secrets:
├── DISPATCH_TOKEN      (PAT with repo scope)
└── TARGET_REPOSITORIES (comma-separated list)

Workflow Permissions:
└── contents: read      (to read repository files)
```

### Required Permissions in Repository B

```
Workflow Permissions:
├── contents: read      (to read repository files)
├── contents: write     (if committing files back)
├── pages: write        (if deploying to GitHub Pages)
└── id-token: write     (for GitHub Pages OIDC auth)
```

### Security Best Practices

```
✅ DO:
- Store PAT as repository secret
- Use minimal required scope (repo)
- Rotate tokens periodically
- Limit TARGET_REPOSITORIES to trusted repos
- Validate content before building

❌ DON'T:
- Commit tokens to code
- Use tokens with excessive permissions
- Share tokens between projects
- Trust content without validation
```

---

## Troubleshooting: Common Issues

### Issue 1: Dispatch Event Not Received

```
Symptom: Repo B workflow doesn't trigger

Check:
┌─ Is DISPATCH_TOKEN configured in Repo A?
├─ Does DISPATCH_TOKEN have 'repo' scope?
├─ Is Repo B in TARGET_REPOSITORIES?
├─ Is event_type "content-update" in both repos?
└─ Check Repo A workflow logs for API errors

Debug:
- Look for HTTP status codes in Repo A logs
  - 201: Success
  - 401: Unauthorized (bad token)
  - 404: Repository not found
  - 422: Invalid payload
```

### Issue 2: Content Not Syncing

```
Symptom: Workflow runs but content doesn't appear

Check:
┌─ Is git clone command working?
├─ Is commit SHA valid?
├─ Are source/destination paths correct?
├─ Does _posts/ directory exist in Repo A?
└─ Are file permissions correct?

Debug:
- Add 'ls -la' commands to see directory contents
- Print environment variables
- Check if files are copied to correct location
```

### Issue 3: Build Failures

```
Symptom: Build step fails after content sync

Check:
┌─ Are dependencies installed?
├─ Is content format valid?
├─ Are frontmatter fields correct?
├─ Are image paths valid?
└─ Is build command correct?

Debug:
- Test build locally with synced content
- Check build logs for specific errors
- Validate markdown and frontmatter
```

---

## Alternative Architectures

### Option 1: Direct Git Submodule
```
Pros: Simpler, native Git feature
Cons: Manual updates required, tightly coupled

Your-Repo/
├── content/
│   └── papers2code (git submodule)
└── ...
```

### Option 2: Scheduled Sync
```
Pros: Regular updates, doesn't require dispatch
Cons: Not real-time, may miss updates

# Runs every hour
on:
  schedule:
    - cron: '0 * * * *'
```

### Option 3: GitHub App Webhooks
```
Pros: More powerful, bi-directional
Cons: Complex setup, requires server

Custom webhook server listens for events
and performs custom logic
```

**Why we chose Repository Dispatch:**
- ✅ Real-time updates
- ✅ No manual intervention
- ✅ Simple setup
- ✅ Native GitHub feature
- ✅ Decoupled repositories

---

## Quick Reference: File Locations

### In Repository A (Papers2Code-updates):

```
.github/workflows/trigger-website-build.yml  ← Sends notifications
Secrets:
  - DISPATCH_TOKEN
  - TARGET_REPOSITORIES
```

### In Repository B (Your Website):

```
.github/workflows/content-update.yml         ← Receives notifications
content/posts/                               ← Content destination
                                              (adjust to your structure)
```

---

## Summary

**The connection works through:**

1. 🔔 **GitHub Actions Workflows** - Automation in both repos
2. 🔐 **Personal Access Token** - Authentication for API calls
3. 📡 **Repository Dispatch API** - Communication mechanism
4. 📦 **Commit SHA** - Exact content versioning
5. 📝 **Markdown Files** - The actual content being shared

**Key Insight:** This is a **loose coupling** pattern where repositories communicate through GitHub's API rather than direct dependencies. Repo A doesn't need to know how Repo B works; it just sends a notification. Repo B independently decides how to handle that notification.

---

For implementation details, see:
- [Integration Guide](INTEGRATION_GUIDE.md)
- [Quick Setup Guide](examples/REPOSITORY_B_SETUP.md)
- [Example Workflows](examples/target-repository-workflows/)
