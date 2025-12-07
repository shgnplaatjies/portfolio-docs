# Code Conventions & Standards

**Purpose**: Comprehensive code style guide, patterns, and conventions for maintaining consistency across the portfolio application.

## Repository-Specific Conventions

### Next.js Client (`nextjs-client/`)

#### TypeScript Configuration
```json
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "paths": {
      "@/*": ["./*"]
    }
  }
}
```

**Requirements**:
- Strict mode enabled
- Explicit return types for functions
- No implicit `any`
- Path aliases using `@/` for root imports

#### File Naming

| Type | Convention | Example |
|------|-----------|---------|
| React Components | PascalCase | `ProjectCard.tsx`, `ExperienceCard.tsx` |
| Pages (App Router) | lowercase | `page.tsx`, `layout.tsx`, `[slug]/page.tsx` |
| Utilities | kebab-case | `server-lib.ts`, `wordpress-types.ts` |
| Hooks | camelCase with `use` prefix | `useTheme.ts`, `useMediaQuery.ts` |
| API Routes | lowercase | `route.ts` (in `api/contact/`) |
| Directories | kebab-case | `api/contact`, `lib/utils` |

#### Component Patterns

**Server Components** (default in App Router):
```typescript
// NO "use client" directive
// CAN use async/await
// CAN fetch data directly
// CANNOT use React hooks
// CANNOT use browser APIs

export default async function ProjectsSection() {
  const projects = await fetchWpProjects();  // Server-side data fetch

  return (
    <section>
      {projects && projects.map(project => (
        <ProjectCard key={project.id} project={project} />
      ))}
    </section>
  );
}
```

**Client Components**:
```typescript
"use client";  // REQUIRED at top of file

import { useState, useEffect } from 'react';

// CAN use React hooks
// CAN use browser APIs
// CANNOT use async/await in component function
// SHOULD minimize client-side JS bundle

interface ContactFormProps {
  initialEmail?: string;
}

export default function ContactForm({ initialEmail }: ContactFormProps) {
  const [email, setEmail] = useState(initialEmail || '');

  // Component logic...
}
```

#### Import Organization

```typescript
// 1. External libraries
import React from 'react';
import { motion } from 'framer-motion';

// 2. Next.js imports
import Image from 'next/image';
import Link from 'next/link';

// 3. Internal utilities (@ alias)
import { fetchWpProjects } from '@/app/lib/server-lib';
import { WordPressProject } from '@/app/lib/wordpress-types';
import { WORDPRESS_CATEGORIES } from '@/app/lib/constants';

// 4. Components
import ProjectCard from '@/app/components/ProjectCard';
import Header from '@/app/components/Header';

// 5. Styles (if any)
import styles from './styles.module.css';

// 6. Types (if separate file)
import type { ProjectCardProps } from './types';
```

#### Function Component Structure

```typescript
// 1. Type/Interface definitions
interface ProjectCardProps {
  project: WordPressProject;
  showExcerpt?: boolean;
}

// 2. Component function
export default function ProjectCard({ project, showExcerpt = true }: ProjectCardProps) {
  // 3. Hooks (in order: state, effects, context, custom hooks)
  const [isExpanded, setIsExpanded] = useState(false);
  const theme = useTheme();

  // 4. Derived values
  const formattedDate = formatDate(project.meta._project_date_start, 'mm/yyyy');

  // 5. Event handlers
  const handleExpand = () => {
    setIsExpanded(!isExpanded);
  };

  // 6. Early returns
  if (!project) return null;

  // 7. Render
  return (
    <div className="project-card">
      {/* JSX */}
    </div>
  );
}
```

#### Tailwind CSS Class Organization

```tsx
<div
  className={`
    // Layout
    flex flex-col

    // Spacing
    p-4 md:p-6 gap-4

    // Sizing
    w-full max-w-2xl

    // Typography
    text-gray-900 text-base font-medium

    // Background & Borders
    bg-white border border-gray-200 rounded-lg

    // Effects
    shadow-md hover:shadow-lg

    // Transitions
    transition-all duration-300

    // Dark mode
    dark:bg-gray-800 dark:text-white dark:border-gray-700
  `}
>
```

**Class Ordering**:
1. Display & Layout (flex, grid, block)
2. Positioning (relative, absolute, z-index)
3. Spacing (margin, padding, gap)
4. Sizing (width, height, max-width)
5. Typography (font, text-color, text-size)
6. Background & Borders
7. Effects (shadow, opacity)
8. Transitions & Animations
9. Responsive modifiers (md:, lg:)
10. State modifiers (hover:, focus:, active:)
11. Dark mode (dark:)

#### Data Fetching Patterns

**Server-Side Fetch with Cache**:
```typescript
export async function fetchWpProjects(): Promise<WordPressProject[] | false> {
  try {
    const response = await fetch(`${WP_API_BASE}/projects?categories=${WORDPRESS_CATEGORIES.PROJECT.id}&per_page=100`, {
      next: {
        revalidate: STANDARD_CACHE_TTL,  // 3600 seconds
        tags: ['wordpress-projects']
      },
      headers: {
        'Content-Type': 'application/json'
      }
    });

    if (!response.ok) {
      console.error('Failed to fetch projects:', response.statusText);
      return false;
    }

    return await response.json();
  } catch (error) {
    console.error('Exception fetching projects:', error);
    return false;
  }
}
```

**Parallel Data Fetching**:
```typescript
// GOOD: Use Promise.all for independent requests
const [projects, categories, tags] = await Promise.all([
  fetchWpProjects(),
  fetchWpAllCategories(),
  fetchWpAllTags()
]);

// BAD: Sequential fetching (slower)
const projects = await fetchWpProjects();
const categories = await fetchWpAllCategories();
const tags = await fetchWpAllTags();
```

#### Error Handling

```typescript
// API Functions: Return false on error
export async function fetchWpProject(slug: string): Promise<WordPressProject | false> {
  try {
    const response = await fetch(url);
    if (!response.ok) return false;
    return await response.json();
  } catch {
    return false;
  }
}

// Components: Handle false returns
const project = await fetchWpProject(slug);
if (!project) {
  return <NotFound />;
}
```

#### Security Practices

```typescript
// Output escaping in JSX (automatic)
<div>{project.title.rendered}</div>  // Auto-escaped

// Dangerous HTML (WordPress content)
<div dangerouslySetInnerHTML={{ __html: project.content.rendered }} />
// OR use custom sanitization:
<PostContent content={project.content.rendered} />  // Custom component with regex transforms

// Environment variables (server-only)
import "server-only";  // At top of server-lib.ts
const WP_DOMAIN = process.env.WP_DOMAIN;  // Only accessible server-side
```

---

### WordPress Plugin (`wordpress-plugin/`)

#### PHP Standards

**Version**: PHP 7.4+ (WordPress Coding Standards + PSR-2 inspired)

#### File Naming

| Type | Convention | Example |
|------|-----------|---------|
| Classes | `class-{name}.php` | `class-post-type-project.php` |
| Functions | `functions-{name}.php` | `functions-helpers.php` |
| Main Plugin | `{plugin-name}.php` | `portfolio-plugin.php` |

#### Class Naming

```php
// Prefix with plugin name to avoid conflicts
class Portfolio_Plugin_Post_Type_Project {
    // Snake_case for methods
    public function register_post_type() {
        // Implementation
    }

    private function get_post_type_args() {
        // Implementation
    }
}
```

#### Function Naming

```php
// Prefix all global functions
function portfolio_plugin_init() {
    // Implementation
}

function portfolio_plugin_get_meta($post_id, $key) {
    // Implementation
}
```

#### Hook Registration

```php
// Use closures for simple hooks
add_action('init', function() {
    // Simple initialization
});

// Use class methods for complex hooks
add_action('add_meta_boxes_project', array($meta_boxes, 'add_meta_boxes'));
add_action('save_post_project', array($meta_boxes, 'save_meta_boxes'), 10, 2);
```

#### Security Practices

**Nonce Verification**:
```php
// Create nonce in form
wp_nonce_field('portfolio_project_nonce', 'portfolio_project_nonce');

// Verify nonce on submit
if (!isset($_POST['portfolio_project_nonce']) ||
    !wp_verify_nonce($_POST['portfolio_project_nonce'], 'portfolio_project_nonce')) {
    return;
}
```

**Capability Checks**:
```php
// Before saving
if (!current_user_can('edit_post', $post_id)) {
    return;
}

// In meta registration
'auth_callback' => function() {
    return current_user_can('edit_posts');
}
```

**Input Sanitization**:
```php
// Text fields
$role = sanitize_text_field($_POST['project_role']);

// URLs
$url = esc_url_raw($_POST['company_url']);

// Numbers
$media_id = absint($_POST['thumbnail_id']);

// JSON
$captions = wp_kses_post($_POST['gallery_captions']);
if (!json_decode($captions)) {
    $captions = '{}';  // Fallback to empty object
}
```

**Output Escaping**:
```php
// HTML attributes
<input type="text" value="<?php echo esc_attr($value); ?>" />

// URLs
<a href="<?php echo esc_url($url); ?>">Link</a>

// HTML content
<p><?php echo esc_html($text); ?></p>

// Allow HTML (only for trusted content)
<div><?php echo wp_kses_post($content); ?></div>
```

#### Meta Box Pattern

```php
class Portfolio_Plugin_Meta_Boxes {
    public function add_meta_boxes($post) {
        add_meta_box(
            'project_details',              // ID
            __('Project Details', 'portfolio-plugin'),  // Title
            array($this, 'render_details_meta_box'),    // Callback
            'project',                      // Post type
            'normal',                       // Context
            'high'                          // Priority
        );
    }

    public function render_details_meta_box($post) {
        // Nonce field
        wp_nonce_field('portfolio_project_nonce', 'portfolio_project_nonce');

        // Get current value
        $role = get_post_meta($post->ID, '_project_role', true);

        // Render field
        ?>
        <label for="project_role"><?php _e('Role:', 'portfolio-plugin'); ?></label>
        <input
            type="text"
            id="project_role"
            name="project_role"
            value="<?php echo esc_attr($role); ?>"
            class="regular-text"
        />
        <?php
    }

    public function save_meta_boxes($post_id, $post) {
        // Autosave check
        if (defined('DOING_AUTOSAVE') && DOING_AUTOSAVE) {
            return;
        }

        // Nonce verification
        if (!isset($_POST['portfolio_project_nonce']) ||
            !wp_verify_nonce($_POST['portfolio_project_nonce'], 'portfolio_project_nonce')) {
            return;
        }

        // Capability check
        if (!current_user_can('edit_post', $post_id)) {
            return;
        }

        // Save meta
        if (isset($_POST['project_role'])) {
            $role = sanitize_text_field($_POST['project_role']);
            update_post_meta($post_id, '_project_role', $role);
        }
    }
}
```

#### REST API Meta Registration

```php
register_meta('post', '_project_role', array(
    'type'         => 'string',
    'description'  => 'User role for the project',
    'single'       => true,
    'show_in_rest' => true,
    'auth_callback' => function() {
        return current_user_can('edit_posts');
    }
));
```

#### JavaScript (Admin)

```javascript
// Use jQuery (provided by WordPress)
(function($) {
    'use strict';

    $(document).ready(function() {
        // Media gallery handling
        $('#add-gallery-images').on('click', function(e) {
            e.preventDefault();

            const frame = wp.media({
                title: 'Select Gallery Images',
                multiple: true,
                library: { type: 'image' }
            });

            frame.on('select', function() {
                const selection = frame.state().get('selection');
                // Process selection
            });

            frame.open();
        });
    });

})(jQuery);
```

---

### WordPress Theme (`wordpress-theme/`)

#### PHP Standards

**Version**: PHP 8.0+ (WordPress Coding Standards)

#### File Structure

```
functions.php                 # Theme initialization ONLY
app/setup.php                # Theme setup and features
app/blocks.php               # Custom block registration
app/acf-helpers.php          # ACF utility functions
templates/{name}.blade.php   # Blade templates
```

#### Theme Initialization Pattern

```php
// functions.php - Keep minimal
<?php
define('SHAGANPLAATJIES_THEME_PATH', get_template_directory());
define('SHAGANPLAATJIES_THEME_URL', get_template_directory_uri());

// Load helpers
require_once SHAGANPLAATJIES_THEME_PATH . '/app/setup.php';
require_once SHAGANPLAATJIES_THEME_PATH . '/app/blocks.php';
require_once SHAGANPLAATJIES_THEME_PATH . '/app/acf-helpers.php';

// Initialize
add_action('after_setup_theme', 'shaganplaatjies_setup');
```

#### Blade Templates

```blade
{{-- templates/page.blade.php --}}

@extends('templates.app')

@section('content')
    <article class="page">
        <header>
            <h1 class="page-title">{!! get_the_title() !!}</h1>
        </header>

        <div class="page-content">
            {!! get_the_content() !!}
        </div>
    </article>
@endsection
```

#### Asset Enqueuing

```php
function shaganplaatjies_enqueue_assets() {
    // CSS
    wp_enqueue_style(
        'shaganplaatjies-style',
        get_template_directory_uri() . '/dist/css/app.css',
        array(),
        filemtime(get_template_directory() . '/dist/css/app.css')  // Cache busting
    );

    // JavaScript
    wp_enqueue_script(
        'shaganplaatjies-script',
        get_template_directory_uri() . '/dist/js/app.js',
        array(),  // Dependencies
        filemtime(get_template_directory() . '/dist/js/app.js'),
        true  // In footer
    );

    // Localization
    wp_localize_script('shaganplaatjies-script', 'shaganplaatjiesTheme', array(
        'ajaxUrl' => admin_url('admin-ajax.php'),
        'nonce'   => wp_create_nonce('shaganplaatjies_nonce'),
    ));
}
add_action('wp_enqueue_scripts', 'shaganplaatjies_enqueue_assets');
```

#### JavaScript (Frontend)

**ESLint Configuration**: Airbnb base + Prettier

```javascript
// resources/js/app.js

/**
 * Initialize theme functionality
 */
function initializeTheme() {
  initializeMobileMenu();
  initializeScrollEffects();
  initializeAccessibility();
  initializeFormBehavior();
}

/**
 * Initialize mobile menu toggle
 */
function initializeMobileMenu() {
  const menuToggle = document.querySelector('[data-menu-toggle]');
  const mobileMenu = document.querySelector('[data-mobile-menu]');

  if (!menuToggle || !mobileMenu) return;

  menuToggle.addEventListener('click', () => {
    mobileMenu.classList.toggle('hidden');
  });

  // Close on outside click
  document.addEventListener('click', (e) => {
    if (!menuToggle.contains(e.target) && !mobileMenu.contains(e.target)) {
      mobileMenu.classList.add('hidden');
    }
  });

  // Close on Escape
  document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape' && !mobileMenu.classList.contains('hidden')) {
      mobileMenu.classList.add('hidden');
    }
  });
}

// Initialize on DOM ready
if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', initializeTheme);
} else {
  initializeTheme();
}
```

#### Tailwind Configuration

```javascript
// tailwind.config.js
module.exports = {
  content: [
    './templates/**/*.blade.php',
    './resources/**/*.js',
    './app/**/*.php',
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#f0f9ff',
          // ... color scale
          900: '#0c4a6e',
        },
      },
      fontFamily: {
        sans: ['-apple-system', 'BlinkMacSystemFont', 'Segoe UI', 'Roboto'],
      },
    },
  },
  plugins: [
    require('@tailwindcss/typography'),
    require('@tailwindcss/forms'),
  ],
};
```

#### Webpack Mix Configuration

```javascript
// webpack.mix.js
const mix = require('laravel-mix');

mix
  .setPublicPath('dist')
  .js('resources/js/app.js', 'js')
  .postCss('resources/css/app.css', 'css', [
    require('postcss-import'),
    require('postcss-nested'),
    require('tailwindcss'),
    require('autoprefixer'),
  ])
  .webpackConfig({
    resolve: {
      alias: {
        '@': path.resolve(__dirname, 'resources/js'),
      },
    },
  })
  .sourceMaps(false, 'source-map')  // Production: no source maps
  .version();  // Production: version assets
```

---

## Common Patterns Across Repositories

### Environment Variables

**Naming**:
```bash
# SCREAMING_SNAKE_CASE for all env vars
WP_DOMAIN=example.com
APP_PORT=3000
RESEND_API_KEY=re_xyz123
```

**Loading**:
```typescript
// Next.js: Use process.env directly (auto-loaded)
const domain = process.env.WP_DOMAIN;

// PHP: Use vlucas/phpdotenv (Bedrock)
$dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
$dotenv->load();
$domain = $_ENV['WP_DOMAIN'];
```

### Git Commit Messages

**Format**:
```
<type>(<scope>): <subject>

<body>

🤖 Generated with Claude Code
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, no logic change)
- `refactor`: Code refactoring
- `test`: Adding/updating tests
- `chore`: Build process, tooling, dependencies

**Examples**:
```
feat(nextjs): add project gallery lightbox component

Implemented GalleryImageDialog with Radix UI Dialog
Supports keyboard navigation and image captions
Lazy loads images for performance

🤖 Generated with Claude Code
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

```
fix(plugin): sanitize gallery caption JSON input

Added JSON validation before saving captions
Fallback to empty object on invalid JSON
Prevents PHP warnings on malformed input

🤖 Generated with Claude Code
Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

### Error Messages

**User-Facing**:
```typescript
// Clear, actionable
"Failed to load projects. Please try again later."

// NOT technical jargon
"API call to /wp-json/wp/v2/projects returned 500"
```

**Developer-Facing** (console logs):
```typescript
console.error('Failed to fetch projects:', response.statusText);
console.error('Exception fetching projects:', error);
```

### Comments

**When to Comment**:
- Complex business logic
- Non-obvious workarounds
- Security-critical sections
- Public APIs

**When NOT to Comment**:
- Self-explanatory code
- Redundant descriptions
- Outdated comments (delete instead of keeping)

**Good Comments**:
```typescript
// Regex transforms WordPress HTML to add Tailwind classes
// WordPress doesn't support class injection, so we post-process
const transformedHTML = content
  .replace(/<h1>/g, '<h1 class="text-4xl font-bold">')
  .replace(/<p>/g, '<p class="mb-4 text-gray-700">');
```

**Bad Comments**:
```typescript
// Set the email variable to the value from the form
const email = formData.get('email');  // Don't do this
```

### Testing

**Next.js (Jest)**:
```typescript
// __tests__/utils.test.ts
import { formatDate } from '../utils';

describe('formatDate', () => {
  it('formats yyyy correctly', () => {
    expect(formatDate('2024-06-15', 'yyyy')).toBe('2024');
  });

  it('formats mm/yyyy correctly', () => {
    expect(formatDate('2024-06-15', 'mm/yyyy')).toBe('06/2024');
  });
});
```

**WordPress (PHPUnit)**:
```php
// tests/test-meta-boxes.php
class Test_Meta_Boxes extends WP_UnitTestCase {
    public function test_save_project_role() {
        $post_id = $this->factory->post->create(['post_type' => 'project']);
        update_post_meta($post_id, '_project_role', 'Developer');

        $role = get_post_meta($post_id, '_project_role', true);
        $this->assertEquals('Developer', $role);
    }
}
```

---

## Code Review Checklist

### Before Committing

- [ ] Code follows repository conventions
- [ ] TypeScript/ESLint/PHPCS passes
- [ ] No console.log (use console.error/warn only)
- [ ] Environment variables not hardcoded
- [ ] Security: Input sanitized, output escaped
- [ ] Error handling implemented
- [ ] Comments added for complex logic
- [ ] Tests added/updated (if applicable)
- [ ] Build succeeds (`npm run build` or `npm run production`)
- [ ] No sensitive data committed (.env files gitignored)

### Before Deploying

- [ ] Tested on staging environment
- [ ] Database migrations run (if applicable)
- [ ] Environment variables set on server
- [ ] Cache cleared after deployment
- [ ] SSL certificates valid
- [ ] GitHub Actions workflow succeeded
- [ ] Rollback plan prepared

---

## Performance Guidelines

### Next.js

**DO**:
- Use Server Components by default
- Cache API responses with `revalidate`
- Use `next/image` for image optimization
- Lazy load below-the-fold components
- Minimize client-side JavaScript

**DON'T**:
- Use Client Components unnecessarily
- Fetch data in client components (use Server Components)
- Include large libraries in client bundle
- Use inline styles (use Tailwind)

### WordPress

**DO**:
- Cache WordPress queries (Transient API)
- Enqueue scripts in footer (`true` parameter)
- Minify and concatenate assets
- Use production builds (Webpack Mix)
- Lazy load images

**DON'T**:
- Query database in loops
- Enqueue multiple small scripts (combine)
- Load full jQuery if not needed
- Use blocking scripts in `<head>`

---

## Accessibility (a11y) Requirements

### Semantic HTML
```html
<!-- GOOD -->
<nav aria-label="Main navigation">
  <ul>
    <li><a href="/">Home</a></li>
  </ul>
</nav>

<main>
  <article>
    <h1>Title</h1>
  </article>
</main>

<!-- BAD -->
<div class="nav">
  <div class="link">Home</div>
</div>
```

### ARIA Labels
```tsx
// Interactive elements without text
<button aria-label="Close menu" onClick={handleClose}>
  <XIcon />
</button>

// Form inputs
<input
  type="email"
  id="email"
  aria-describedby="email-help"
  aria-required="true"
/>
<span id="email-help">We'll never share your email</span>
```

### Keyboard Navigation
```typescript
// Support Enter/Space for custom interactive elements
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      handleClick();
    }
  }}
>
  Custom Button
</div>
```

### Focus Management
```css
/* Always provide visible focus indicators */
.button:focus-visible {
  outline: 2px solid theme('colors.primary.500');
  outline-offset: 2px;
}

/* Don't remove outlines globally */
/* NEVER: *:focus { outline: none; } */
```

---

## Documentation Requirements

### Function Documentation

**TypeScript (JSDoc)**:
```typescript
/**
 * Fetches all WordPress projects from the REST API
 * Filters by PROJECT category (ID: 38) and caches for 1 hour
 *
 * @returns Array of projects or false on error
 * @example
 * ```typescript
 * const projects = await fetchWpProjects();
 * if (projects) {
 *   console.log(`Found ${projects.length} projects`);
 * }
 * ```
 */
export async function fetchWpProjects(): Promise<WordPressProject[] | false> {
  // Implementation
}
```

**PHP (PHPDoc)**:
```php
/**
 * Register the project custom post type
 *
 * Creates a public post type with REST API support
 * and custom taxonomy support for categories and tags
 *
 * @since 1.0.0
 * @return void
 */
public function register_post_type() {
    // Implementation
}
```

### README Requirements

Each repository should have:
- **Purpose**: One sentence description
- **Tech Stack**: Key technologies
- **Setup**: Installation instructions
- **Development**: How to run locally
- **Deployment**: How to deploy
- **Environment Variables**: List and description
- **Scripts**: Available npm/composer commands

---

This comprehensive code conventions guide ensures consistency and maintainability across the entire portfolio application.
