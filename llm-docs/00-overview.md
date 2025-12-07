# Portfolio Application - LLM Documentation Overview

**Purpose**: This documentation is optimized for LLM agent consumption to enable efficient code generation, debugging, and modifications across the portfolio application.

## System Architecture

This is a **hybrid headless WordPress portfolio application** consisting of three repositories:

### 1. nextjs-client (Frontend)
- **Path**: `C:\Portfolio\nextjs-client`
- **Purpose**: Modern Next.js 15 frontend for portfolio display
- **Tech**: Next.js 15, React 18, TypeScript, Tailwind CSS
- **Deployment**: cPanel via FTPS (GitHub Actions)
- **URL Pattern**: `https://staging.shaganplaatjies.co.za`

### 2. wordpress-plugin (Backend - Content Management)
- **Path**: `C:\Portfolio\wordpress-plugin`
- **Purpose**: Custom post type "project" with REST API exposure
- **Tech**: PHP 7.4+, WordPress 6.0+
- **Deployment**: cPanel via FTPS (GitHub Actions)
- **API Base**: `/wp-json/wp/v2`

### 3. wordpress-theme (Backend - Traditional Display)
- **Path**: `C:\Portfolio\wordpress-theme`
- **Purpose**: WordPress theme (traditional + iframe client support)
- **Tech**: PHP 8.0+, Tailwind CSS, Laravel Mix, Bedrock structure
- **Deployment**: cPanel via FTPS (GitHub Actions)

### 4. portfolio-docs (Documentation)
- **Path**: `C:\Portfolio\portfolio-docs`
- **Purpose**: Comprehensive project documentation
- **Audience**: LLM agents + human developers

## Data Flow Architecture

```
Content Creation:
WordPress Admin → Custom Post Type "Project" (Plugin) → WordPress Database

Content Delivery:
WordPress Database → REST API Endpoint → Next.js Server-Side Fetch (1h cache) → React Components → User

Traditional Route (Optional):
WordPress Database → WordPress Theme PHP Templates → User
```

## Repository Relationships

1. **wordpress-plugin** ↔ **nextjs-client**
   - Plugin exposes REST API
   - Next.js consumes API endpoints
   - Shared data contract via WordPress REST API schema

2. **wordpress-theme** ↔ **nextjs-client**
   - Theme can embed client via iframe
   - PostMessage communication for routing
   - Optional traditional WordPress rendering

3. **wordpress-plugin** ↔ **wordpress-theme**
   - Plugin registers custom post type
   - Theme can display projects via traditional templates
   - Shared WordPress environment

## Key Integration Points

### WordPress REST API Contract
- **Base URL**: `https://staging.blog.shaganplaatjies.co.za/wp-json/wp/v2`
- **Endpoints**: `/posts`, `/projects`, `/media`, `/categories`, `/tags`
- **Authentication**: HTTP Basic Auth with application passwords (write operations only)
- **Response Format**: WordPress REST API v2 schema

### Custom Post Type: "project"
- **Slug**: `project`
- **REST Base**: `/projects`
- **Custom Fields** (exposed via REST API):
  - `_project_role` - User's role
  - `_project_company` - Company name
  - `_project_company_url` - Company website
  - `_project_source_url` - Project demo/repo URL
  - `_project_gallery` - Comma-separated media IDs
  - `_project_gallery_captions` - JSON caption mapping
  - `_project_date_type` - "single" or "range"
  - `_project_date_format` - "yyyy", "mm/yyyy", or "dd/mm/yyyy"
  - `_project_date_start` - Start date (YYYY-MM-DD)
  - `_project_date_end` - End date (YYYY-MM-DD, optional)

### Categories (Content Classification)
- **Blog Post** (ID: 37) - Blog content
- **Project** (ID: 38) - Portfolio projects
- **Work Experience** (ID: 41) - Employment history
- **Uncategorized** (ID: 1) - Default category

## Environment Configuration

### Next.js Client
```env
# WordPress Integration
WP_DOMAIN=staging.blog.shaganplaatjies.co.za
WP_JSON_API_URI=/wp-json/wp/v2
WP_POSTS_URI=/wp-json/wp/v2/posts

# Application
NODE_ENV=development|production
APP_PORT=3000
APP_HOST=staging.shaganplaatjies.co.za
APP_HOST_URL=https://staging.shaganplaatjies.co.za

# Services
RESEND_API_KEY=<email_service_key>
PUBLIC_RESUME_URL=<resume_download_url>

# CORS
ALLOWED_ORIGIN=https://staging.shaganplaatjies.co.za
```

### WordPress Plugin
```env
WP_JWT_TOKEN=<base64_encoded_credentials>
WP_URL=https://staging.blog.shaganplaatjies.co.za
```

### WordPress Theme
```env
# Bedrock structure
DB_NAME=<database_name>
DB_USER=<database_user>
DB_PASSWORD=<database_password>
DB_HOST=localhost
WP_ENV=development|staging|production
WP_HOME=<site_url>
WP_SITEURL=${WP_HOME}/wp

# Theme
THEME_KEY=shaganplaatjies
THEME_NAME=Shagan Plaatjies
```

## Development Workflow

### Setup Order
1. WordPress Theme + Plugin (backend first)
2. Next.js Client (frontend consuming backend)
3. Content creation via WordPress admin
4. Testing on staging environment
5. Deploy to production

### Local Development
```bash
# WordPress Theme
cd wordpress-theme/app/bedrock/web/app/themes/shaganplaatjies
npm install
npm run watch

# WordPress Plugin
# No build required (pure PHP)

# Next.js Client
cd nextjs-client
npm install
npm run dev
```

### Deployment
- **Trigger**: Push to `main` (production) or `stg` (staging)
- **Method**: GitHub Actions → FTPS → cPanel
- **Build**: Automated via CI/CD
- **Cache**: 1-hour TTL on Next.js API cache

## CI/CD Pipeline Summary

All three repositories have GitHub Actions workflows:

1. **Build Stage**
   - Install dependencies
   - Run linters (ESLint, Prettier)
   - Run tests (Jest)
   - Build production assets

2. **Deploy Stage**
   - Upload via FTPS to cPanel
   - Exclude source files, dev dependencies
   - Include only production-ready files
   - Trigger PM2 restart (Next.js only)

## Documentation Structure

```
llm-docs/
├── 00-overview.md              (this file)
├── 01-architecture.md          (detailed architecture)
├── 02-data-contracts.md        (API schemas and contracts)
├── 03-nextjs-client.md         (Next.js implementation details)
├── 04-wordpress-plugin.md      (Plugin implementation details)
├── 05-wordpress-theme.md       (Theme implementation details)
├── 06-deployment.md            (CI/CD and deployment)
├── 07-development-workflow.md  (development patterns)
└── 08-code-conventions.md      (code standards and rules)
```

## Quick Reference

### Common Tasks

**Add new project**:
```bash
cd wordpress-plugin
node bulk-upload.js  # Uses projects.csv
```

**Fetch projects in Next.js**:
```typescript
import { fetchWpProjects } from '@/app/lib/server-lib';
const projects = await fetchWpProjects();
```

**Deploy to staging**:
```bash
git push origin stg
```

**Deploy to production**:
```bash
git push origin main
```

### Key Files

| Repository | File | Purpose |
|------------|------|---------|
| nextjs-client | `app/lib/server-lib.ts` | WordPress API integration |
| nextjs-client | `app/lib/wordpress-types.ts` | TypeScript type definitions |
| nextjs-client | `.github/workflows/deploy-to-cpanel.yml` | CI/CD pipeline |
| wordpress-plugin | `portfolio-plugin.php` | Plugin entry point |
| wordpress-plugin | `includes/class-meta-boxes.php` | Admin UI and meta handling |
| wordpress-plugin | `bulk-upload.js` | Bulk project creation |
| wordpress-theme | `functions.php` | Theme initialization |
| wordpress-theme | `webpack.mix.js` | Build configuration |
| wordpress-theme | `.github/workflows/build-and-deploy.yml` | CI/CD pipeline |

## Technical Highlights

1. **Comprehensive CI/CD**: All repos have automated testing and deployment
2. **Type Safety**: TypeScript throughout Next.js client
3. **Caching Strategy**: 1-hour server-side cache for WordPress API
4. **Responsive Design**: Mobile-first Tailwind CSS
5. **SEO Optimization**: Server-side rendering with Next.js
6. **Security**: Nonce verification, capability checks, input sanitization
7. **Modern Tooling**: Webpack, Babel, PostCSS, ESLint, Prettier, Jest
8. **Bulk Operations**: Automated project creation and media upload
9. **Screenshot Capture**: Playwright-based responsive screenshots
10. **AI-First Development**: Optimized for LLM-assisted development

## Next Steps

Read the detailed documentation files in order:
1. `01-architecture.md` - Understand the complete architecture
2. `02-data-contracts.md` - Learn the API contracts and data models
3. `03-nextjs-client.md` - Next.js implementation specifics
4. `04-wordpress-plugin.md` - WordPress plugin implementation
5. `05-wordpress-theme.md` - WordPress theme implementation
6. `06-deployment.md` - Deployment processes
7. `07-development-workflow.md` - Development patterns
8. `08-code-conventions.md` - Code standards and conventions
