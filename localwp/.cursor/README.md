# WordPress Cursor Rules - Usage Guide

Ready-to-use cursor rules organized in the modern `.cursor/rules/*.mdc` format. These rules help Cursor AI understand WordPress development best practices and apply them intelligently based on the files you're working on.

## Quick Start

### 1. Copy the `.cursor` Folder to Your Project

Navigate to your WordPress project root and copy the entire `.cursor` folder:

```bash
# From your WordPress project root
cp -r /path/to/agentic-rules/localwp/.cursor .
```

That's it! Cursor will automatically load and apply the rules based on the files you're editing.

### 2. Choose Your Rules

The `.cursor/rules/` directory contains three subdirectories:

- **`plugin/`** - Rules for WordPress plugin development
- **`theme/`** - Rules for WordPress theme development  
- **`woocommerce/`** - Rules for WooCommerce customization

You can keep all three sets of rules, or remove the ones you don't need.

## How It Works

### MDC File Structure

Each rule file (`.mdc`) contains:

1. **Frontmatter** (YAML metadata)
2. **Rule content** (Markdown)

Example:

```yaml
---
description: Core WordPress plugin development principles
globs: ["wp-content/plugins/**/*.php"]
alwaysApply: true
---

# Core Plugin Principles

- Single Responsibility: Each plugin should have one clear purpose
- WordPress APIs First: Use built-in APIs over custom solutions
...
```

### Frontmatter Options

- **`description`**: Brief explanation of what the rule does
- **`globs`**: File patterns that trigger this rule (uses glob syntax)
- **`alwaysApply`**: If `true`, rule applies to all files; if `false`, only applies when glob pattern matches

## Rule Categories

### Plugin Rules (7 files)

| File | Always Apply | Description |
|------|-------------|-------------|
| `001-core-principles.mdc` | ✅ Yes | Fundamental plugin development principles |
| `002-security.mdc` | ✅ Yes | Security requirements (sanitize, escape, nonces) |
| `003-hooks-filters.mdc` | ✅ Yes | WordPress hooks and filters best practices |
| `100-activation-lifecycle.mdc` | 📍 Context | Activation, deactivation, and uninstall patterns |
| `101-admin-interface.mdc` | 📍 Context | Admin pages and Settings API |
| `102-ajax-handlers.mdc` | 📍 Context | AJAX request handling and security |
| `103-rest-api.mdc` | 📍 Context | REST API endpoints and permissions |

**Context-aware rules** apply only when you're editing files that match their glob patterns (e.g., admin rules apply to `admin/*.php` files).

### Theme Rules (6 files)

| File | Always Apply | Description |
|------|-------------|-------------|
| `001-core-principles.mdc` | ✅ Yes | Core theme development principles |
| `002-security.mdc` | ✅ Yes | Theme security requirements |
| `003-template-hierarchy.mdc` | ✅ Yes | Template hierarchy and WordPress template tags |
| `100-functions-setup.mdc` | 📍 Context | Theme setup in functions.php |
| `101-template-files.mdc` | 📍 Context | Template file patterns and best practices |
| `102-block-themes.mdc` | 📍 Context | Block themes and theme.json |

### WooCommerce Rules (7 files)

| File | Always Apply | Description |
|------|-------------|-------------|
| `001-core-principles.mdc` | ✅ Yes | Core WooCommerce development principles |
| `002-security.mdc` | ✅ Yes | WooCommerce security and payment data handling |
| `003-hooks-patterns.mdc` | ✅ Yes | Essential WooCommerce hooks and filters |
| `100-products.mdc` | 📍 Context | Product operations and queries |
| `101-cart-checkout.mdc` | 📍 Context | Cart and checkout customization |
| `102-orders.mdc` | 📍 Context | Order management and customer data |
| `103-payment-gateways.mdc` | 📍 Context | Custom payment gateways and webhooks |

## Understanding Glob Patterns

Glob patterns determine when context-aware rules apply:

- `**` - Matches any directory depth
- `*` - Matches any characters in filename
- `{a,b}` - Matches either a or b
- Examples:
  - `wp-content/plugins/**/*.php` - All PHP files in any plugin
  - `wp-content/themes/*/functions.php` - functions.php in any theme
  - `**/*{cart,checkout}*.php` - Files with "cart" or "checkout" in name

## Customization

### Option 1: Remove Unwanted Rules

If you're only building a plugin, you can delete the theme and woocommerce folders:

```bash
rm -rf .cursor/rules/theme
rm -rf .cursor/rules/woocommerce
```

### Option 2: Modify Existing Rules

Edit any `.mdc` file to adjust rules for your project needs:

```bash
# Example: Make a rule less strict or add project-specific patterns
nano .cursor/rules/plugin/002-security.mdc
```

### Option 3: Add Custom Rules

Create new `.mdc` files following the numbering convention:

- `001-099`: Core/always-applied rules
- `100-199`: Context-specific rules
- `200-299`: Advanced/specialized rules

Example custom rule:

```yaml
---
description: Custom API integration patterns for MyPlugin
globs: ["wp-content/plugins/my-plugin/includes/api/**/*.php"]
alwaysApply: false
---

# MyPlugin API Rules

- Always use the MyPlugin_API wrapper class
- Cache API responses for 1 hour minimum
...
```

## Best Practices

### Do's ✅

- Keep rules concise (under 20 lines of content)
- Use specific glob patterns for context-aware rules
- Set `alwaysApply: true` only for critical rules (security, core principles)
- Document any custom rules you add
- Update rules as you learn new patterns

### Don'ts ❌

- Don't make every rule `alwaysApply: true` (performance impact)
- Don't duplicate rules across different files
- Don't make rules too verbose (AI will lose focus)
- Don't include project-specific code in rules (keep them general)

## Troubleshooting

### Rules Not Being Applied?

1. Check that `.cursor` folder is in your project root
2. Verify file paths match glob patterns in frontmatter
3. Try restarting Cursor IDE
4. Check Cursor settings for rules configuration

### Too Many Rules Firing?

- Review `alwaysApply` settings - most rules should be `false`
- Make glob patterns more specific
- Remove unused rule categories (theme/plugin/woo)

### Rules Conflicting?

- Lower-numbered files (001) have higher priority
- Review glob patterns for overlaps
- Consolidate similar rules into one file

## Migration from Legacy `.cursorrules`

If you have existing `.cursorrules` files:

1. **Backup**: Copy your old `.cursorrules` to `.cursorrules.backup`
2. **Copy**: Install the new `.cursor/` folder structure
3. **Customize**: Add any custom rules from your old file to new `.mdc` files
4. **Test**: Verify Cursor applies rules correctly
5. **Remove**: Delete old `.cursorrules` file when satisfied

## Examples

### Example 1: Plugin Project Setup

```bash
# In your plugin root directory
cp -r /path/to/agentic-rules/localwp/.cursor .

# Remove theme and WooCommerce rules (not needed for plugin)
rm -rf .cursor/rules/theme
rm -rf .cursor/rules/woocommerce

# Start coding - rules apply automatically!
```

### Example 2: WooCommerce Theme Project

```bash
# In your theme root directory
cp -r /path/to/agentic-rules/localwp/.cursor .

# Remove plugin rules (not needed for theme)
rm -rf .cursor/rules/plugin

# Keep both theme and woocommerce rules
# Start coding!
```

### Example 3: Full WordPress Site Development

```bash
# In your WordPress root directory
cp -r /path/to/agentic-rules/localwp/.cursor .

# Keep all rules - they'll apply based on file context
# Plugin files get plugin rules, theme files get theme rules, etc.
```

## Resources

- [Cursor Rules Documentation](https://docs.cursor.com/en/context)
- [WordPress Coding Standards](https://developer.wordpress.org/coding-standards/)
- [WordPress Developer Resources](https://developer.wordpress.org/)
- [WooCommerce Documentation](https://woocommerce.com/documentation/)

## Support

These rules are maintained as part of the [agentic-rules](https://github.com/yourusername/agentic-rules) repository. Issues and contributions welcome!

---

**Last Updated**: November 2025  
**Format Version**: Modern `.cursor/rules/*.mdc` (Cursor IDE 2025+)






