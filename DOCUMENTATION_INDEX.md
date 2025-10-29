# Documentation Index

Complete guide to Papers2Code-updates repository and how it connects to target repositories.

## 🚀 Quick Start

**New to this repository?** Start here:

1. **[README.md](README.md)** - Overview and basic setup
2. **[Repository B Setup Guide](examples/REPOSITORY_B_SETUP.md)** - Quick reference for setting up target repositories
3. **[Example Workflows](examples/target-repository-workflows/)** - Choose a template for your tech stack

## 📖 Comprehensive Documentation

### Core Concepts

- **[Integration Guide](INTEGRATION_GUIDE.md)** - Complete guide covering:
  - How the connection works (Repository Dispatch API)
  - Setting up Repository A (this repo)
  - Setting up Repository B (target websites)
  - Data flow and payload structure
  - Complete examples for Jekyll, Hugo, Next.js
  - Advanced use cases (multi-source, transformation, validation)
  - Troubleshooting and security considerations

- **[Connection Diagram](CONNECTION_DIAGRAM.md)** - Visual explanation covering:
  - Architecture overview with diagrams
  - Step-by-step data flow
  - Component explanations (PAT, Repository Dispatch, Commit SHA)
  - Security and permissions
  - Alternative architectures

### Reference Guides

- **[Repository B Setup](examples/REPOSITORY_B_SETUP.md)** - Quick reference covering:
  - Step-by-step setup checklist
  - Minimal required workflow
  - Common repository structures
  - Payload data structure
  - Troubleshooting common issues

- **[FAQ](FAQ.md)** - Frequently asked questions covering:
  - General questions about the repository pattern
  - Setup and configuration
  - Technical implementation details
  - Content management
  - Workflow customization
  - Security considerations
  - Debugging and troubleshooting

### Example Workflows

Located in [examples/target-repository-workflows/](examples/target-repository-workflows/):

1. **[basic-example.yml](examples/target-repository-workflows/basic-example.yml)**
   - Simplest starting point
   - Customizable for any build system
   - Well-commented placeholder steps

2. **[jekyll-github-pages.yml](examples/target-repository-workflows/jekyll-github-pages.yml)**
   - Complete Jekyll + GitHub Pages setup
   - Ruby and Bundler configuration
   - Automatic deployment

3. **[hugo-example.yml](examples/target-repository-workflows/hugo-example.yml)**
   - Hugo static site generator
   - Theme support via git submodules
   - Optimized build configuration

4. **[nextjs-example.yml](examples/target-repository-workflows/nextjs-example.yml)**
   - Next.js with Node.js
   - Supports Vercel or GitHub Pages
   - Static export configuration

5. **[advanced-multi-source.yml](examples/target-repository-workflows/advanced-multi-source.yml)**
   - Aggregate content from multiple repositories
   - Dynamic routing based on source
   - Content transformation and validation

Each example includes:
- ✅ Complete, working workflow configuration
- ✅ Detailed comments explaining each step
- ✅ Customization instructions
- ✅ Build and deployment setup

## 📁 Repository Structure

```
Papers2Code-updates/
├── README.md                          # Main overview and quick start
├── INTEGRATION_GUIDE.md               # Comprehensive integration guide
├── CONNECTION_DIAGRAM.md              # Visual explanation of architecture
├── FAQ.md                             # Frequently asked questions
├── DOCUMENTATION_INDEX.md             # This file - documentation overview
│
├── _posts/                            # Content directory
│   └── sample-update.md              # Example blog post with frontmatter
│
├── .github/
│   └── workflows/
│       └── trigger-website-build.yml # Workflow that sends notifications
│
└── examples/
    ├── REPOSITORY_B_SETUP.md         # Quick setup reference
    ├── FASTAPI_REACT_VERCEL_GUIDE.md # FastAPI + React + Vercel guide
    └── target-repository-workflows/   # Example workflows for target repos
        ├── README.md                  # Guide to choosing examples
        ├── basic-example.yml
        ├── jekyll-github-pages.yml
        ├── hugo-example.yml
        ├── nextjs-example.yml
        ├── fastapi-react-vercel.yml
        └── advanced-multi-source.yml
```

## 🎯 Use Cases and Examples

### Use Case 1: Simple Blog with Jekyll
**Documentation:** [Jekyll Example](examples/target-repository-workflows/jekyll-github-pages.yml)

Setup a Jekyll blog that automatically rebuilds when you add posts to this repository.

**You need:**
- This repository configured with secrets
- Jekyll site repository with the example workflow
- GitHub Pages enabled

**Time to setup:** ~15 minutes

---

### Use Case 2: Documentation Site with Hugo
**Documentation:** [Hugo Example](examples/target-repository-workflows/hugo-example.yml)

Create a documentation site that stays in sync with content updates.

**You need:**
- This repository configured with secrets
- Hugo site repository with the example workflow
- Hugo theme configured

**Time to setup:** ~20 minutes

---

### Use Case 3: Modern Web App with Next.js
**Documentation:** [Next.js Example](examples/target-repository-workflows/nextjs-example.yml)

Build a modern web application that fetches and displays blog content.

**You need:**
- This repository configured with secrets
- Next.js application with the example workflow
- Deployment target (Vercel or GitHub Pages)

**Time to setup:** ~25 minutes

---

### Use Case 4: Full-Stack App with FastAPI + React on Vercel
**Documentation:** [FastAPI + React + Vercel Guide](examples/FASTAPI_REACT_VERCEL_GUIDE.md)

Build a full-stack application with FastAPI backend and React frontend that automatically updates content.

**You need:**
- This repository configured with secrets
- FastAPI + React project with the workflow
- Vercel project configured
- Vercel token or deploy hook

**Time to setup:** ~30 minutes

---

### Use Case 5: Multi-Source Content Hub
**Documentation:** [Advanced Multi-Source Example](examples/target-repository-workflows/advanced-multi-source.yml)

Aggregate content from multiple repositories into a single website.

**You need:**
- Multiple content repositories (like this one)
- Website repository with the advanced workflow
- Content categorization strategy

**Time to setup:** ~45 minutes

---

## 🔧 Configuration Reference

### Repository A (This Repository)

**Required Secrets:**
```
DISPATCH_TOKEN       - Personal Access Token with 'repo' scope
TARGET_REPOSITORIES  - Comma-separated list: "owner/repo1,owner/repo2"
```

**Workflow File:**
```
.github/workflows/trigger-website-build.yml
```

**Triggers:**
- Push to main/master branch
- Changes to _posts/*.md files
- Manual dispatch via Actions UI

---

### Repository B (Target Website)

**Required File:**
```
.github/workflows/content-update.yml
```

**Event Listener:**
```yaml
on:
  repository_dispatch:
    types: [content-update]
```

**Payload Received:**
```json
{
  "repository": "RyanKim17920/Papers2Code-updates",
  "ref": "refs/heads/main",
  "sha": "abc123..."
}
```

---

## 🐛 Troubleshooting

Quick links to troubleshooting sections:

- **[Workflow Not Triggering](FAQ.md#q-repository-b-workflow-didnt-trigger-why)** - Common causes and solutions
- **[Content Not Syncing](FAQ.md#q-can-i-see-what-changed-before-rebuilding)** - Debug content fetch issues
- **[Build Failures](FAQ.md#q-what-if-the-workflow-fails)** - Diagnose build problems
- **[Authentication Errors](INTEGRATION_GUIDE.md#troubleshooting)** - Token and permission issues
- **[Rate Limits](FAQ.md#q-what-are-the-rate-limits)** - API usage limits

For more help:
1. Check the [FAQ](FAQ.md)
2. Review the [Integration Guide](INTEGRATION_GUIDE.md) troubleshooting section
3. Check workflow logs in both repositories
4. Open an issue in this repository

---

## 🔐 Security

**Important security considerations:**

- ✅ Store `DISPATCH_TOKEN` as repository secret (never commit)
- ✅ Use minimum required token scope (`repo`)
- ✅ Validate content before building in Repository B
- ✅ Rotate tokens periodically
- ✅ Monitor workflow logs for suspicious activity

See [Integration Guide - Security Considerations](INTEGRATION_GUIDE.md#security-considerations) for details.

---

## 📚 Additional Resources

### GitHub Documentation
- [Repository Dispatch Events](https://docs.github.com/en/rest/repos/repos#create-a-repository-dispatch-event)
- [GitHub Actions Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Creating Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [Encrypted Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

### Static Site Generators
- [Jekyll Documentation](https://jekyllrb.com/docs/)
- [Hugo Documentation](https://gohugo.io/documentation/)
- [Next.js Documentation](https://nextjs.org/docs)

### GitHub Actions
- [Actions Marketplace](https://github.com/marketplace?type=actions)
- [Workflow Examples](https://github.com/actions/starter-workflows)

---

## 🤝 Contributing

Found an issue or want to improve the documentation?

1. Check existing issues first
2. Open a new issue with details
3. Submit a pull request with improvements

We welcome:
- Documentation improvements
- New workflow examples
- Bug fixes
- Use case examples
- FAQ additions

---

## 📝 License

This repository is part of the Papers2Code project.

---

## 📞 Support

Need help?

1. **Check documentation first:**
   - [FAQ](FAQ.md) - Common questions
   - [Integration Guide](INTEGRATION_GUIDE.md) - Comprehensive guide
   - [Repository B Setup](examples/REPOSITORY_B_SETUP.md) - Quick reference

2. **Check workflow logs:**
   - Repository A: Actions → "Trigger Website Build"
   - Repository B: Actions → "Rebuild on Content Update"

3. **Open an issue:**
   - Provide workflow logs
   - Describe expected vs actual behavior
   - Include repository configuration (without secrets)

---

## 🗺️ Documentation Map

```
Start Here
    ↓
README.md ──────────────────┐
    ↓                       │
    ├─ Quick Start          │
    ├─ Features            │
    └─ Basic Setup         │
         ↓                  │
         │                  │
Need More Detail?          │
    ↓                       │
INTEGRATION_GUIDE.md       │
    ↓                       │
    ├─ How It Works        │
    ├─ Complete Setup      │
    ├─ Examples            │
    └─ Advanced Topics     │
         ↓                  │
         │                  │
Visual Learner?            │
    ↓                       │
CONNECTION_DIAGRAM.md      │
    ↓                       │
    ├─ Architecture        │
    ├─ Data Flow          │
    └─ Components         │
         ↓                  │
         │                  │
Ready to Implement?        │
    ↓                       │
REPOSITORY_B_SETUP.md ─────┘
    ↓
    ├─ Step-by-Step
    ├─ Choose Template
    └─ Test Integration
         ↓
         │
examples/target-repository-workflows/
    ↓
    ├─ basic-example.yml
    ├─ jekyll-github-pages.yml
    ├─ hugo-example.yml
    ├─ nextjs-example.yml
    └─ advanced-multi-source.yml
         ↓
         │
Have Questions?
    ↓
FAQ.md
    ↓
    ├─ General
    ├─ Setup
    ├─ Technical
    ├─ Content
    └─ Security
```

---

**Last Updated:** 2025-10-29

For the most recent version of this documentation, visit:
https://github.com/RyanKim17920/Papers2Code-updates
