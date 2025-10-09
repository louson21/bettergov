# BetterGov.ph Copilot Instructions

## Repository Overview

**BetterGov.ph** is a community-led initiative to create a better and more usable Philippine national government website. It's a modern React-based web application that provides comprehensive government services information, directory, and resources for Philippine citizens.

### High-Level Project Information
- **Project Type**: React single-page application (SPA) with TypeScript
- **Languages**: TypeScript/JavaScript (React), Node.js (functions), Shell scripts
- **Main Framework**: React 19.1.0 with Vite as build tool
- **Runtime**: Node.js v22.16.0 (see .nvmrc)
- **Styling**: Tailwind CSS v4.1.13
- **Testing**: Playwright for E2E tests
- **Search**: Meilisearch integration
- **Deployment**: Docker containerization, Cloudflare Workers for functions
- **Repository Size**: ~150+ files, medium-sized project

## Build and Development Instructions

### Essential Prerequisites
- **Node.js v22.16.0** (enforced by .nvmrc - always use `nvm use` or `fnm use`)
- **npm v10+** (comes with Node.js)
- Git for version control

### Critical Setup Steps

1. **ALWAYS use the correct Node.js version first:**
   ```bash
   nvm use  # or fnm use
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```
   - This automatically runs `husky` setup via the `prepare` script
   - Expect ~763 packages to be installed
   - Expect 7 vulnerabilities (2 low, 4 moderate, 1 high) - this is normal

3. **Environment configuration:**
   ```bash
   cp .env.example .env
   ```
   - Edit `.env` with your Meilisearch credentials if needed
   - Required for search functionality

### Build and Development Commands

**Development server:**
```bash
npm run dev
# Serves on http://localhost:5173
# Uses Vite with hot reloading
```

**Production build:**
```bash
npm run build
```
- **Build sequence**: `tsc` → `generate:metadata` → `write-ver-to-json.ts` → `vite build`
- Generates `dist/` directory with static files
- Creates `sitemap.xml` and `llms.txt` automatically
- Adds git commit hash to `src/version.json`
- Build time: ~30-60 seconds

**Linting and formatting:**
```bash
npm run lint          # ESLint check (must pass with 0 warnings)
npm run format        # Prettier format
npx prettier --check . # Check formatting (used in CI)
```

**Testing:**
```bash
npm run test:e2e           # Run all E2E tests headlessly  
npm run test:e2e:ui        # Interactive Playwright UI
npm run test:e2e:headed    # Tests with visible browser
npm run test:e2e:debug     # Debug mode with inspector
npm run test:e2e:codegen   # Record new tests
```
- E2E tests run on http://localhost:5173 (auto-started)
- Tests cover: critical flows, navigation, accessibility, performance
- Currently ignore performance.spec.ts and accessibility.spec.ts

### Docker Usage
```bash
# Development with hot-reload
docker-compose up
# Serves on http://localhost:5173

# Production build and serve
docker build -t bettergov .
docker run -p 8080:80 bettergov
# Serves on http://localhost:8080
```

### Common Issues and Workarounds

1. **Prettier formatting failures:** 
   - `src/version.json` often has formatting issues after build
   - Fix: `npx prettier --write src/version.json`

2. **Node.js version mismatch:**
   - ALWAYS run `nvm use` before any npm commands
   - The project requires exactly v22.16.0

3. **Build failures after dependency changes:**
   - Clear node_modules: `rm -rf node_modules package-lock.json && npm install`

4. **E2E test failures:**
   - Ensure dev server is running on port 5173
   - Performance and accessibility tests are currently ignored

## Project Architecture and Layout

### Root Directory Structure
```
├── .github/                    # GitHub workflows and config
├── .husky/                     # Git hooks (pre-commit, commit-msg)
├── docs/                       # Documentation (Meilisearch setup)
├── e2e/                        # Playwright end-to-end tests
├── functions/                  # Cloudflare Workers functions
├── public/                     # Static assets, locales, logos
├── scripts/                    # Data processing and build scripts
├── src/                        # React application source
└── dist/                       # Build output (generated)
```

### Source Code Organization (`src/`)
```
src/
├── components/                 # Reusable React components
├── data/                       # Static data files
├── i18n/                       # Internationalization files
├── lib/                        # Utility libraries
├── pages/                      # Page components (routes)
├── types/                      # TypeScript type definitions
├── utils/                      # Helper functions
├── App.tsx                     # Main app component with routing
├── main.tsx                    # React entry point
├── i18n.ts                     # i18next configuration
└── version.json                # Build version info (auto-generated)
```

### Key Configuration Files
- `vite.config.ts` - Vite build configuration with React and Tailwind
- `tsconfig.app.json` - TypeScript config for app code
- `tsconfig.node.json` - TypeScript config for Node.js scripts
- `playwright.config.ts` - E2E test configuration
- `eslint.config.js` - ESLint rules with TypeScript, React, Prettier
- `commitlint.config.js` - Conventional commits validation
- `.lintstagedrc.js` - Pre-commit hook configuration
- `docker-compose.yml` - Development container setup
- `Dockerfile` - Multi-stage build (dev and production)

### CI/CD and Validation Pipeline

**GitHub Actions workflows (.github/workflows/):**
1. **build-test.yml** (main validation):
   - **Build image**: Docker build with commit hash
   - **Lint checks**: ESLint + Prettier formatting
   - **E2E tests**: Playwright tests across 4 shards (Chrome, Firefox, Safari, Mobile)
   - **Report merging**: Combines test reports

2. **Additional workflows**:
   - `stale.yml` - Stale issue management
   - `github-action-linter.yml` - GitHub Actions validation
   - `index-meilisearch.yml` - Search index updates

**Pre-commit validation (Husky):**
- **Pre-commit**: `lint-staged` runs ESLint fix + Prettier on changed files
- **Commit message**: `commitlint` validates conventional commit format

### Git Workflow Requirements
- **Conventional Commits** required (enforced by commitlint)
- **Format**: `<type>[scope]: <description>`
- **Types**: feat, fix, docs, style, refactor, test, chore
- **Examples**: 
  - `feat(search): add meilisearch integration`
  - `fix(navbar): correct mobile menu toggle`
  - `docs: update contributing guide`

### Search and Data Processing

The project includes comprehensive data processing scripts in `scripts/`:
- **Meilisearch indexing**: `index_meilisearch.cjs`, `create_search_key.cjs`
- **Flood control data**: `index_flood_control_arcgis.js`
- **Contractor profiles**: `fetch_contractor_profiles.cjs`, `process_contractors.cjs`
- **Site generation**: `generate-sitemap.js`, `generate-llms-txt.js`

**Search setup requires**:
1. Meilisearch server installation (see docs/Meilisearch.md)
2. Environment variables in .env
3. Data indexing via npm scripts

### Functions (Cloudflare Workers)
```
functions/
├── api/                       # API endpoints
├── lib/                       # Shared utilities  
├── index.ts                   # Main worker entry point
└── types.ts                   # Worker type definitions
```

**Development commands:**
```bash
npm run functions:dev          # Local development server
npm run functions:build        # TypeScript compilation
npm run functions:deploy       # Deploy to Cloudflare
```

### Internationalization (i18n)
- **Framework**: react-i18next
- **Languages**: English (primary), Filipino/Tagalog
- **Locale files**: `public/locales/[lang]/[namespace].json`
- **Auto-detection**: Browser language detection enabled

## Validation Steps for Code Changes

1. **Always run before committing:**
   ```bash
   npm run lint
   npx prettier --check .
   npm run build
   ```

2. **For component changes:**
   ```bash
   npm run test:e2e:headed  # Visual confirmation
   ```

3. **For search-related changes:**
   - Verify Meilisearch integration still works
   - Test search functionality manually

4. **For Docker changes:**
   ```bash
   docker-compose up  # Test development container
   ```

## Important Notes for Coding Agents

1. **Trust these instructions** - Only search for additional information if these instructions are incomplete or incorrect

2. **Always use exact Node.js version** - Run `nvm use` before any npm commands

3. **Respect the build pipeline** - The build process has specific steps that must complete in order

4. **Follow conventional commits** - All commits must follow the conventional commits specification

5. **Test thoroughly** - E2E tests must pass; formatting must be correct

6. **Docker is optional for development** - Use `docker-compose up` for containerized development, or `npm run dev` for local

7. **Search functionality requires setup** - Meilisearch must be configured for full functionality

8. **The project is actively maintained** - Check for recent changes in package.json and workflows before making assumptions about dependencies or processes