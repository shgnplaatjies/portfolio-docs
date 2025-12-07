# Data Contracts & API Schemas

**Purpose**: Comprehensive API contracts, data models, and type definitions for the portfolio application.

## WordPress REST API Base Configuration

### Endpoints Base URL
```
Production: https://blog.shaganplaatjies.co.za/wp-json/wp/v2
Staging:    https://staging.blog.shaganplaatjies.co.za/wp-json/wp/v2
```

### Authentication

**Public Endpoints** (GET requests):
- No authentication required
- Used by Next.js frontend for content retrieval

**Protected Endpoints** (POST, PUT, DELETE):
```http
Authorization: Basic {base64(username:password)}
```

Example:
```typescript
const credentials = btoa(`${username}:${password}`);
headers: {
  'Authorization': `Basic ${credentials}`,
  'Content-Type': 'application/json'
}
```

## Custom Post Type: "project"

### REST API Endpoint
```
GET    /wp-json/wp/v2/projects          # List all projects
GET    /wp-json/wp/v2/projects/{id}     # Get single project by ID
GET    /wp-json/wp/v2/projects?slug={slug}  # Get by slug
POST   /wp-json/wp/v2/projects          # Create project (auth required)
POST   /wp-json/wp/v2/projects/{id}     # Update project (auth required)
DELETE /wp-json/wp/v2/projects/{id}     # Delete project (auth required)
```

### Query Parameters

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `per_page` | int | Items per page (default: 10) | `?per_page=100` |
| `page` | int | Page number | `?page=2` |
| `categories` | int\|array | Filter by category ID(s) | `?categories=38` |
| `tags` | int\|array | Filter by tag ID(s) | `?tags=42,43` |
| `search` | string | Search term | `?search=react` |
| `orderby` | string | Sort field (date, title, etc.) | `?orderby=date` |
| `order` | string | Sort direction (asc, desc) | `?order=desc` |
| `status` | string | Post status | `?status=publish` |
| `slug` | string | Filter by slug | `?slug=my-project` |

### Response Schema: Project Object

```typescript
interface WpProjectApiResponse {
  id: number;                    // WordPress post ID
  date_gmt: string;              // Publication date (ISO 8601)
  modified_gmt: string;          // Last modified date (ISO 8601)
  slug: string;                  // URL-friendly slug
  status: string;                // "publish", "draft", "pending"
  type: string;                  // "project"
  link: string;                  // Permalink URL
  title: {
    rendered: string;            // HTML-decoded title
  };
  content: {
    rendered: string;            // Full HTML content
    protected: boolean;          // Password protected flag
  };
  excerpt: {
    rendered: string;            // Auto-generated excerpt (HTML)
    protected: boolean;
  };
  featured_media: number;        // Media ID (0 if none)
  categories: number[];          // Category ID array
  tags: number[];                // Tag ID array
  meta: ProjectMeta;             // Custom fields (see below)
  _links: {                      // HAL links
    self: Array<{href: string}>;
    collection: Array<{href: string}>;
    about: Array<{href: string}>;
    "wp:featuredmedia": Array<{embeddable: boolean; href: string}>;
    "wp:attachment": Array<{href: string}>;
    "wp:term": Array<{taxonomy: string; embeddable: boolean; href: string}>;
    curies: Array<{name: string; href: string; templated: boolean}>;
  };
}
```

### Custom Fields (Meta)

```typescript
interface ProjectMeta {
  _project_subtext?: string;      // Brief project description (1-2 sentences)
  _project_role?: string;         // User's role/position
  _project_company?: string;      // Company/organization name
  _project_company_url?: string;  // Company website URL
  _project_source_url?: string;   // Project demo/repo/live URL
  _project_gallery?: string;      // Comma-separated media IDs (e.g., "123,456,789")
  _project_gallery_captions?: string;  // JSON: {"123": "Caption text", "456": "Another"}
  _project_thumbnail?: string;    // Featured image attachment ID
  _project_date_type?: "single" | "range";  // Date display type
  _project_date_format?: "yyyy" | "mm/yyyy" | "dd/mm/yyyy";  // Date format
  _project_date_start?: string;   // Start date (YYYY-MM-DD)
  _project_date_end?: string;     // End date (YYYY-MM-DD), empty for ongoing
}
```

### Example Response

```json
{
  "id": 186,
  "date_gmt": "2025-03-01T00:00:00",
  "modified_gmt": "2024-12-07T10:30:15",
  "slug": "lead-software-engineer",
  "status": "publish",
  "type": "project",
  "link": "https://staging.blog.shaganplaatjies.co.za/projects/lead-software-engineer/",
  "title": {
    "rendered": "Lead Software Engineer at Broadway Media"
  },
  "content": {
    "rendered": "<p>Leading technical delivery for a global entertainment platform...</p>",
    "protected": false
  },
  "excerpt": {
    "rendered": "",
    "protected": false
  },
  "featured_media": 187,
  "categories": [41],
  "tags": [42, 17, 43, 36, 26, 53, 45, 46, 47],
  "meta": {
    "_project_subtext": "Leading technical delivery for global entertainment platform",
    "_project_role": "Lead Software Engineer",
    "_project_company": "Broadway Media",
    "_project_company_url": "https://www.broadwaymedia.com/",
    "_project_source_url": "",
    "_project_gallery": "188,189,190",
    "_project_gallery_captions": "{\"188\":\"Mobile view\",\"189\":\"Desktop interface\",\"190\":\"Admin panel\"}",
    "_project_thumbnail": "187",
    "_project_date_type": "range",
    "_project_date_format": "mm/yyyy",
    "_project_date_start": "2025-03-01",
    "_project_date_end": ""
  }
}
```

### Create/Update Request Body

```json
{
  "title": "Project Title at Company Name",
  "content": "<p>Detailed project description with HTML formatting</p>",
  "status": "publish",
  "categories": [38],
  "tags": [42, 43, 48],
  "featured_media": 123,
  "meta": {
    "_project_subtext": "Brief project tagline",
    "_project_role": "Senior Developer",
    "_project_company": "Company Name",
    "_project_company_url": "https://company.com",
    "_project_source_url": "https://demo.example.com",
    "_project_gallery": "124,125,126",
    "_project_gallery_captions": "{\"124\":\"Mobile view\",\"125\":\"Tablet view\",\"126\":\"Desktop\"}",
    "_project_thumbnail": "123",
    "_project_date_type": "range",
    "_project_date_format": "mm/yyyy",
    "_project_date_start": "2024-01-01",
    "_project_date_end": "2024-12-31"
  }
}
```

## Media Endpoints

### Base Endpoint
```
GET    /wp-json/wp/v2/media          # List all media
GET    /wp-json/wp/v2/media/{id}     # Get single media by ID
POST   /wp-json/wp/v2/media          # Upload media (auth required, multipart/form-data)
DELETE /wp-json/wp/v2/media/{id}     # Delete media (auth required)
```

### Media Response Schema

```typescript
interface WpMediaApiResponse {
  id: number;                    // Media attachment ID
  date_gmt: string;              // Upload date (ISO 8601)
  modified_gmt: string;          // Last modified
  slug: string;                  // Media slug
  status: string;                // "inherit" (inherits from parent post)
  type: string;                  // "attachment"
  link: string;                  // Attachment page URL
  title: {
    rendered: string;            // Media title
  };
  caption: {
    rendered: string;            // Media caption (HTML)
  };
  alt_text: string;              // Alt text for images
  media_type: string;            // "image", "video", "file", "audio"
  mime_type: string;             // "image/jpeg", "image/png", etc.
  media_details: {
    width: number;               // Image width (px)
    height: number;              // Image height (px)
    file: string;                // Filename
    filesize: number;            // File size (bytes)
    sizes: {
      thumbnail?: ImageSize;
      medium?: ImageSize;
      large?: ImageSize;
      full?: ImageSize;
      [key: string]: ImageSize;
    };
    image_meta: {
      aperture: string;
      credit: string;
      camera: string;
      caption: string;
      created_timestamp: string;
      copyright: string;
      focal_length: string;
      iso: string;
      shutter_speed: string;
      title: string;
      orientation: string;
      keywords: string[];
    };
  };
  source_url: string;            // Direct URL to file
  _links: object;                // HAL links
}

interface ImageSize {
  file: string;                  // Filename
  width: number;                 // Width (px)
  height: number;                // Height (px)
  filesize?: number;             // File size (bytes)
  mime_type: string;             // MIME type
  source_url: string;            // Direct URL
}
```

### Example Media Response

```json
{
  "id": 187,
  "date_gmt": "2024-12-05T14:22:30",
  "slug": "broadway-media-logo",
  "status": "inherit",
  "type": "attachment",
  "link": "https://staging.blog.shaganplaatjies.co.za/broadway-media-logo/",
  "title": {
    "rendered": "Broadway Media Logo"
  },
  "caption": {
    "rendered": ""
  },
  "alt_text": "Broadway Media company logo",
  "media_type": "image",
  "mime_type": "image/png",
  "media_details": {
    "width": 1200,
    "height": 630,
    "file": "2024/12/broadway-media-logo.png",
    "filesize": 45678,
    "sizes": {
      "thumbnail": {
        "file": "broadway-media-logo-150x150.png",
        "width": 150,
        "height": 150,
        "mime_type": "image/png",
        "source_url": "https://staging.blog.shaganplaatjies.co.za/wp-content/uploads/2024/12/broadway-media-logo-150x150.png"
      },
      "medium": {
        "file": "broadway-media-logo-300x157.png",
        "width": 300,
        "height": 157,
        "mime_type": "image/png",
        "source_url": "https://staging.blog.shaganplaatjies.co.za/wp-content/uploads/2024/12/broadway-media-logo-300x157.png"
      },
      "full": {
        "file": "broadway-media-logo.png",
        "width": 1200,
        "height": 630,
        "mime_type": "image/png",
        "source_url": "https://staging.blog.shaganplaatjies.co.za/wp-content/uploads/2024/12/broadway-media-logo.png"
      }
    }
  },
  "source_url": "https://staging.blog.shaganplaatjies.co.za/wp-content/uploads/2024/12/broadway-media-logo.png"
}
```

### Media Upload Request

```http
POST /wp-json/wp/v2/media HTTP/1.1
Authorization: Basic {credentials}
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="file"; filename="image.jpg"
Content-Type: image/jpeg

[binary data]
------WebKitFormBoundary--
```

## Categories & Tags (Taxonomies)

### Categories Endpoint

```
GET /wp-json/wp/v2/categories          # List all categories
GET /wp-json/wp/v2/categories/{id}     # Get single category
```

### Category Response Schema

```typescript
interface WpCategoryApiResponse {
  id: number;                    // Category term ID
  count: number;                 // Number of posts in category
  description: string;           // Category description
  link: string;                  // Category archive URL
  name: string;                  // Category name
  slug: string;                  // Category slug
  taxonomy: string;              // "category"
  parent: number;                // Parent category ID (0 for top-level)
  meta: any[];                   // Custom meta
  _links: object;                // HAL links
}
```

### Predefined Categories

```typescript
const WORDPRESS_CATEGORIES = {
  BLOG_POST: {
    id: 37,
    name: "Blog Post",
    slug: "blog-post"
  },
  PROJECT: {
    id: 38,
    name: "Project",
    slug: "project"
  },
  WORK_EXPERIENCE: {
    id: 41,
    name: "Work Experience",
    slug: "work-experience"
  },
  UNCATEGORIZED: {
    id: 1,
    name: "Uncategorized",
    slug: "uncategorized"
  }
};
```

### Tags Endpoint

```
GET /wp-json/wp/v2/tags               # List all tags
GET /wp-json/wp/v2/tags/{id}          # Get single tag
```

### Tag Response Schema

```typescript
interface WpTagApiResponse {
  id: number;                    // Tag term ID
  count: number;                 // Number of posts with tag
  description: string;           // Tag description
  link: string;                  // Tag archive URL
  name: string;                  // Tag name
  slug: string;                  // Tag slug
  taxonomy: string;              // "post_tag"
  meta: any[];                   // Custom meta
  _links: object;                // HAL links
}
```

## Next.js Client TypeScript Definitions

### Location
```
File: nextjs-client/app/lib/wordpress-types.ts
```

### Core Types

```typescript
// WordPress Post (Blog)
export interface WordPressPost {
  id: number;
  date_gmt: string;
  modified_gmt: string;
  slug: string;
  status: string;
  link: string;
  title: {
    rendered: string;
  };
  content: {
    rendered: string;
  };
  excerpt: {
    rendered: string;
  };
  featured_media: number;
  categories: number[];
  tags: number[];
}

// WordPress Project (Custom Post Type)
export interface WordPressProject extends WordPressPost {
  type: "project";
  meta: {
    _project_subtext?: string;
    _project_role?: string;
    _project_company?: string;
    _project_company_url?: string;
    _project_source_url?: string;
    _project_gallery?: string;
    _project_gallery_captions?: string;
    _project_thumbnail?: string;
    _project_date_type?: "single" | "range";
    _project_date_format?: "yyyy" | "mm/yyyy" | "dd/mm/yyyy";
    _project_date_start?: string;
    _project_date_end?: string;
  };
}

// WordPress Media
export interface WordPressMedia {
  id: number;
  date_gmt: string;
  slug: string;
  source_url: string;
  alt_text: string;
  media_type: string;
  mime_type: string;
  media_details: {
    width: number;
    height: number;
    file: string;
    filesize: number;
    sizes: {
      [key: string]: {
        file: string;
        width: number;
        height: number;
        mime_type: string;
        source_url: string;
      };
    };
  };
}

// Category
export interface WordPressCategory {
  id: number;
  count: number;
  description: string;
  link: string;
  name: string;
  slug: string;
  taxonomy: "category";
}

// Tag
export interface WordPressTag {
  id: number;
  count: number;
  description: string;
  link: string;
  name: string;
  slug: string;
  taxonomy: "post_tag";
}
```

## API Client Functions (Next.js)

### Location
```
File: nextjs-client/app/lib/server-lib.ts
```

### Function Signatures

```typescript
// Fetch all projects (filtered by PROJECT category ID: 38)
export async function fetchWpProjects(): Promise<WordPressProject[] | false>

// Fetch all projects (no category filter)
export async function fetchAllWpProjects(): Promise<WordPressProject[] | false>

// Fetch single project by ID or slug
export async function fetchWpProject(idOrSlug: string | number): Promise<WordPressProject | false>

// Fetch all blog posts (filtered by BLOG_POST category ID: 37)
export async function fetchWpPosts(): Promise<WordPressPost[] | false>

// Fetch single post by ID or slug
export async function fetchWpPost(idOrSlug: string | number): Promise<WordPressPost | false>

// Fetch media by ID
export async function fetchWpMediaById(id: number): Promise<WordPressMedia | false>

// Fetch all categories
export async function fetchWpAllCategories(): Promise<WordPressCategory[] | false>

// Fetch all tags
export async function fetchWpAllTags(): Promise<WordPressTag[] | false>
```

### Implementation Pattern

```typescript
const response = await fetch(`${WP_API_BASE}/projects`, {
  next: {
    revalidate: STANDARD_CACHE_TTL  // 3600 seconds (1 hour)
  },
  headers: {
    'Content-Type': 'application/json'
  }
});

if (!response.ok) {
  console.error('Failed to fetch projects:', response.statusText);
  return false;
}

const data = await response.json();
return data;
```

## Data Processing Utilities

### Gallery Parsing

```typescript
// Input: "123,456,789"
// Output: [123, 456, 789]
function parseGalleryIds(gallery: string): number[] {
  return gallery.split(',').map(id => parseInt(id.trim(), 10)).filter(id => !isNaN(id));
}

// Input: '{"123": "Mobile view", "456": "Desktop"}'
// Output: { 123: "Mobile view", 456: "Desktop" }
function parseGalleryCaptions(captions: string): Record<number, string> {
  try {
    const parsed = JSON.parse(captions);
    return Object.fromEntries(
      Object.entries(parsed).map(([k, v]) => [parseInt(k, 10), v as string])
    );
  } catch {
    return {};
  }
}
```

### Date Formatting

```typescript
function formatDate(dateString: string, format: "yyyy" | "mm/yyyy" | "dd/mm/yyyy"): string {
  const date = new Date(dateString);

  switch (format) {
    case "yyyy":
      return date.getFullYear().toString();
    case "mm/yyyy":
      return `${String(date.getMonth() + 1).padStart(2, '0')}/${date.getFullYear()}`;
    case "dd/mm/yyyy":
      return `${String(date.getDate()).padStart(2, '0')}/${String(date.getMonth() + 1).padStart(2, '0')}/${date.getFullYear()}`;
    default:
      return dateString;
  }
}

function formatDateRange(
  start: string,
  end: string | undefined,
  format: "yyyy" | "mm/yyyy" | "dd/mm/yyyy"
): string {
  const startFormatted = formatDate(start, format);
  const endFormatted = end ? formatDate(end, format) : "Present";
  return `${startFormatted} - ${endFormatted}`;
}
```

## Error Handling

### API Error Responses

```typescript
// Network error
{
  error: true,
  message: "Failed to fetch",
  status: 0
}

// Not found (404)
{
  code: "rest_post_invalid_id",
  message: "Invalid post ID.",
  data: {
    status: 404
  }
}

// Unauthorized (401)
{
  code: "rest_forbidden",
  message: "Sorry, you are not allowed to do that.",
  data: {
    status: 401
  }
}

// Validation error (400)
{
  code: "rest_invalid_param",
  message: "Invalid parameter(s): meta",
  data: {
    status: 400,
    params: {
      meta: "Invalid meta field value"
    }
  }
}
```

### Client-Side Error Handling

```typescript
try {
  const projects = await fetchWpProjects();
  if (projects === false) {
    // Handle API failure
    console.error('Failed to load projects');
    return null;
  }
  // Process projects
} catch (error) {
  console.error('Exception:', error);
  return null;
}
```

## Cache Strategy

### Next.js Server-Side Caching

```typescript
fetch(url, {
  next: {
    revalidate: 3600,  // 1 hour TTL
    tags: ['wordpress-projects']  // Cache tag for on-demand invalidation
  }
});
```

### Cache Invalidation

```typescript
// On-demand revalidation (requires Next.js API route)
import { revalidateTag } from 'next/cache';

export async function POST(request: Request) {
  revalidateTag('wordpress-projects');
  return Response.json({ revalidated: true, now: Date.now() });
}
```

## Rate Limiting

### WordPress REST API Limits
- No built-in rate limiting
- Server-side limits depend on hosting (cPanel)
- Recommended: 100 requests/minute for bulk operations

### Client Implementation
```typescript
async function bulkOperation(items: any[]) {
  for (const item of items) {
    await processItem(item);
    await delay(500);  // 500ms delay = 120 requests/minute
  }
}

function delay(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

## Testing Data Contracts

### Sample cURL Requests

```bash
# Get all projects
curl "https://staging.blog.shaganplaatjies.co.za/wp-json/wp/v2/projects?per_page=100"

# Get single project
curl "https://staging.blog.shaganplaatjies.co.za/wp-json/wp/v2/projects/186"

# Get media
curl "https://staging.blog.shaganplaatjies.co.za/wp-json/wp/v2/media/187"

# Create project (authenticated)
curl -X POST "https://staging.blog.shaganplaatjies.co.za/wp-json/wp/v2/projects" \
  -H "Authorization: Basic $(echo -n 'user:pass' | base64)" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Test Project",
    "status": "publish",
    "categories": [38],
    "meta": {
      "_project_role": "Developer"
    }'
```

## Contract Versioning

### Current API Version
- **WordPress REST API**: v2
- **Custom Fields Schema**: v1.0.0 (defined in plugin)
- **TypeScript Types**: Matches WordPress REST API v2 + custom fields

### Breaking Changes Protocol
1. Update plugin to expose new fields via REST API
2. Update TypeScript types in Next.js client
3. Update server-lib.ts functions if needed
4. Deploy plugin first (backward compatible)
5. Deploy Next.js client second (uses new fields)

### Backward Compatibility
- Optional custom fields (use `?` in TypeScript)
- Default values for missing fields
- Graceful degradation in UI components
