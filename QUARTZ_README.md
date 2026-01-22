# Quartz Deployment Guide

This guide explains how to deploy your LLM World Engine Knowledge Base using Quartz v4.

## 📋 What's Been Set Up

✅ **Quartz v4 installed** in `quartz-site/` directory
✅ **Content copied** from `obsidian-analysis/` to `quartz-site/content/`
✅ **Configuration customized** for LLM World Engine Knowledge Base
✅ **Custom index page** created with navigation
✅ **Build tested** locally

## 🚀 Deployment Options

### Option 1: GitHub Pages (Recommended - Free)

#### Prerequisites
- GitHub account
- Git installed locally
- Your repository on GitHub

#### Step-by-Step Deployment

1. **Create a new GitHub repository** (if you haven't already):
   ```bash
   # On GitHub.com, create a new repository named "llm-world-engine"
   # DO NOT initialize with README, .gitignore, or license
   ```

2. **Configure Quartz for your GitHub username**:
   ```bash
   # Edit quartz-site/quartz.config.ts
   # Change line 19 from:
   baseUrl: "your-username.github.io/llm-world-engine",
   # To:
   baseUrl: "YOUR-ACTUAL-USERNAME.github.io/llm-world-engine",
   ```

3. **Initialize git in the quartz-site directory**:
   ```bash
   cd quartz-site
   git init
   git add .
   git commit -m "Initial Quartz setup with LLM World Engine content"
   ```

4. **Connect to GitHub and push**:
   ```bash
   git remote add origin https://github.com/YOUR-USERNAME/llm-world-engine.git
   git branch -M main
   git push -u origin main
   ```

5. **Deploy to GitHub Pages**:

   I've added a GitHub Action that automatically builds and deploys your site whenever you push to the `main` branch.

   To trigger the first deployment, simply push your changes:
   ```bash
   npx quartz sync
   ```

   This will:
   - Commit any local changes
   - Pull latest from GitHub
   - Push your content to the `main` branch
   - Trigger the GitHub Action to build and deploy to your site

6. **Enable GitHub Pages**:
   - Go to your repository on GitHub
   - Click **Settings** → **Pages**
   - Under "Build and deployment" > "Source", select: **GitHub Actions**
   - (No need to select a branch, the workflow handles it)

7. **Access your site**:
   - Your site will be available at: `https://YOUR-USERNAME.github.io/llm-world-engine/`
   - It may take 2-5 minutes for the first deployment

#### Updating Your Site

Whenever you want to update the content:

```bash
cd quartz-site

# Update content in content/ directory
# Then sync to GitHub Pages:
npx quartz sync
```

## ⚙️ Configuration Options

### Edit Site Settings

File: `quartz-site/quartz.config.ts`

```typescript
configuration: {
  pageTitle: "Your Site Title",           // Change site title
  baseUrl: "your-site.github.io",         // Your deployment URL
  ignorePatterns: ["private", "drafts"],  // Files to exclude
  defaultDateType: "created",             // Date display preference
  // ... more options
}
```

### Customize Theme

Edit colors in `quartz.config.ts`:

```typescript
theme: {
  typography: {
    header: "Schibsted Grotesk",  // Header font
    body: "Source Sans Pro",      // Body font
    code: "IBM Plex Mono",        // Code font
  },
  colors: {
    lightMode: {
      light: "#faf8f8",    // Background
      dark: "#2b2b2b",     // Text
      secondary: "#284b63", // Links
      // ... more colors
    },
    darkMode: {
      // Dark mode colors
    }
  }
}
```

### Enable/Disable Features

In `quartz.config.ts` plugins section:

```typescript
plugins: {
  transformers: [
    Plugin.FrontMatter(),
    Plugin.TableOfContents(),     // Disable TOC
    Plugin.Latex(),                // Disable math rendering
    // ... more plugins
  ],
  emitters: [
    Plugin.ContentPage(),
    Plugin.TagPage(),              // Disable tag pages
    Plugin.ContentIndex({
      enableSiteMap: true,         // Enable/disable sitemap
      enableRSS: true,             // Enable/disable RSS feed
    }),
  ]
}
```

---

## 🎨 Customizing Layout

File: `quartz-site/quartz.layout.ts`

This controls what appears on each page:

```typescript
export const defaultContentPageLayout: PageLayout = {
  beforeBody: [
    Component.Breadcrumbs(),      // Navigation breadcrumbs
    Component.ArticleTitle(),     // Page title
    Component.TagList(),          // Tags
  ],
  left: [
    Component.PageTitle(),        // Site title
    Component.Search(),           // Search box
    Component.Darkmode(),         // Dark mode toggle
    Component.Explorer(),         // File tree
  ],
  right: [
    Component.Graph(),            // Graph view
    Component.TableOfContents(),  // TOC
    Component.Backlinks(),        // Backlinks
  ],
}
```

Remove components you don't want or reorder them.
