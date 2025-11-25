# LocalWP Cursor Rules

A comprehensive collection of cursor rules and guidelines for WordPress development using Local by Flywheel. These rules help you leverage Cursor AI to build WordPress websites faster and more efficiently while following best practices.

## Overview

This repository provides modern, context-aware cursor rules for WordPress development. Rules are organized using the `.cursor/rules/*.mdc` format with smart glob patterns that automatically apply rules based on the files you're editing.

## Quick Start

**Ready-to-Use Cursor Rules:**

- **[Plugin Development](.cursor/rules/plugin/)** - 7 focused rules for WordPress plugin development
- **[Theme Development](.cursor/rules/theme/)** - 6 focused rules for WordPress theme development
- **[WooCommerce](.cursor/rules/woocommerce/)** - 7 focused rules for WooCommerce customization

**Documentation:**

- **[Usage Guide](.cursor/README.md)** - Comprehensive guide with usage instructions, customization examples, and troubleshooting

## How to Use These Rules

### Method 1: Copy Modern .cursor Folder (Recommended) ⭐

Copy the entire `.cursor` folder to your WordPress project for automatic, context-aware rule application:

```bash
# From your WordPress project root
cp -r /path/to/agentic-rules/localwp/.cursor .
```

**Benefits:**
- Rules apply automatically based on the files you're editing
- Organized by context (plugin, theme, WooCommerce)
- Uses glob patterns for intelligent rule matching
- Better performance (only relevant rules load)
- Easy to customize and maintain

See the **[Usage Guide](.cursor/README.md)** for detailed usage instructions.

### Method 2: Symlink .cursor Folder (Advanced)

Create a symlink for automatic updates when this repository changes:

```bash
# From your WordPress project root
ln -s /path/to/agentic-rules/localwp/.cursor .cursor
```

### Method 3: Custom Selection

Remove rule categories you don't need:

```bash
# Example: Keep only plugin rules, remove theme and WooCommerce
cp -r /path/to/agentic-rules/localwp/.cursor .
rm -rf .cursor/rules/theme
rm -rf .cursor/rules/woocommerce
```

## Working with Cursor AI

### Effective Prompting

When working with Cursor AI on WordPress projects, reference these rules by:

1. **Context Setting**: Start by telling Cursor what type of WordPress development you're doing (theme, plugin, WooCommerce, etc.)
2. **Standards Reference**: Mention you want to follow WordPress coding standards and best practices
3. **Specific Patterns**: Reference specific patterns from these docs (e.g., "Use the WordPress template hierarchy")
4. **Security First**: Always emphasize security (sanitization, escaping, nonces)

### Example Prompts

```
"Create a custom WordPress theme following WordPress coding standards. 
Use proper template hierarchy and enqueue assets correctly."

"Build a WooCommerce product filter using hooks and filters. 
Follow WooCommerce best practices and ensure proper sanitization."

"Create a custom Gutenberg block with block.json registration. 
Include InspectorControls and follow WordPress block development guidelines."
```

## Project Structure

```
localwp/
├── README.md                 # This file
└── .cursor/                  # Modern cursor rules (copy to your project)
    ├── README.md             # Detailed usage guide
    └── rules/
        ├── plugin/           # Plugin development rules (7 files)
        ├── theme/            # Theme development rules (6 files)
        └── woocommerce/      # WooCommerce rules (7 files)
```

### Understanding the .cursor/rules Structure

The `.cursor/rules/` directory uses the modern **MDC (Markdown with Metadata)** format:

- **Each `.mdc` file** contains YAML frontmatter with metadata
- **Glob patterns** determine when rules apply to specific files
- **alwaysApply** flag controls global vs context-aware rules
- **Organized by category**: plugin, theme, woocommerce

Example `.mdc` file structure:

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

## Best Practices Summary

### Security
- Always sanitize input data
- Always escape output data
- Use nonces for form submissions
- Check user capabilities
- Validate and verify all data

### Performance
- Use transients for expensive operations
- Optimize database queries
- Enqueue assets properly (minified, combined)
- Use object caching when available
- Lazy load images and content

### Code Quality
- Follow WordPress Coding Standards (WPCS)
- Use proper naming conventions
- Comment your code adequately
- Keep functions small and focused
- Use WordPress core functions when available

### Development Workflow
- Use Local by Flywheel for local development
- Version control with Git
- Test on multiple PHP versions
- Use debugging tools (Query Monitor, Debug Bar)
- Follow semantic versioning

## Migration from Legacy .cursorrules Files

If you're using the legacy single-file `.cursorrules` format, here's how to migrate:

### Why Migrate?

The legacy `.cursorrules` file format has been deprecated in favor of `.cursor/rules/*.mdc`:

| Legacy Format | Modern Format |
|--------------|---------------|
| Single monolithic file | Multiple focused files |
| Rules always apply | Context-aware with glob patterns |
| Hard to maintain | Easy to update individual rules |
| Performance impact | Only loads relevant rules |
| Merge conflicts in teams | Clean separation of concerns |

### Migration Steps

1. **Backup** your existing `.cursorrules` file
2. **Copy** the new `.cursor` folder to your project root
3. **Test** that rules apply correctly (restart Cursor if needed)
4. **Customize** any rules specific to your project
5. **Remove** the old `.cursorrules` file when satisfied

### What Changed?

- **File format**: Plain text → YAML frontmatter + Markdown
- **Organization**: Single file → Multiple categorized files
- **Application**: Always applied → Glob-pattern based
- **Location**: `.cursorrules` → `.cursor/rules/*.mdc`

## Contributing to These Rules

These rules should evolve based on:
- WordPress core updates
- New best practices
- Community feedback
- Personal experience with projects

Keep documentation updated as WordPress evolves and new patterns emerge.

### Rule Development Guidelines

When adding or modifying rules:
- Keep rule content concise (under 20 lines)
- Use specific glob patterns for context-aware rules
- Set `alwaysApply: true` only for critical rules (security, core principles)
- Test rules with actual WordPress projects
- Document any custom patterns clearly

## Resources

### Official Documentation
- [Cursor Rules Documentation](https://docs.cursor.com/en/context) - MDC file format and context system
- [WordPress Developer Resources](https://developer.wordpress.org/)
- [WordPress Coding Standards](https://developer.wordpress.org/coding-standards/)
- [Plugin Developer Handbook](https://developer.wordpress.org/plugins/)
- [Theme Developer Handbook](https://developer.wordpress.org/themes/)
- [Block Editor Handbook](https://developer.wordpress.org/block-editor/)

### Tools
- [Local by Flywheel](https://localwp.com/) - Local WordPress development
- [WP-CLI](https://wp-cli.org/) - Command-line interface for WordPress
- [Query Monitor](https://wordpress.org/plugins/query-monitor/) - Debugging plugin
- [Advanced Custom Fields](https://www.advancedcustomfields.com/) - Custom fields plugin

### Community
- [WordPress Stack Exchange](https://wordpress.stackexchange.com/)
- [Make WordPress](https://make.wordpress.org/)
- [WordPress TV](https://wordpress.tv/)

## License

These cursor rules are provided as-is for personal and commercial use. Adapt them to your needs.

---

**Last Updated**: November 2025
**Maintained by**: Gabriel Ciuca Albu

