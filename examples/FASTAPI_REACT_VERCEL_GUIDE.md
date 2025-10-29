# Integration Guide: FastAPI + React + Vercel

This guide shows you how to integrate Papers2Code-updates with a Vercel-deployed application that has a FastAPI backend and React frontend.

## Architecture Overview

```
┌────────────────────────────────┐
│  Papers2Code-updates           │
│  (Content Repository)          │
│                                │
│  📝 _posts/*.md files          │
└──────────────┬─────────────────┘
               │
               │ repository_dispatch event
               │
               ▼
┌────────────────────────────────┐
│  Your Repository (Repo B)      │
│  FastAPI + React + Vercel      │
│                                │
│  ├── backend/                  │
│  │   ├── main.py (FastAPI)    │
│  │   └── content/             │
│  │       ├── posts/*.md        │ ← Content synced here
│  │       └── posts.json        │ ← Generated from markdown
│  │                             │
│  ├── frontend/                 │
│  │   ├── src/ (React)          │
│  │   └── public/               │
│  │       └── content/          │ ← Or sync here
│  │                             │
│  └── .github/workflows/        │
│      └── content-update.yml    │ ← Workflow to handle updates
└──────────────┬─────────────────┘
               │
               │ Deploy
               ▼
┌────────────────────────────────┐
│  Vercel                        │
│  (Hosting Platform)            │
└────────────────────────────────┘
```

## Setup Steps

### Step 1: Choose Your Content Strategy

You have three main options for where to store markdown content:

#### Option A: Backend Content (Recommended for dynamic content)
Store markdown in `backend/content/posts/` and serve via FastAPI:

```python
# backend/main.py
from fastapi import FastAPI
import json
from pathlib import Path

app = FastAPI()

@app.get("/api/posts")
def get_posts():
    posts_file = Path("content/posts.json")
    if posts_file.exists():
        with open(posts_file) as f:
            return json.load(f)
    return []

@app.get("/api/posts/{slug}")
def get_post(slug: str):
    posts = get_posts()
    for post in posts:
        if post['slug'] == slug:
            return post
    return {"error": "Post not found"}
```

#### Option B: Frontend Public Content (For static builds)
Store in `frontend/public/content/` for React to fetch:

```javascript
// frontend/src/hooks/usePosts.js
import { useEffect, useState } from 'react';

export function usePosts() {
  const [posts, setPosts] = useState([]);
  
  useEffect(() => {
    fetch('/content/posts.json')
      .then(res => res.json())
      .then(data => setPosts(data));
  }, []);
  
  return posts;
}
```

#### Option C: Root-level Content Directory
Store in `content/posts/` and access from both backend and frontend.

### Step 2: Add Workflow to Your Repository

Copy the workflow file to your repository:

```bash
# In your FastAPI + React repository (Repository B)
mkdir -p .github/workflows
cp path/to/fastapi-react-vercel.yml .github/workflows/content-update.yml
```

### Step 3: Configure Repository Secrets

Add these secrets to your repository (Settings → Secrets and variables → Actions):

#### For Vercel CLI Deployment (Option A):

```
VERCEL_TOKEN       - Your Vercel token
VERCEL_ORG_ID      - Your Vercel organization ID
VERCEL_PROJECT_ID  - Your Vercel project ID
```

**How to get these values:**

1. **VERCEL_TOKEN**: 
   - Go to https://vercel.com/account/tokens
   - Create a new token
   - Copy and save as repository secret

2. **VERCEL_ORG_ID** and **VERCEL_PROJECT_ID**:
   ```bash
   # In your project directory
   npx vercel link
   # This creates .vercel/project.json
   cat .vercel/project.json
   ```

#### For Deploy Hook (Option B - Simpler):

```
VERCEL_DEPLOY_HOOK_URL - Your Vercel deploy hook URL
```

**How to create a deploy hook:**
1. Go to your Vercel project → Settings → Git → Deploy Hooks
2. Create a new hook (e.g., "Content Update Hook")
3. Copy the URL and save as repository secret

### Step 4: Customize the Workflow

Edit `.github/workflows/content-update.yml` based on your project structure:

#### A. Update Content Paths

Find this section and adjust paths:

```yaml
# Sync to backend directory (if FastAPI serves markdown)
mkdir -p ../backend/content/posts
cp -r _posts/* ../backend/content/posts/
```

Change `backend/content/posts` to match your structure, e.g.:
- `api/content/posts`
- `server/content/posts`
- `content/posts`

#### B. Choose Deployment Method

**Method 1: Vercel CLI** (More control, can set environment variables)

```yaml
- name: Deploy to Vercel
  if: true  # Enable this
  env:
    VERCEL_TOKEN: ${{ secrets.VERCEL_TOKEN }}
    VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
    VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
  run: |
    npm install -g vercel
    vercel --prod --token=$VERCEL_TOKEN --yes
```

**Method 2: Deploy Hook** (Simpler, less configuration)

```yaml
- name: Trigger Vercel Deploy Hook
  if: true  # Enable this
  env:
    VERCEL_DEPLOY_HOOK_URL: ${{ secrets.VERCEL_DEPLOY_HOOK_URL }}
  run: |
    curl -X POST "$VERCEL_DEPLOY_HOOK_URL"
```

#### C. Enable/Disable Content Processing

If you want to generate JSON from markdown (recommended for FastAPI):

```yaml
- name: Process markdown to JSON
  run: |
    # This step converts markdown to posts.json
    python process_content.py
```

Keep this enabled if your FastAPI serves posts via JSON API.

### Step 5: Project Structure Examples

#### Example 1: Monorepo Structure

```
your-repo/
├── backend/
│   ├── main.py
│   ├── requirements.txt
│   └── content/
│       ├── posts/              ← Markdown files here
│       │   ├── 2025-10-28-post.md
│       │   └── 2025-10-29-post.md
│       └── posts.json          ← Generated API data
│
├── frontend/
│   ├── package.json
│   ├── src/
│   │   ├── App.jsx
│   │   └── components/
│   │       └── PostList.jsx
│   └── public/
│
├── .github/
│   └── workflows/
│       └── content-update.yml
│
└── vercel.json
```

#### Example 2: Separate Backend/Frontend

```
your-repo/
├── api/                        ← FastAPI
│   ├── main.py
│   └── content/
│       └── posts/              ← Content here
│
├── src/                        ← React
│   ├── App.jsx
│   └── hooks/
│       └── usePosts.js
│
├── .github/workflows/
└── vercel.json
```

### Step 6: Configure Vercel

Update your `vercel.json` to handle both backend and frontend:

#### For Monorepo:

```json
{
  "version": 2,
  "builds": [
    {
      "src": "backend/main.py",
      "use": "@vercel/python"
    },
    {
      "src": "frontend/package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "frontend/build"
      }
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "backend/main.py"
    },
    {
      "src": "/(.*)",
      "dest": "frontend/$1"
    }
  ]
}
```

#### For Python API + React Frontend:

```json
{
  "version": 2,
  "builds": [
    {
      "src": "api/main.py",
      "use": "@vercel/python"
    },
    {
      "src": "package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "build"
      }
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "api/main.py"
    },
    {
      "src": "/(.*)",
      "dest": "/$1"
    }
  ]
}
```

### Step 7: FastAPI Implementation

Here's a complete example of serving markdown content via FastAPI:

```python
# backend/main.py
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
import json
from pathlib import Path
from typing import List, Dict, Any

app = FastAPI()

# Enable CORS for React frontend
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Adjust for production
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Path to processed content
POSTS_FILE = Path(__file__).parent / "content" / "posts.json"

@app.get("/")
def read_root():
    return {"message": "FastAPI + React Blog API"}

@app.get("/api/posts", response_model=List[Dict[str, Any]])
def get_all_posts():
    """Get all blog posts"""
    if not POSTS_FILE.exists():
        return []
    
    with open(POSTS_FILE, 'r', encoding='utf-8') as f:
        posts = json.load(f)
    
    # Return metadata only (without full content)
    return [
        {
            "slug": post["slug"],
            "title": post["title"],
            "date": post["date"],
            "author": post["author"],
            "categories": post["categories"],
            "tags": post["tags"],
            "description": post["description"]
        }
        for post in posts
    ]

@app.get("/api/posts/{slug}", response_model=Dict[str, Any])
def get_post(slug: str):
    """Get a specific blog post by slug"""
    if not POSTS_FILE.exists():
        raise HTTPException(status_code=404, detail="No posts found")
    
    with open(POSTS_FILE, 'r', encoding='utf-8') as f:
        posts = json.load(f)
    
    for post in posts:
        if post["slug"] == slug:
            return post
    
    raise HTTPException(status_code=404, detail=f"Post '{slug}' not found")

@app.get("/api/categories")
def get_categories():
    """Get all unique categories"""
    if not POSTS_FILE.exists():
        return []
    
    with open(POSTS_FILE, 'r', encoding='utf-8') as f:
        posts = json.load(f)
    
    categories = set()
    for post in posts:
        categories.update(post.get("categories", []))
    
    return sorted(list(categories))

@app.get("/api/tags")
def get_tags():
    """Get all unique tags"""
    if not POSTS_FILE.exists():
        return []
    
    with open(POSTS_FILE, 'r', encoding='utf-8') as f:
        posts = json.load(f)
    
    tags = set()
    for post in posts:
        tags.update(post.get("tags", []))
    
    return sorted(list(tags))
```

### Step 8: React Implementation

Example React component to display posts:

```javascript
// frontend/src/components/PostList.jsx
import React, { useEffect, useState } from 'react';

function PostList() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Fetch from FastAPI backend
    fetch('/api/posts')
      .then(res => res.json())
      .then(data => {
        setPosts(data);
        setLoading(false);
      })
      .catch(err => {
        console.error('Error fetching posts:', err);
        setLoading(false);
      });
  }, []);

  if (loading) return <div>Loading posts...</div>;

  return (
    <div className="post-list">
      <h1>Blog Posts</h1>
      {posts.map(post => (
        <article key={post.slug} className="post-preview">
          <h2>{post.title}</h2>
          <p className="post-meta">
            By {post.author} on {post.date}
          </p>
          <p>{post.description}</p>
          <a href={`/posts/${post.slug}`}>Read more →</a>
        </article>
      ))}
    </div>
  );
}

export default PostList;
```

## Testing the Integration

### Test 1: Manual Workflow Trigger

1. Go to your repository's Actions tab
2. Select "Update Content and Deploy to Vercel"
3. Click "Run workflow"
4. Check the logs to ensure:
   - Content is fetched
   - Files are synced
   - JSON is generated (if enabled)
   - Vercel deployment succeeds

### Test 2: End-to-End Test

1. In Papers2Code-updates, create a test post:

```bash
# In Papers2Code-updates repository
cat > _posts/2025-10-29-test-integration.md << 'EOF'
---
title: "Test Integration Post"
date: 2025-10-29
author: "Test Author"
categories: [testing]
tags: [integration, test]
description: "Testing the FastAPI + React integration"
---

# Test Post

This is a test post to verify the integration works correctly.
EOF

git add _posts/2025-10-29-test-integration.md
git commit -m "Add test post"
git push
```

2. Monitor both repositories:
   - Papers2Code-updates: Check "Trigger Website Build" workflow
   - Your repository: Check "Update Content and Deploy to Vercel" workflow

3. Verify deployment:
   - Visit your Vercel URL
   - Check if the new post appears
   - Test the API endpoint: `https://your-app.vercel.app/api/posts`

## Troubleshooting

### Issue: Workflow not triggering

**Check:**
- Is your repository listed in Papers2Code-updates `TARGET_REPOSITORIES` secret?
- Does the workflow file have the correct event type (`content-update`)?

```bash
# In your repository
cat .github/workflows/content-update.yml | grep -A 2 "repository_dispatch"
# Should show:
#   repository_dispatch:
#     types: [content-update]
```

### Issue: Content not appearing in API

**Check:**
1. Verify files were copied:
   - Check workflow logs for "Content synced successfully"
   - Verify paths match your project structure

2. Verify JSON generation:
   - Check if `posts.json` was created
   - Look for "Content processing complete" in logs

3. Verify FastAPI is reading the file:
   ```python
   # Add debug logging to main.py
   import logging
   logging.info(f"Posts file exists: {POSTS_FILE.exists()}")
   ```

### Issue: Vercel deployment fails

**For CLI method:**
- Verify all three secrets are set correctly
- Check that `vercel.json` is properly configured
- Review Vercel deployment logs

**For Deploy Hook method:**
- Verify the deploy hook URL is correct
- Check Vercel dashboard for deployment status

### Issue: CORS errors in React

Add CORS middleware to FastAPI:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "https://your-app.vercel.app"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## Advanced: Processing Markdown in FastAPI

If you want FastAPI to process markdown on-the-fly instead of pre-generating JSON:

```python
# backend/main.py
from fastapi import FastAPI
import frontmatter
from pathlib import Path
import markdown

app = FastAPI()

POSTS_DIR = Path(__file__).parent / "content" / "posts"

@app.get("/api/posts")
def get_posts():
    posts = []
    for md_file in POSTS_DIR.glob("*.md"):
        with open(md_file, 'r', encoding='utf-8') as f:
            post = frontmatter.load(f)
            posts.append({
                "slug": md_file.stem,
                "title": post.get("title"),
                "date": str(post.get("date")),
                "author": post.get("author"),
                "description": post.get("description")
            })
    return sorted(posts, key=lambda x: x['date'], reverse=True)

@app.get("/api/posts/{slug}")
def get_post(slug: str):
    md_file = POSTS_DIR / f"{slug}.md"
    if not md_file.exists():
        raise HTTPException(status_code=404)
    
    with open(md_file, 'r', encoding='utf-8') as f:
        post = frontmatter.load(f)
        return {
            "slug": slug,
            "title": post.get("title"),
            "date": str(post.get("date")),
            "author": post.get("author"),
            "categories": post.get("categories", []),
            "tags": post.get("tags", []),
            "description": post.get("description"),
            "content": markdown.markdown(post.content)  # Convert to HTML
        }
```

Don't forget to add dependencies:

```txt
# backend/requirements.txt
fastapi
uvicorn
python-frontmatter
markdown
```

## Summary Checklist

- [ ] Copy `fastapi-react-vercel.yml` to `.github/workflows/content-update.yml`
- [ ] Update content paths in workflow to match your structure
- [ ] Choose and configure deployment method (CLI or Deploy Hook)
- [ ] Add Vercel secrets to repository settings
- [ ] Update `vercel.json` configuration
- [ ] Implement FastAPI endpoints to serve posts
- [ ] Implement React components to display posts
- [ ] Test manual workflow trigger
- [ ] Test end-to-end with a new post in Papers2Code-updates
- [ ] Verify deployment on Vercel
- [ ] Monitor both workflows for errors

## Additional Resources

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [React Documentation](https://react.dev/)
- [Vercel Python Documentation](https://vercel.com/docs/functions/runtimes/python)
- [Vercel Deploy Hooks](https://vercel.com/docs/deployments/deploy-hooks)

---

Need help? Open an issue in the Papers2Code-updates repository with:
- Your project structure
- Workflow logs
- Vercel deployment logs
