# Integration Guide: Connecting Papers2Code-updates to Repository B

This guide explains how this content repository (Papers2Code-updates) connects to other repositories (referred to as "Repository B" or "target repositories") and how to structure Repository B to receive content updates.

## Table of Contents
1. [How the Connection Works](#how-the-connection-works)
2. [Setting Up Repository A (This Repository)](#setting-up-repository-a-this-repository)
3. [Setting Up Repository B (Target Repository)](#setting-up-repository-b-target-repository)
4. [Data Flow and Payload Structure](#data-flow-and-payload-structure)
5. [Complete Examples](#complete-examples)
6. [Advanced Use Cases](#advanced-use-cases)

---

## How the Connection Works

The connection between this repository (Repository A) and target repositories (Repository B) uses GitHub's **Repository Dispatch Events** mechanism.

### High-Level Flow:

```
┌─────────────────────────────────────┐
│  Repository A (Papers2Code-updates) │
│                                     │
│  1. Markdown file pushed to _posts/ │
│  2. Workflow triggered              │
│  3. GitHub API call made            │
└──────────────┬──────────────────────┘
               │
               │ repository_dispatch event
               ▼
┌─────────────────────────────────────┐
│  Repository B (Target Website)      │
│                                     │
│  1. Receives dispatch event         │
│  2. Workflow triggered              │
│  3. Fetches content from Repo A     │
│  4. Rebuilds website                │
└─────────────────────────────────────┘
```

### Key Components:

1. **GitHub Actions Workflow** (in Repo A): Monitors changes to markdown files
2. **GitHub API**: Sends webhook-like notifications to other repositories
3. **Personal Access Token**: Authenticates API calls to trigger events in Repo B
4. **Repository Dispatch Listener** (in Repo B): Receives notifications and triggers rebuild

---

## Setting Up Repository A (This Repository)

### Prerequisites:
- GitHub Personal Access Token with `repo` scope

### Step 1: Create a Personal Access Token

1. Go to GitHub Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Give it a descriptive name: "Papers2Code Content Dispatch"
4. Select the `repo` scope (full control of private repositories)
5. Set an appropriate expiration date
6. Generate and copy the token

### Step 2: Configure Repository Secrets

Add these secrets to Repository A (Settings → Secrets and variables → Actions → New repository secret):

#### `DISPATCH_TOKEN`
- **Value**: Your Personal Access Token from Step 1
- **Purpose**: Authenticates API calls to trigger events in target repositories

#### `TARGET_REPOSITORIES`
- **Value**: Comma-separated list of target repositories
- **Format**: `owner1/repo1,owner2/repo2,owner3/repo3`
- **Example**: `RyanKim17920/my-website,JohnDoe/blog-site`

### Step 3: Verify Workflow Configuration

The workflow at `.github/workflows/trigger-website-build.yml` is already configured to:
- Trigger on pushes to `_posts/` directory
- Parse the target repositories from secrets
- Send dispatch events to each target repository

---

## Setting Up Repository B (Target Repository)

Repository B must be configured to receive and respond to dispatch events from Repository A.

### Required Structure:

```
Repository-B/
├── .github/
│   └── workflows/
│       └── content-update.yml    # Workflow that listens for dispatch events
├── content/                      # Directory where MD files will be stored
│   └── posts/                    # (or any structure you prefer)
├── [your build system files]     # e.g., package.json, requirements.txt, etc.
└── [your site generation files]  # e.g., Jekyll, Hugo, Next.js files
```

### Step 1: Create the Workflow File

Create `.github/workflows/content-update.yml` in Repository B:

```yaml
name: Rebuild on Content Update

on:
  repository_dispatch:
    types: [content-update]  # Must match the event_type sent from Repo A

jobs:
  rebuild-site:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout Repository B
        uses: actions/checkout@v4
      
      - name: Display notification details
        run: |
          echo "Content update notification received!"
          echo "Source repository: ${{ github.event.client_payload.repository }}"
          echo "Source ref: ${{ github.event.client_payload.ref }}"
          echo "Source commit: ${{ github.event.client_payload.sha }}"
      
      - name: Fetch content from Repository A
        env:
          CONTENT_REPO: ${{ github.event.client_payload.repository }}
          CONTENT_SHA: ${{ github.event.client_payload.sha }}
        run: |
          # Clone the content repository
          git clone https://github.com/$CONTENT_REPO.git content-repo
          cd content-repo
          git checkout $CONTENT_SHA
          
          # Copy markdown files to your content directory
          mkdir -p ../content/posts
          cp -r _posts/* ../content/posts/
          
          echo "Content synchronized successfully"
      
      - name: Install dependencies
        run: |
          # Add your build tool installation here
          # Examples:
          # npm install
          # pip install -r requirements.txt
          # bundle install
          echo "Installing dependencies..."
      
      - name: Build site
        run: |
          # Add your build commands here
          # Examples:
          # npm run build
          # hugo --minify
          # jekyll build
          echo "Building site..."
      
      - name: Deploy
        run: |
          # Add your deployment commands here
          # Examples:
          # npm run deploy
          # ./deploy.sh
          echo "Deploying site..."
```

### Step 2: Customize for Your Build System

Depending on your site generator, customize the workflow:

#### For Jekyll:
```yaml
      - name: Build with Jekyll
        uses: actions/jekyll-build-pages@v1
        with:
          source: ./
          destination: ./_site
      
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./_site
```

#### For Hugo:
```yaml
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
      
      - name: Build with Hugo
        run: hugo --minify
      
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

#### For Next.js:
```yaml
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build Next.js site
        run: npm run build
      
      - name: Export static site
        run: npm run export
      
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./out
```

### Step 3: Grant Workflow Permissions

Ensure the workflow has necessary permissions:

1. Go to Repository B Settings → Actions → General
2. Under "Workflow permissions", select "Read and write permissions"
3. Check "Allow GitHub Actions to create and approve pull requests" (if needed)

---

## Data Flow and Payload Structure

### When a Markdown File is Pushed to Repository A:

1. **Trigger**: Push to `main`/`master` branch in `_posts/` directory
2. **Detection**: Workflow identifies changed markdown files
3. **API Call**: For each target repository, sends:

```bash
curl -X POST \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Authorization: token ${DISPATCH_TOKEN}" \
  "https://api.github.com/repos/${TARGET_REPO}/dispatches" \
  -d '{
    "event_type": "content-update",
    "client_payload": {
      "repository": "RyanKim17920/Papers2Code-updates",
      "ref": "refs/heads/main",
      "sha": "abc123def456..."
    }
  }'
```

### Payload Structure Received by Repository B:

```json
{
  "event_type": "content-update",
  "client_payload": {
    "repository": "RyanKim17920/Papers2Code-updates",
    "ref": "refs/heads/main",
    "sha": "abc123def456789"
  }
}
```

### Accessing Payload Data in Repository B Workflow:

```yaml
${{ github.event.client_payload.repository }}  # Source repo (owner/name)
${{ github.event.client_payload.ref }}         # Git reference (branch)
${{ github.event.client_payload.sha }}         # Commit SHA
```

---

## Complete Examples

### Example 1: Simple Static Site with Jekyll

**Repository B Structure:**
```
my-website/
├── .github/workflows/content-update.yml
├── _config.yml
├── content/
│   └── _posts/
├── _layouts/
└── index.html
```

**Workflow:**
```yaml
name: Rebuild Jekyll Site

on:
  repository_dispatch:
    types: [content-update]

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Fetch content from Papers2Code-updates
        run: |
          git clone https://github.com/${{ github.event.client_payload.repository }}.git temp
          cd temp && git checkout ${{ github.event.client_payload.sha }}
          cp -r _posts/* ../content/_posts/
      
      - name: Build Jekyll site
        uses: actions/jekyll-build-pages@v1
      
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./_site
```

### Example 2: Node.js-based Blog

**Repository B Structure:**
```
blog-site/
├── .github/workflows/content-update.yml
├── package.json
├── src/
│   └── posts/           # Markdown files go here
├── build/
└── scripts/
    └── sync-content.js  # Custom sync script
```

**Workflow:**
```yaml
name: Update and Deploy Blog

on:
  repository_dispatch:
    types: [content-update]

jobs:
  update-content:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Sync content from source repo
        env:
          SOURCE_REPO: ${{ github.event.client_payload.repository }}
          SOURCE_SHA: ${{ github.event.client_payload.sha }}
        run: |
          # Clone source repo
          git clone https://github.com/$SOURCE_REPO.git source-content
          cd source-content
          git checkout $SOURCE_SHA
          
          # Copy markdown files
          cp -r _posts/* ../src/posts/
          
          # Parse frontmatter and generate JSON index
          node ../scripts/sync-content.js
      
      - name: Build site
        run: |
          npm ci
          npm run build
      
      - name: Deploy to Vercel
        run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
```

### Example 3: Multiple Repository Support

**Repository B Workflow (Advanced):**
```yaml
name: Aggregate Content from Multiple Sources

on:
  repository_dispatch:
    types: [content-update]

jobs:
  aggregate-content:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Identify source repository
        id: identify
        run: |
          SOURCE="${{ github.event.client_payload.repository }}"
          echo "source=$SOURCE" >> $GITHUB_OUTPUT
          
          # Map source repo to content directory
          case "$SOURCE" in
            "RyanKim17920/Papers2Code-updates")
              echo "target_dir=content/papers" >> $GITHUB_OUTPUT
              ;;
            "OtherUser/tutorials-repo")
              echo "target_dir=content/tutorials" >> $GITHUB_OUTPUT
              ;;
            *)
              echo "target_dir=content/general" >> $GITHUB_OUTPUT
              ;;
          esac
      
      - name: Fetch and organize content
        run: |
          git clone https://github.com/${{ steps.identify.outputs.source }}.git temp
          cd temp && git checkout ${{ github.event.client_payload.sha }}
          mkdir -p ../${{ steps.identify.outputs.target_dir }}
          cp -r _posts/* ../${{ steps.identify.outputs.target_dir }}/
      
      - name: Build and deploy
        run: |
          npm ci
          npm run build
          npm run deploy
```

---

## Advanced Use Cases

### Use Case 1: Selective Content Sync

Only sync specific categories or tags:

```yaml
- name: Sync specific content
  run: |
    git clone https://github.com/${{ github.event.client_payload.repository }}.git source
    cd source && git checkout ${{ github.event.client_payload.sha }}
    
    # Only copy files with specific frontmatter
    for file in _posts/*.md; do
      if grep -q "categories: \[.*tutorials.*\]" "$file"; then
        cp "$file" ../content/posts/
      fi
    done
```

### Use Case 2: Content Transformation

Transform markdown files before building:

```yaml
- name: Transform content
  run: |
    # Convert frontmatter format
    python scripts/transform-frontmatter.py
    
    # Optimize images
    find content/posts -type f -name "*.jpg" -exec mogrify -resize 800x600 {} \;
    
    # Generate metadata index
    python scripts/generate-index.py
```

### Use Case 3: Multi-stage Deployment

Deploy to staging first, then production:

```yaml
jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    steps:
      # ... fetch and build steps ...
      
      - name: Deploy to staging
        run: npm run deploy:staging
  
  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy to production
        run: npm run deploy:production
```

### Use Case 4: Content Validation

Validate content before deploying:

```yaml
- name: Validate markdown files
  run: |
    npm install -g markdownlint-cli
    markdownlint content/posts/*.md
    
    # Validate frontmatter
    python scripts/validate-frontmatter.py
    
    # Check for broken links
    npm run check-links
```

---

## Troubleshooting

### Problem: Repository B workflow not triggering

**Possible causes:**
1. `DISPATCH_TOKEN` doesn't have `repo` scope
2. Target repository name in `TARGET_REPOSITORIES` is incorrect
3. Repository B workflow file has wrong event type

**Solutions:**
- Verify token permissions in GitHub Settings → Developer settings
- Check workflow logs in Repository A for API response codes
- Ensure event type is `content-update` in both repositories
- Check Repository B has the workflow file in `.github/workflows/`

### Problem: Cannot access content from Repository A

**Possible causes:**
1. Repository A is private and Repository B doesn't have access
2. Commit SHA in payload is incorrect
3. Network/authentication issues

**Solutions:**
- If Repository A is private, add a deploy key or use a PAT with read access
- Verify the SHA in Repository B workflow logs
- Add error handling in clone/checkout steps

### Problem: Build fails after content sync

**Possible causes:**
1. Incompatible markdown format or frontmatter structure
2. Missing required fields in frontmatter
3. File paths don't match expected structure

**Solutions:**
- Add content validation step before building
- Transform/normalize content after syncing
- Document required frontmatter structure in Repository A README

---

## Security Considerations

1. **Token Security**: Store `DISPATCH_TOKEN` as a repository secret, never commit it
2. **Scope Limitation**: Grant minimum necessary permissions (only `repo` scope needed)
3. **Content Validation**: Validate content before building to prevent malicious code execution
4. **Access Control**: Limit who can push to Repository A's main branch
5. **Audit Trail**: Monitor workflow runs in both repositories for unexpected activity

---

## Summary Checklist

### Repository A (Papers2Code-updates):
- [x] Workflow configured at `.github/workflows/trigger-website-build.yml`
- [ ] `DISPATCH_TOKEN` secret added with `repo` scope
- [ ] `TARGET_REPOSITORIES` secret configured with target repos
- [ ] Test push to verify workflow triggers

### Repository B (Target Website):
- [ ] Workflow file created at `.github/workflows/content-update.yml`
- [ ] Event type matches: `repository_dispatch.types: [content-update]`
- [ ] Steps to fetch content from Repository A implemented
- [ ] Build and deployment steps configured
- [ ] Workflow permissions set correctly
- [ ] Test by manually triggering dispatch event

---

## Additional Resources

- [GitHub Repository Dispatch Documentation](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event)
- [GitHub Actions Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Creating Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

---

For questions or issues, please open an issue in the Papers2Code-updates repository.
