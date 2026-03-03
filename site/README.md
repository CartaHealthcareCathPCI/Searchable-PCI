# NCDR CathPCI® Registry Coding Guidance Search

A comprehensive search engine for NCDR CathPCI Registry coding guidance, built by Carta Healthcare.

## Features

- **Single-file deployment** - All data embedded, no external dependencies
- **Multi-source search** - Search across Data Dictionary, Supporting Definitions, Additional Coding Directives, and Pending Updates
- **Real-time filtering** - Filter by source type with instant results
- **Smart formatting** - Automatic text cleaning and organization
- **Search highlighting** - Search terms highlighted in results
- **Mobile responsive** - Works seamlessly on all devices

## Data Sources

This search engine includes 608 searchable items from:
- **Data Dictionary** (346 main definitions)
- **Supporting Definitions** (95 definitions)
- **Additional Coding Directives** (145 directives)
- **Pending Definition Updates** (22 updates)

## Deployment to GitHub Pages

This repository is configured to automatically deploy to GitHub Pages when you push to the `main` branch.

### Setup Instructions

1. **Create a new GitHub repository**
   - Go to https://github.com/new
   - Name your repository (e.g., `cathpci-coding-search`)
   - Make it public or private (GitHub Pages works with both)
   - Don't initialize with README, .gitignore, or license

2. **Upload files to GitHub**
   - Upload all files from this folder to your repository
   - Make sure the `.github/workflows/deploy.yml` file is included

3. **Enable GitHub Pages**
   - Go to your repository Settings
   - Navigate to "Pages" in the left sidebar
   - Under "Source", select "GitHub Actions"
   - Save the settings

4. **Deploy**
   - Push to the `main` branch or click "Actions" → "Deploy to GitHub Pages" → "Run workflow"
   - The site will be available at: `https://[your-username].github.io/[repository-name]/`

## File Structure

```
.
├── index.html              # Main search interface
├── .github/
│   └── workflows/
│       └── deploy.yml      # GitHub Actions deployment workflow
└── README.md               # This file
```

## Local Development

To test locally, simply open `index.html` in a web browser. No build process or server required!

## Updates

To update the search data:
1. Modify the embedded data in `index.html`
2. Commit and push to the `main` branch
3. GitHub Actions will automatically redeploy

## Branding

This tool features Carta Healthcare branding with official logo and uses Space Grotesk font for headings.

---

**Built by Carta Healthcare** | NCDR CathPCI® is a registered trademark
