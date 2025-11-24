# General WordPress Development Rules

## Purpose

This document provides comprehensive cursor rules and guidelines for general WordPress development. These rules apply to all WordPress projects, whether you're building themes, plugins, or customizations.

## Key Principles

### 1. WordPress Coding Standards (WPCS)

Follow the official WordPress Coding Standards for all code:

- **PHP**: Follow [WordPress PHP Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/)
- **JavaScript**: Follow [WordPress JavaScript Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/javascript/)
- **CSS**: Follow [WordPress CSS Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/css/)
- **HTML**: Follow [WordPress HTML Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/html/)

### 2. Security First

Security is paramount in WordPress development:

- **Sanitize** all input data
- **Escape** all output data
- **Validate** data types and formats
- **Use nonces** for form submissions
- **Check capabilities** before allowing actions

### 3. Performance Optimization

Write efficient code that scales:

- Minimize database queries
- Use WordPress caching mechanisms
- Optimize asset loading
- Avoid blocking operations
- Use transients for expensive operations

### 4. Use WordPress Core Functions

Don't reinvent the wheel:

- Use built-in WordPress functions when available
- Leverage WordPress APIs (Settings, Options, Transients, etc.)
- Follow WordPress best practices and patterns
- Stay compatible with WordPress core

## Common Patterns

### File Structure

```
wp-content/
├── themes/
│   └── your-theme/
│       ├── style.css
│       ├── functions.php
│       ├── index.php
│       ├── header.php
│       ├── footer.php
│       ├── sidebar.php
│       └── ...
├── plugins/
│   └── your-plugin/
│       ├── your-plugin.php
│       ├── includes/
│       ├── assets/
│       └── ...
└── uploads/
```

### Naming Conventions

```php
// Functions: lowercase with underscores
function my_custom_function() {}

// Classes: Capitalized words, underscores separate words
class My_Custom_Class {}

// Constants: Uppercase with underscores
define( 'MY_CONSTANT', 'value' );

// Variables: Lowercase with underscores
$my_variable = 'value';

// Hooks: Lowercase with underscores, prefixed
add_action( 'my_prefix_custom_action', 'callback' );
add_filter( 'my_prefix_custom_filter', 'callback' );
```

### Security Patterns

#### Sanitization (Input)

```php
// Sanitize text input
$text = sanitize_text_field( $_POST['text'] );

// Sanitize email
$email = sanitize_email( $_POST['email'] );

// Sanitize URL
$url = esc_url_raw( $_POST['url'] );

// Sanitize textarea
$textarea = sanitize_textarea_field( $_POST['textarea'] );

// Sanitize HTML
$html = wp_kses_post( $_POST['content'] );

// Sanitize file name
$filename = sanitize_file_name( $_FILES['file']['name'] );
```

#### Escaping (Output)

```php
// Escape HTML
echo esc_html( $text );

// Escape attributes
echo '<div class="' . esc_attr( $class ) . '">';

// Escape URLs
echo '<a href="' . esc_url( $url ) . '">';

// Escape JavaScript
echo '<script>var data = ' . wp_json_encode( $data ) . ';</script>';

// Escape SQL (use $wpdb->prepare instead)
$wpdb->prepare( "SELECT * FROM table WHERE id = %d", $id );
```

#### Nonces

```php
// Create nonce
wp_nonce_field( 'my_action', 'my_nonce' );

// Verify nonce
if ( ! isset( $_POST['my_nonce'] ) || ! wp_verify_nonce( $_POST['my_nonce'], 'my_action' ) ) {
    wp_die( 'Security check failed' );
}

// URL nonce
$url = wp_nonce_url( admin_url( 'admin.php?action=my_action' ), 'my_action' );

// Verify URL nonce
if ( ! isset( $_GET['_wpnonce'] ) || ! wp_verify_nonce( $_GET['_wpnonce'], 'my_action' ) ) {
    wp_die( 'Security check failed' );
}
```

#### Capability Checks

```php
// Check if user can perform action
if ( ! current_user_can( 'manage_options' ) ) {
    wp_die( 'You do not have sufficient permissions' );
}

// Check for specific post
if ( ! current_user_can( 'edit_post', $post_id ) ) {
    wp_die( 'You cannot edit this post' );
}

// Common capabilities
// - manage_options (Administrator)
// - edit_posts (Editor, Administrator)
// - publish_posts (Author, Editor, Administrator)
// - edit_published_posts
// - delete_posts
```

### Database Query Patterns

#### Using $wpdb

```php
global $wpdb;

// Prepared statement (ALWAYS use prepare for user input)
$results = $wpdb->get_results( 
    $wpdb->prepare( 
        "SELECT * FROM {$wpdb->prefix}custom_table WHERE user_id = %d AND status = %s",
        $user_id,
        $status
    )
);

// Get single variable
$count = $wpdb->get_var( 
    $wpdb->prepare( "SELECT COUNT(*) FROM {$wpdb->prefix}posts WHERE post_status = %s", 'publish' )
);

// Get single row
$row = $wpdb->get_row( 
    $wpdb->prepare( "SELECT * FROM {$wpdb->prefix}posts WHERE ID = %d", $post_id )
);

// Insert
$wpdb->insert(
    $wpdb->prefix . 'custom_table',
    array(
        'column1' => $value1,
        'column2' => $value2,
    ),
    array( '%s', '%d' ) // Format: %s (string), %d (integer), %f (float)
);

// Update
$wpdb->update(
    $wpdb->prefix . 'custom_table',
    array( 'column1' => $new_value ),
    array( 'id' => $id ),
    array( '%s' ),
    array( '%d' )
);

// Delete
$wpdb->delete(
    $wpdb->prefix . 'custom_table',
    array( 'id' => $id ),
    array( '%d' )
);
```

#### Using WP_Query

```php
// Basic query
$args = array(
    'post_type'      => 'post',
    'posts_per_page' => 10,
    'post_status'    => 'publish',
);
$query = new WP_Query( $args );

if ( $query->have_posts() ) {
    while ( $query->have_posts() ) {
        $query->the_post();
        // Display post content
        the_title();
        the_content();
    }
    wp_reset_postdata();
}

// Advanced query with meta
$args = array(
    'post_type'      => 'product',
    'posts_per_page' => -1,
    'meta_query'     => array(
        'relation' => 'AND',
        array(
            'key'     => 'price',
            'value'   => 100,
            'compare' => '>=',
            'type'    => 'NUMERIC',
        ),
        array(
            'key'     => 'in_stock',
            'value'   => 'yes',
        ),
    ),
    'tax_query'      => array(
        array(
            'taxonomy' => 'category',
            'field'    => 'slug',
            'terms'    => 'featured',
        ),
    ),
);
$query = new WP_Query( $args );
```

### Hooks and Filters

#### Actions

```php
// Add action
add_action( 'init', 'my_custom_init_function' );

function my_custom_init_function() {
    // Your code here
}

// Action with priority and arguments
add_action( 'save_post', 'my_save_post_function', 10, 2 );

function my_save_post_function( $post_id, $post ) {
    // Your code here
}

// Remove action
remove_action( 'init', 'my_custom_init_function' );

// Do action (custom)
do_action( 'my_custom_action', $arg1, $arg2 );
```

#### Filters

```php
// Add filter
add_filter( 'the_content', 'my_content_filter' );

function my_content_filter( $content ) {
    // Modify content
    return $content;
}

// Filter with priority and arguments
add_filter( 'the_title', 'my_title_filter', 10, 2 );

function my_title_filter( $title, $post_id ) {
    // Modify title
    return $title;
}

// Remove filter
remove_filter( 'the_content', 'my_content_filter' );

// Apply filter (custom)
$value = apply_filters( 'my_custom_filter', $value, $arg1, $arg2 );
```

### Caching and Transients

```php
// Set transient (expires after specified time)
set_transient( 'my_transient', $data, HOUR_IN_SECONDS );

// Get transient
$data = get_transient( 'my_transient' );
if ( false === $data ) {
    // Data not in cache, regenerate
    $data = expensive_operation();
    set_transient( 'my_transient', $data, HOUR_IN_SECONDS );
}

// Delete transient
delete_transient( 'my_transient' );

// Site transients (network-wide in multisite)
set_site_transient( 'my_site_transient', $data, DAY_IN_SECONDS );
$data = get_site_transient( 'my_site_transient' );
delete_site_transient( 'my_site_transient' );

// Object cache (if available)
wp_cache_set( 'my_key', $data, 'my_group', 3600 );
$data = wp_cache_get( 'my_key', 'my_group' );
wp_cache_delete( 'my_key', 'my_group' );
```

### Options API

```php
// Get option
$value = get_option( 'my_option', 'default_value' );

// Update option
update_option( 'my_option', $new_value );

// Add option (only if it doesn't exist)
add_option( 'my_option', $value );

// Delete option
delete_option( 'my_option' );

// Site options (network-wide in multisite)
get_site_option( 'my_site_option' );
update_site_option( 'my_site_option', $value );
delete_site_option( 'my_site_option' );
```

### Custom Post Types

```php
function register_custom_post_type() {
    $args = array(
        'labels'              => array(
            'name'          => 'Products',
            'singular_name' => 'Product',
        ),
        'public'              => true,
        'has_archive'         => true,
        'rewrite'             => array( 'slug' => 'products' ),
        'supports'            => array( 'title', 'editor', 'thumbnail', 'excerpt' ),
        'show_in_rest'        => true, // Enable Gutenberg
        'menu_icon'           => 'dashicons-products',
    );
    
    register_post_type( 'product', $args );
}
add_action( 'init', 'register_custom_post_type' );
```

### Custom Taxonomies

```php
function register_custom_taxonomy() {
    $args = array(
        'labels'            => array(
            'name'          => 'Product Categories',
            'singular_name' => 'Product Category',
        ),
        'hierarchical'      => true, // true = like categories, false = like tags
        'public'            => true,
        'show_in_rest'      => true,
        'rewrite'           => array( 'slug' => 'product-category' ),
    );
    
    register_taxonomy( 'product_category', 'product', $args );
}
add_action( 'init', 'register_custom_taxonomy' );
```

## Best Practices

### DO's

1. **Use WordPress Functions**: Always use WordPress core functions instead of raw PHP when available
2. **Prefix Everything**: Use unique prefixes for functions, classes, and constants to avoid conflicts
3. **Sanitize Input**: Always sanitize user input before processing
4. **Escape Output**: Always escape data before outputting to prevent XSS
5. **Use Nonces**: Implement nonces for all form submissions and AJAX requests
6. **Check Capabilities**: Always verify user permissions before performing actions
7. **Prepare Database Queries**: Use $wpdb->prepare() for all database queries with variables
8. **Use Transients**: Cache expensive operations using transients
9. **Enqueue Assets**: Use wp_enqueue_script() and wp_enqueue_style() for all assets
10. **Comment Your Code**: Write clear comments explaining complex logic
11. **Use Hooks**: Leverage WordPress hooks instead of modifying core files
12. **Handle Errors**: Implement proper error handling and validation
13. **Use wp_reset_postdata()**: Always reset post data after custom queries
14. **Follow WPCS**: Adhere to WordPress Coding Standards
15. **Test Thoroughly**: Test on different PHP versions and WordPress versions

### DON'Ts

1. **Don't Modify Core**: Never modify WordPress core files
2. **Don't Use $_GET/$_POST Directly**: Always sanitize user input first
3. **Don't Echo Unsanitized Data**: Always escape output
4. **Don't Use mysql_* Functions**: Use $wpdb instead
5. **Don't Hard-code URLs**: Use WordPress functions like get_permalink(), home_url()
6. **Don't Suppress Errors with @**: Properly handle errors instead
7. **Don't Use query_posts()**: Use WP_Query or get_posts() instead
8. **Don't Forget Nonces**: Always implement nonce verification
9. **Don't Ignore Errors**: Check for errors and handle them appropriately
10. **Don't Use Global $post Directly**: Use get_post() or setup_postdata()
11. **Don't Hardcode Database Tables**: Use $wpdb->prefix
12. **Don't Use extract()**: It can cause variable name conflicts
13. **Don't Load Unnecessary Assets**: Only enqueue scripts/styles when needed
14. **Don't Create Slow Queries**: Optimize database queries
15. **Don't Store Credentials in Code**: Use wp-config.php or environment variables

## Cursor-Specific Instructions

When prompting Cursor AI for WordPress development:

1. **Specify WordPress Context**: Always mention you're working with WordPress
2. **Request Security Measures**: Explicitly ask for sanitization, escaping, and nonces
3. **Ask for WPCS Compliance**: Request code that follows WordPress Coding Standards
4. **Mention Performance**: Ask for optimized, efficient code with proper caching
5. **Request Comments**: Ask for well-commented code explaining functionality
6. **Specify Hooks**: Mention you want to use WordPress hooks (actions/filters)
7. **Ask for Validation**: Request proper error handling and validation

### Example Prompts for Cursor

```
"Create a WordPress function that [describes functionality]. 
Follow WordPress coding standards, include proper sanitization and escaping, 
and add security checks with nonces and capability verification."

"Build a custom WordPress query that [describes query]. 
Use WP_Query, optimize for performance with proper caching, 
and include error handling."

"Implement a WordPress hook that [describes action]. 
Use the appropriate action or filter, include comments explaining the code, 
and follow WordPress best practices."
```

## Common Pitfalls

### 1. SQL Injection

```php
// WRONG - Vulnerable to SQL injection
$results = $wpdb->get_results( "SELECT * FROM table WHERE id = {$_GET['id']}" );

// CORRECT - Using prepared statements
$results = $wpdb->get_results( 
    $wpdb->prepare( "SELECT * FROM table WHERE id = %d", $_GET['id'] )
);
```

### 2. XSS Vulnerabilities

```php
// WRONG - Can output malicious JavaScript
echo $_POST['user_input'];

// CORRECT - Escaped output
echo esc_html( $_POST['user_input'] );
```

### 3. Missing Nonce Verification

```php
// WRONG - No security check
if ( isset( $_POST['submit'] ) ) {
    update_option( 'my_option', $_POST['value'] );
}

// CORRECT - With nonce verification
if ( isset( $_POST['submit'] ) && check_admin_referer( 'my_action', 'my_nonce' ) ) {
    update_option( 'my_option', sanitize_text_field( $_POST['value'] ) );
}
```

### 4. Not Checking Capabilities

```php
// WRONG - Anyone can delete
if ( isset( $_GET['delete'] ) ) {
    wp_delete_post( $_GET['post_id'] );
}

// CORRECT - Check user capability
if ( isset( $_GET['delete'] ) && current_user_can( 'delete_posts' ) ) {
    wp_delete_post( absint( $_GET['post_id'] ) );
}
```

### 5. Inefficient Queries

```php
// WRONG - Queries all posts
$all_posts = get_posts( array( 'numberposts' => -1 ) );
foreach ( $all_posts as $post ) {
    if ( get_post_meta( $post->ID, 'featured', true ) == 'yes' ) {
        // Process post
    }
}

// CORRECT - Query only what you need
$featured_posts = get_posts( array(
    'meta_key'   => 'featured',
    'meta_value' => 'yes',
    'posts_per_page' => 10,
) );
```

## Resources

### Official Documentation
- [WordPress Developer Resources](https://developer.wordpress.org/)
- [WordPress Coding Standards](https://developer.wordpress.org/coding-standards/)
- [WordPress APIs](https://codex.wordpress.org/WordPress_APIs)
- [Data Validation](https://developer.wordpress.org/plugins/security/data-validation/)
- [Securing Input](https://developer.wordpress.org/plugins/security/securing-input/)
- [Securing Output](https://developer.wordpress.org/plugins/security/securing-output/)

### Tools
- [PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer) - Code standards checker
- [WordPress Coding Standards](https://github.com/WordPress/WordPress-Coding-Standards) - WPCS for PHPCS
- [Query Monitor](https://wordpress.org/plugins/query-monitor/) - Debugging plugin
- [Debug Bar](https://wordpress.org/plugins/debug-bar/) - Debugging toolbar

### Books and Guides
- [WordPress Plugin Handbook](https://developer.wordpress.org/plugins/)
- [WordPress Theme Handbook](https://developer.wordpress.org/themes/)
- [Common APIs Handbook](https://developer.wordpress.org/apis/)

---

**Remember**: Security, performance, and code quality are not optional in WordPress development. Always follow these guidelines to build robust, secure, and maintainable WordPress solutions.

