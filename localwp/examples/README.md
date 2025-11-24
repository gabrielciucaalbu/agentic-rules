# Example .cursorrules Templates

This folder contains ready-to-use `.cursorrules` templates for different types of WordPress development projects.

## Available Templates

### .cursorrules-theme
For WordPress theme development projects.

**Use when:**
- Building a custom WordPress theme
- Creating a child theme
- Developing block themes
- Customizing theme templates

**Includes rules for:**
- Template hierarchy
- Theme structure
- Functions.php setup
- Asset enqueuing
- Security and sanitization
- Theme.json configuration

### .cursorrules-plugin
For WordPress plugin development projects.

**Use when:**
- Creating custom WordPress plugins
- Building admin functionality
- Adding custom post types
- Implementing shortcodes
- Creating REST API endpoints

**Includes rules for:**
- Plugin structure
- Activation/deactivation hooks
- Settings API
- Security best practices
- Database operations
- AJAX handling

### .cursorrules-woo
For WooCommerce development and customization.

**Use when:**
- Customizing WooCommerce stores
- Building WooCommerce extensions
- Creating custom payment gateways
- Modifying checkout process
- Adding product customizations

**Includes rules for:**
- WooCommerce hooks and filters
- Template overrides
- Cart and checkout customization
- Product operations
- Order management
- Payment gateway development

## How to Use These Templates

### Method 1: Copy to Your Project

1. Choose the appropriate template for your project type
2. Copy the file to your WordPress project root
3. Rename it to `.cursorrules` (remove the suffix)

```bash
# Example for theme development
cp .cursorrules-theme /path/to/your/theme/.cursorrules

# Example for plugin development
cp .cursorrules-plugin /path/to/your/plugin/.cursorrules

# Example for WooCommerce
cp .cursorrules-woo /path/to/your/theme/.cursorrules
```

### Method 2: Symlink (Advanced)

Create a symlink to keep the rules up to date:

```bash
# Link theme rules
ln -s /path/to/agentic-rules/localwp/examples/.cursorrules-theme /path/to/your/theme/.cursorrules

# Link plugin rules
ln -s /path/to/agentic-rules/localwp/examples/.cursorrules-plugin /path/to/your/plugin/.cursorrules
```

**Advantage:** Rules stay synchronized with this repository  
**Disadvantage:** Changes here affect your projects immediately

### Method 3: Combine Multiple Templates

For complex projects, you can combine rules from multiple templates:

```bash
# Example: WooCommerce theme (combine theme + woo rules)
cat .cursorrules-theme .cursorrules-woo > /path/to/your/theme/.cursorrules
```

### Method 4: Reference in Your Own .cursorrules

Create your own `.cursorrules` file that references these:

```
Follow WordPress theme development best practices as outlined in:
/path/to/agentic-rules/localwp/general-wordpress.md
/path/to/agentic-rules/localwp/theme-development.md

Additionally, for this project:
- Use Tailwind CSS for styling
- Follow BEM naming convention for custom CSS
- Implement lazy loading for images
```

## Customizing Templates

These templates are starting points. Feel free to:

1. **Add project-specific rules:**
   ```
   # Project-specific rules
   - Use custom post type 'portfolio'
   - Implement custom taxonomy 'project-type'
   - Follow company coding standards
   ```

2. **Remove irrelevant rules:**
   - Comment out or delete rules that don't apply to your project

3. **Adjust priorities:**
   - Emphasize specific aspects important to your project
   - Add stricter requirements where needed

## Best Practices

### DO:
- ✅ Start with the closest matching template
- ✅ Customize for your specific needs
- ✅ Keep rules concise and actionable
- ✅ Update rules as your project evolves
- ✅ Share common patterns across team projects

### DON'T:
- ❌ Use multiple conflicting templates at once
- ❌ Make rules too vague ("be good")
- ❌ Include rules for technologies you're not using
- ❌ Forget to update rules when requirements change

## Examples of Combined Use

### WordPress + WooCommerce Theme
```bash
cat .cursorrules-theme .cursorrules-woo > .cursorrules
```

### WordPress Plugin with REST API
Copy `.cursorrules-plugin` and add REST API specific rules:
```
Additionally for REST API development:
- Use register_rest_route() for all custom endpoints
- Implement proper permission_callback
- Validate all input parameters
- Return appropriate HTTP status codes
```

### Theme with ACF Integration
Copy `.cursorrules-theme` and add:
```
ACF Integration Rules:
- Use get_field() instead of the_field()
- Always check if field exists before displaying
- Use ACF blocks for custom Gutenberg blocks
- Escape all ACF field output
```

## Version Control

If using version control:

**Option 1: Commit .cursorrules**
- Ensures team consistency
- Rules versioned with project
- Team must sync manually

**Option 2: .gitignore .cursorrules**
- Each developer can customize
- Add `.cursorrules.example` as template
- Good for different development styles

## Updating Templates

These templates may be updated over time. To get updates:

```bash
# If using symlinks, just pull repository updates
cd /path/to/agentic-rules
git pull

# If using copied files, re-copy the template
cp /path/to/agentic-rules/localwp/examples/.cursorrules-theme .cursorrules
```

## Troubleshooting

### Cursor Not Following Rules

1. Ensure `.cursorrules` is in project root
2. Check file is actually named `.cursorrules` (with dot)
3. Verify file has content and is not empty
4. Restart Cursor if needed
5. Be specific in your prompts to Cursor

### Rules Too Strict

If Cursor is being overly cautious:
- Simplify the rules
- Remove less important guidelines
- Be more explicit in your requests to override specific rules

### Rules Not Strict Enough

Add more specific requirements:
```
CRITICAL RULES (never skip):
- Always escape output with esc_html()
- Always sanitize input with sanitize_text_field()
- Always check current_user_can() before protected operations
```

## Contributing

If you have improvements to these templates:
1. Test them on real projects
2. Ensure they follow WordPress standards
3. Keep them general enough for most use cases
4. Update this README with examples

## Additional Resources

For more detailed information, see the main documentation files:
- `../general-wordpress.md` - WordPress best practices
- `../theme-development.md` - Theme development guide
- `../plugin-development.md` - Plugin development guide
- `../woocommerce.md` - WooCommerce development guide
- `../acf-integration.md` - ACF development guide
- `../gutenberg-blocks.md` - Block development guide
- `../rest-api.md` - REST API development guide

---

**Note:** These templates are designed to help Cursor AI understand WordPress best practices. They work best when combined with clear, specific prompts about what you want to build.

