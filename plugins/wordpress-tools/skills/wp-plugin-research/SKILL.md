# wp-plugin-research

WordPress Plugin Research Agent - Research plugin APIs, documentation, and create implementation plans.

## Purpose

Research WordPress plugin APIs and documentation to enable proper integration with the wp-import/wp-export workflow.

## Workflow

1. **Identify the plugin**: Get plugin name, slug, and version
2. **Research documentation**: Find official docs, REST API endpoints, hooks
3. **Analyze data structures**: Understand how the plugin stores data
4. **Create implementation plan**: Document how to export/import plugin data
5. **Provide code examples**: Show REST API calls and data formats

## Research Sources

### Official Sources
- WordPress.org plugin page: `https://wordpress.org/plugins/{slug}/`
- Plugin's GitHub repository (if open source)
- Official plugin documentation site
- WordPress REST API handbook

### Data Discovery
- Plugin meta prefixes (check `wp_postmeta` table patterns)
- Custom post types registered by plugin
- Custom REST API endpoints
- Plugin options in `wp_options` table

## Common Plugin Patterns

### SEO Plugins

#### Rank Math
- Meta prefix: `rank_math_*`
- REST API: Standard WP REST with custom meta
- Key fields:
  ```json
  {
    "rank_math_title": "SEO Title | Site",
    "rank_math_description": "Meta description",
    "rank_math_focus_keyword": "keyword",
    "rank_math_robots": ["index", "follow"],
    "rank_math_canonical_url": "",
    "rank_math_og_title": "Social title",
    "rank_math_og_description": "Social description"
  }
  ```

#### Yoast SEO
- Meta prefix: `_yoast_wpseo_*`
- REST API: Adds `yoast_head` to standard endpoints
- Key fields:
  ```json
  {
    "_yoast_wpseo_title": "%%title%% %%page%% %%sep%% %%sitename%%",
    "_yoast_wpseo_metadesc": "Meta description",
    "_yoast_wpseo_focuskw": "focus keyword",
    "_yoast_wpseo_canonical": "",
    "_yoast_wpseo_opengraph-title": "",
    "_yoast_wpseo_opengraph-description": ""
  }
  ```

### Form Plugins

#### Contact Form 7
- REST endpoint: `/wp-json/contact-form-7/v1/contact-forms`
- Custom post type: `wpcf7_contact_form`
- Data structure:
  ```json
  {
    "id": 123,
    "slug": "contact-form-1",
    "title": "Contact Form",
    "locale": "en_US",
    "form": "[text* your-name]...",
    "mail": {
      "subject": "Contact from [your-name]",
      "sender": "[your-email]",
      "body": "...",
      "recipient": "admin@example.com"
    },
    "messages": {
      "mail_sent_ok": "Thank you!",
      "mail_sent_ng": "Error sending."
    }
  }
  ```

#### Gravity Forms
- REST endpoint: `/wp-json/gf/v2/forms`
- Requires API key authentication
- License required for REST API

#### WPForms
- No public REST API by default
- Data stored in custom post type: `wpforms`
- Form entries in custom table

### E-commerce

#### WooCommerce
- Meta prefix: `_wc_*`, `_product_*`
- REST endpoint: `/wp-json/wc/v3/`
- Requires consumer key/secret authentication
- Products, orders, customers have dedicated endpoints

### Page Builders

#### Elementor
- Meta key: `_elementor_data` (JSON)
- Meta key: `_elementor_edit_mode` = "builder"
- Complex nested JSON structure for widgets

#### Beaver Builder
- Meta key: `_fl_builder_data` (serialized PHP)
- Meta key: `_fl_builder_draft` for unpublished changes

### Custom Fields

#### ACF (Advanced Custom Fields)
- Meta prefix: varies by field group
- Field definitions in `acf-field-group` post type
- Values stored as regular post meta
- Relationship fields store post IDs

## Research Template

When researching a new plugin, document:

```markdown
## Plugin: {Plugin Name}

### Basic Info
- Slug: {slug}
- Version: {version}
- WordPress.org: https://wordpress.org/plugins/{slug}/
- Documentation: {url}

### Data Storage
- Meta prefix: `{prefix}_*`
- Custom post types: {list}
- Custom tables: {list}
- Options: {list}

### REST API
- Endpoint: `/wp-json/{namespace}/v{version}/`
- Authentication: {method}
- Key endpoints:
  - GET {endpoint}: {description}
  - POST {endpoint}: {description}

### Export Strategy
1. {step}
2. {step}

### Import Strategy
1. {step}
2. {step}

### Data Format
```json
{
  "example": "structure"
}
```

### Implementation Notes
- {note}
- {note}
```

## Example Research Task

**User**: "I need to export/import Gravity Forms data"

**Research Process**:
1. Check Gravity Forms REST API documentation
2. Identify authentication requirements (API keys)
3. Document form structure and entry format
4. Note license requirements
5. Create export/import plan

**Output**:
```markdown
## Plugin: Gravity Forms

### REST API
- Endpoint: `/wp-json/gf/v2/`
- Authentication: API keys (Settings > REST API)
- License: Required for API access

### Key Endpoints
- GET `/forms`: List all forms
- GET `/forms/{id}`: Get form definition
- GET `/forms/{id}/entries`: Get form submissions

### Export Strategy
1. Configure API keys in Gravity Forms settings
2. Add `GF_API_KEY` and `GF_API_SECRET` to .env
3. Fetch forms via REST API
4. Export form definitions to `gravity-forms.json`

### Import Strategy
1. Forms cannot be created via REST API (license limitation)
2. Manual import via WordPress admin required
3. Can update existing forms via PUT `/forms/{id}`
```

## Tools to Use

- `WebSearch`: Find plugin documentation
- `WebFetch`: Read documentation pages
- `Grep`: Search existing codebase for plugin patterns
- `Read`: Examine exported plugin data files
