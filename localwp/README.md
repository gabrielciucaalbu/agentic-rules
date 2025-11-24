# LocalWP Cursor Rules

A comprehensive collection of cursor rules and guidelines for WordPress development using Local by Flywheel. These rules help you leverage Cursor AI to build WordPress websites faster and more efficiently while following best practices.

## Overview

This repository contains specialized documentation for different aspects of WordPress development. Each file provides cursor rules, patterns, and best practices for a specific area of WordPress development.

## Quick Reference

### Core Documentation

- **[General WordPress](general-wordpress.md)** - WordPress coding standards, security, performance, and best practices
- **[Theme Development](theme-development.md)** - Custom theme creation, template hierarchy, and theme.json
- **[Plugin Development](plugin-development.md)** - Creating plugins, hooks, filters, and WordPress APIs

### Specialized Areas

- **[ACF Integration](acf-integration.md)** - Advanced Custom Fields patterns and best practices
- **[WooCommerce](woocommerce.md)** - E-commerce customization and development
- **[Gutenberg Blocks](gutenberg-blocks.md)** - Block editor and custom block development
- **[REST API](rest-api.md)** - Custom endpoints and API customization

### Examples

- **[Example Templates](examples/)** - Ready-to-use .cursorrules files for different project types

## How to Use These Rules

### Method 1: Direct Reference (Recommended for this repo)

Keep this repository as a reference and consult the relevant documentation when working on WordPress projects. The markdown files contain detailed guidelines that help you prompt Cursor AI effectively.

### Method 2: Copy Rules to Project

1. Navigate to the `examples/` folder
2. Choose the appropriate `.cursorrules` template for your project type
3. Copy it to your WordPress project root as `.cursorrules`
4. Customize the rules based on your specific needs

### Method 3: Symlink (Advanced)

Create a symlink from your WordPress project to a rules file in this repository:

```bash
# From your WordPress project root
ln -s /path/to/agentic-rules/localwp/examples/.cursorrules-theme .cursorrules
```

### Method 4: Compose Custom Rules

Combine guidelines from multiple documentation files to create custom rules tailored to your project needs.

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
├── general-wordpress.md      # Core WordPress guidelines
├── theme-development.md      # Theme development rules
├── plugin-development.md     # Plugin development rules
├── acf-integration.md       # ACF patterns and practices
├── woocommerce.md           # WooCommerce development
├── gutenberg-blocks.md      # Block editor development
├── rest-api.md              # REST API customization
└── examples/                # Example .cursorrules files
    ├── .cursorrules-theme
    ├── .cursorrules-plugin
    └── .cursorrules-woo
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

## Contributing to These Rules

These rules should evolve based on:
- WordPress core updates
- New best practices
- Community feedback
- Personal experience with projects

Keep documentation updated as WordPress evolves and new patterns emerge.

## Resources

### Official Documentation
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

