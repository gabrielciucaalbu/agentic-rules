# WordPress Plugin Development Rules

## Purpose

Comprehensive guidelines for building WordPress plugins following best practices. This document covers plugin structure, hooks and filters, WordPress APIs, security, and maintainable plugin architecture.

## Key Principles

### 1. Single Responsibility

Each plugin should do one thing well. Don't create a plugin that tries to do everything.

### 2. Use WordPress APIs

Leverage WordPress's built-in APIs instead of creating custom solutions:
- Settings API
- Options API
- Transients API
- HTTP API
- Rewrite API

### 3. Security First

- Validate and sanitize all input
- Escape all output
- Use nonces for form submissions
- Check user capabilities
- Prepare database queries

### 4. Hooks Over Direct Modifications

Never modify WordPress core. Use hooks (actions and filters) to extend functionality.

## Plugin File Structure

### Basic Plugin Structure

```
plugin-name/
├── plugin-name.php           # Main plugin file (REQUIRED)
├── uninstall.php             # Uninstall cleanup
├── README.txt                # WordPress.org readme
├── readme.md                 # GitHub readme
├── includes/                 # PHP classes and functions
│   ├── class-plugin-name.php
│   ├── class-admin.php
│   ├── class-public.php
│   └── functions.php
├── admin/                    # Admin-specific functionality
│   ├── css/
│   │   └── admin.css
│   ├── js/
│   │   └── admin.js
│   ├── partials/
│   │   └── admin-display.php
│   └── class-admin.php
├── public/                   # Public-facing functionality
│   ├── css/
│   │   └── public.css
│   ├── js/
│   │   └── public.js
│   └── class-public.php
├── languages/                # Translation files
│   └── plugin-name.pot
└── assets/                   # Plugin assets (for repository)
    ├── icon-256x256.png
    └── banner-1544x500.png
```

### Object-Oriented Structure (Advanced)

```
plugin-name/
├── plugin-name.php
├── includes/
│   ├── class-plugin-name.php              # Main plugin class
│   ├── class-loader.php                   # Hooks loader
│   ├── class-i18n.php                     # Internationalization
│   ├── class-activator.php                # Activation hooks
│   ├── class-deactivator.php              # Deactivation hooks
│   └── class-uninstaller.php              # Uninstall cleanup
└── ...
```

## Common Patterns

### Main Plugin File Header

```php
<?php
/**
 * Plugin Name:       My Custom Plugin
 * Plugin URI:        https://example.com/plugin-name
 * Description:       A brief description of what the plugin does.
 * Version:           1.0.0
 * Requires at least: 6.0
 * Requires PHP:      7.4
 * Author:            Your Name
 * Author URI:        https://example.com
 * License:           GPL v2 or later
 * License URI:       https://www.gnu.org/licenses/gpl-2.0.html
 * Text Domain:       my-custom-plugin
 * Domain Path:       /languages
 */

// If this file is called directly, abort.
if ( ! defined( 'WPINC' ) ) {
    die;
}

// Plugin version
define( 'MY_PLUGIN_VERSION', '1.0.0' );

// Plugin directory path
define( 'MY_PLUGIN_PATH', plugin_dir_path( __FILE__ ) );

// Plugin directory URL
define( 'MY_PLUGIN_URL', plugin_dir_url( __FILE__ ) );

/**
 * The code that runs during plugin activation.
 */
function activate_my_plugin() {
    require_once MY_PLUGIN_PATH . 'includes/class-activator.php';
    My_Plugin_Activator::activate();
}
register_activation_hook( __FILE__, 'activate_my_plugin' );

/**
 * The code that runs during plugin deactivation.
 */
function deactivate_my_plugin() {
    require_once MY_PLUGIN_PATH . 'includes/class-deactivator.php';
    My_Plugin_Deactivator::deactivate();
}
register_deactivation_hook( __FILE__, 'deactivate_my_plugin' );

/**
 * Initialize the plugin.
 */
function run_my_plugin() {
    require_once MY_PLUGIN_PATH . 'includes/class-plugin-name.php';
    $plugin = new My_Plugin();
    $plugin->run();
}
run_my_plugin();
```

### Plugin Activation

```php
<?php
/**
 * Fired during plugin activation.
 */
class My_Plugin_Activator {
    
    public static function activate() {
        // Create custom database tables if needed
        self::create_tables();
        
        // Set default options
        self::set_default_options();
        
        // Flush rewrite rules if adding custom post types or endpoints
        flush_rewrite_rules();
        
        // Schedule cron events
        if ( ! wp_next_scheduled( 'my_plugin_daily_event' ) ) {
            wp_schedule_event( time(), 'daily', 'my_plugin_daily_event' );
        }
        
        // Check for required PHP version
        if ( version_compare( PHP_VERSION, '7.4', '<' ) ) {
            deactivate_plugins( plugin_basename( __FILE__ ) );
            wp_die( 
                esc_html__( 'This plugin requires PHP 7.4 or higher.', 'my-custom-plugin' )
            );
        }
    }
    
    private static function create_tables() {
        global $wpdb;
        $charset_collate = $wpdb->get_charset_collate();
        $table_name = $wpdb->prefix . 'my_plugin_data';
        
        $sql = "CREATE TABLE IF NOT EXISTS $table_name (
            id mediumint(9) NOT NULL AUTO_INCREMENT,
            user_id bigint(20) NOT NULL,
            data_value text NOT NULL,
            created_at datetime DEFAULT CURRENT_TIMESTAMP NOT NULL,
            PRIMARY KEY  (id),
            KEY user_id (user_id)
        ) $charset_collate;";
        
        require_once ABSPATH . 'wp-admin/includes/upgrade.php';
        dbDelta( $sql );
    }
    
    private static function set_default_options() {
        $defaults = array(
            'my_plugin_option_1' => 'default_value',
            'my_plugin_option_2' => true,
        );
        
        foreach ( $defaults as $option => $value ) {
            if ( ! get_option( $option ) ) {
                add_option( $option, $value );
            }
        }
    }
}
```

### Plugin Deactivation

```php
<?php
/**
 * Fired during plugin deactivation.
 */
class My_Plugin_Deactivator {
    
    public static function deactivate() {
        // Clear scheduled events
        $timestamp = wp_next_scheduled( 'my_plugin_daily_event' );
        if ( $timestamp ) {
            wp_unschedule_event( $timestamp, 'my_plugin_daily_event' );
        }
        
        // Flush rewrite rules
        flush_rewrite_rules();
        
        // Note: Don't delete options or data here
        // That should be done in uninstall.php
    }
}
```

### Plugin Uninstall (uninstall.php)

```php
<?php
/**
 * Fired when the plugin is uninstalled.
 */

// If uninstall not called from WordPress, then exit.
if ( ! defined( 'WP_UNINSTALL_PLUGIN' ) ) {
    exit;
}

// Delete options
$options = array(
    'my_plugin_option_1',
    'my_plugin_option_2',
);

foreach ( $options as $option ) {
    delete_option( $option );
}

// For multisite
if ( is_multisite() ) {
    foreach ( $options as $option ) {
        delete_site_option( $option );
    }
}

// Drop custom tables
global $wpdb;
$table_name = $wpdb->prefix . 'my_plugin_data';
$wpdb->query( "DROP TABLE IF EXISTS $table_name" );

// Delete post meta for all posts
$wpdb->query( "DELETE FROM $wpdb->postmeta WHERE meta_key LIKE 'my_plugin_%'" );

// Delete user meta for all users
$wpdb->query( "DELETE FROM $wpdb->usermeta WHERE meta_key LIKE 'my_plugin_%'" );

// Delete transients
$wpdb->query( "DELETE FROM $wpdb->options WHERE option_name LIKE '_transient_my_plugin_%'" );
$wpdb->query( "DELETE FROM $wpdb->options WHERE option_name LIKE '_transient_timeout_my_plugin_%'" );

// Clear any cached data
wp_cache_flush();
```

### Hooks and Filters

#### Adding Actions

```php
<?php
// Simple action
add_action( 'init', 'my_plugin_init' );

function my_plugin_init() {
    // Your code here
}

// Action with priority and parameters
add_action( 'save_post', 'my_plugin_save_post', 10, 3 );

function my_plugin_save_post( $post_id, $post, $update ) {
    // Verify nonce
    if ( ! isset( $_POST['my_plugin_nonce'] ) || 
         ! wp_verify_nonce( $_POST['my_plugin_nonce'], 'my_plugin_save' ) ) {
        return;
    }
    
    // Check autosave
    if ( defined( 'DOING_AUTOSAVE' ) && DOING_AUTOSAVE ) {
        return;
    }
    
    // Check permissions
    if ( ! current_user_can( 'edit_post', $post_id ) ) {
        return;
    }
    
    // Your code here
    if ( isset( $_POST['my_custom_field'] ) ) {
        update_post_meta( 
            $post_id, 
            'my_custom_field', 
            sanitize_text_field( $_POST['my_custom_field'] ) 
        );
    }
}

// Class-based actions
class My_Plugin_Class {
    
    public function __construct() {
        add_action( 'init', array( $this, 'init' ) );
        add_action( 'admin_menu', array( $this, 'add_menu' ) );
    }
    
    public function init() {
        // Initialization code
    }
    
    public function add_menu() {
        add_menu_page(
            'My Plugin',
            'My Plugin',
            'manage_options',
            'my-plugin',
            array( $this, 'display_page' ),
            'dashicons-admin-generic'
        );
    }
    
    public function display_page() {
        // Display admin page
    }
}

new My_Plugin_Class();
```

#### Adding Filters

```php
<?php
// Simple filter
add_filter( 'the_content', 'my_plugin_content_filter' );

function my_plugin_content_filter( $content ) {
    // Modify content
    $custom_content = '<div class="my-custom-wrapper">' . $content . '</div>';
    return $custom_content;
}

// Filter with priority and multiple parameters
add_filter( 'the_title', 'my_plugin_title_filter', 10, 2 );

function my_plugin_title_filter( $title, $post_id ) {
    if ( get_post_type( $post_id ) === 'my_custom_type' ) {
        $title = '[Custom] ' . $title;
    }
    return $title;
}

// Conditional filter
function my_plugin_conditional_filter( $content ) {
    if ( is_single() && in_the_loop() && is_main_query() ) {
        $content .= '<p>Additional content for single posts.</p>';
    }
    return $content;
}
add_filter( 'the_content', 'my_plugin_conditional_filter', 20 );
```

### Creating Custom Hooks

```php
<?php
// In your plugin code where you want others to hook in

// Custom action
function my_plugin_process_data( $data ) {
    // Before processing
    do_action( 'my_plugin_before_process', $data );
    
    // Process data
    $processed = process_data( $data );
    
    // After processing
    do_action( 'my_plugin_after_process', $processed, $data );
    
    return $processed;
}

// Custom filter
function my_plugin_get_value( $id ) {
    $value = get_option( 'my_plugin_value_' . $id );
    
    // Allow filtering the value
    return apply_filters( 'my_plugin_value', $value, $id );
}

// Usage by other developers:
add_action( 'my_plugin_before_process', function( $data ) {
    // Custom code before processing
} );

add_filter( 'my_plugin_value', function( $value, $id ) {
    // Modify the value
    return $value * 2;
}, 10, 2 );
```

### Settings API

```php
<?php
/**
 * Register plugin settings
 */
function my_plugin_register_settings() {
    // Register a setting
    register_setting(
        'my_plugin_options_group',  // Option group
        'my_plugin_options',         // Option name
        array(
            'type'              => 'array',
            'sanitize_callback' => 'my_plugin_sanitize_options',
            'default'           => array(),
        )
    );
    
    // Add settings section
    add_settings_section(
        'my_plugin_main_section',           // ID
        __( 'Main Settings', 'my-custom-plugin' ),  // Title
        'my_plugin_section_callback',       // Callback
        'my_plugin_settings'                // Page
    );
    
    // Add settings fields
    add_settings_field(
        'my_plugin_text_field',             // ID
        __( 'Text Field', 'my-custom-plugin' ),     // Title
        'my_plugin_text_field_callback',    // Callback
        'my_plugin_settings',               // Page
        'my_plugin_main_section'            // Section
    );
    
    add_settings_field(
        'my_plugin_checkbox_field',
        __( 'Enable Feature', 'my-custom-plugin' ),
        'my_plugin_checkbox_field_callback',
        'my_plugin_settings',
        'my_plugin_main_section'
    );
}
add_action( 'admin_init', 'my_plugin_register_settings' );

/**
 * Section callback
 */
function my_plugin_section_callback() {
    echo '<p>' . esc_html__( 'Configure your plugin settings below.', 'my-custom-plugin' ) . '</p>';
}

/**
 * Text field callback
 */
function my_plugin_text_field_callback() {
    $options = get_option( 'my_plugin_options' );
    $value = isset( $options['text_field'] ) ? $options['text_field'] : '';
    ?>
    <input type="text" 
           name="my_plugin_options[text_field]" 
           value="<?php echo esc_attr( $value ); ?>" 
           class="regular-text" />
    <?php
}

/**
 * Checkbox field callback
 */
function my_plugin_checkbox_field_callback() {
    $options = get_option( 'my_plugin_options' );
    $checked = isset( $options['checkbox_field'] ) && $options['checkbox_field'] ? 'checked' : '';
    ?>
    <input type="checkbox" 
           name="my_plugin_options[checkbox_field]" 
           value="1" 
           <?php echo $checked; ?> />
    <?php
}

/**
 * Sanitize options
 */
function my_plugin_sanitize_options( $input ) {
    $output = array();
    
    if ( isset( $input['text_field'] ) ) {
        $output['text_field'] = sanitize_text_field( $input['text_field'] );
    }
    
    if ( isset( $input['checkbox_field'] ) ) {
        $output['checkbox_field'] = (bool) $input['checkbox_field'];
    }
    
    return $output;
}

/**
 * Display settings page
 */
function my_plugin_settings_page() {
    if ( ! current_user_can( 'manage_options' ) ) {
        return;
    }
    ?>
    <div class="wrap">
        <h1><?php echo esc_html( get_admin_page_title() ); ?></h1>
        <form action="options.php" method="post">
            <?php
            settings_fields( 'my_plugin_options_group' );
            do_settings_sections( 'my_plugin_settings' );
            submit_button();
            ?>
        </form>
    </div>
    <?php
}
```

### Admin Menu and Pages

```php
<?php
/**
 * Add admin menu
 */
function my_plugin_admin_menu() {
    // Top-level menu
    add_menu_page(
        __( 'My Plugin', 'my-custom-plugin' ),      // Page title
        __( 'My Plugin', 'my-custom-plugin' ),      // Menu title
        'manage_options',                            // Capability
        'my-plugin',                                 // Menu slug
        'my_plugin_main_page',                       // Callback function
        'dashicons-admin-generic',                   // Icon
        20                                           // Position
    );
    
    // Submenu page
    add_submenu_page(
        'my-plugin',                                 // Parent slug
        __( 'Settings', 'my-custom-plugin' ),       // Page title
        __( 'Settings', 'my-custom-plugin' ),       // Menu title
        'manage_options',                            // Capability
        'my-plugin-settings',                        // Menu slug
        'my_plugin_settings_page'                    // Callback function
    );
    
    // Add submenu to existing menu
    add_submenu_page(
        'options-general.php',                       // Parent (Settings menu)
        __( 'My Plugin Settings', 'my-custom-plugin' ),
        __( 'My Plugin', 'my-custom-plugin' ),
        'manage_options',
        'my-plugin-settings',
        'my_plugin_settings_page'
    );
    
    // Alternative: add to other menus
    // 'index.php'          - Dashboard
    // 'edit.php'           - Posts
    // 'upload.php'         - Media
    // 'edit.php?post_type=page' - Pages
    // 'edit-comments.php'  - Comments
    // 'themes.php'         - Appearance
    // 'plugins.php'        - Plugins
    // 'users.php'          - Users
    // 'tools.php'          - Tools
    // 'options-general.php'- Settings
}
add_action( 'admin_menu', 'my_plugin_admin_menu' );
```

### Enqueuing Scripts and Styles

```php
<?php
/**
 * Enqueue admin scripts and styles
 */
function my_plugin_admin_enqueue( $hook ) {
    // Only load on plugin pages
    if ( 'toplevel_page_my-plugin' !== $hook ) {
        return;
    }
    
    // Enqueue CSS
    wp_enqueue_style(
        'my-plugin-admin',
        MY_PLUGIN_URL . 'admin/css/admin.css',
        array(),
        MY_PLUGIN_VERSION
    );
    
    // Enqueue JavaScript
    wp_enqueue_script(
        'my-plugin-admin',
        MY_PLUGIN_URL . 'admin/js/admin.js',
        array( 'jquery' ),
        MY_PLUGIN_VERSION,
        true
    );
    
    // Localize script
    wp_localize_script(
        'my-plugin-admin',
        'myPluginData',
        array(
            'ajaxUrl' => admin_url( 'admin-ajax.php' ),
            'nonce'   => wp_create_nonce( 'my_plugin_nonce' ),
            'strings' => array(
                'confirm' => __( 'Are you sure?', 'my-custom-plugin' ),
                'success' => __( 'Success!', 'my-custom-plugin' ),
            ),
        )
    );
}
add_action( 'admin_enqueue_scripts', 'my_plugin_admin_enqueue' );

/**
 * Enqueue public scripts and styles
 */
function my_plugin_public_enqueue() {
    wp_enqueue_style(
        'my-plugin-public',
        MY_PLUGIN_URL . 'public/css/public.css',
        array(),
        MY_PLUGIN_VERSION
    );
    
    wp_enqueue_script(
        'my-plugin-public',
        MY_PLUGIN_URL . 'public/js/public.js',
        array( 'jquery' ),
        MY_PLUGIN_VERSION,
        true
    );
}
add_action( 'wp_enqueue_scripts', 'my_plugin_public_enqueue' );
```

### AJAX Handlers

```php
<?php
/**
 * AJAX handler for logged-in users
 */
function my_plugin_ajax_handler() {
    // Verify nonce
    check_ajax_referer( 'my_plugin_nonce', 'nonce' );
    
    // Check permissions
    if ( ! current_user_can( 'edit_posts' ) ) {
        wp_send_json_error( array(
            'message' => __( 'Insufficient permissions.', 'my-custom-plugin' ),
        ) );
    }
    
    // Get and sanitize data
    $data = isset( $_POST['data'] ) ? sanitize_text_field( $_POST['data'] ) : '';
    
    if ( empty( $data ) ) {
        wp_send_json_error( array(
            'message' => __( 'Data is required.', 'my-custom-plugin' ),
        ) );
    }
    
    // Process data
    $result = my_plugin_process_data( $data );
    
    // Send response
    if ( $result ) {
        wp_send_json_success( array(
            'message' => __( 'Data processed successfully.', 'my-custom-plugin' ),
            'result'  => $result,
        ) );
    } else {
        wp_send_json_error( array(
            'message' => __( 'Failed to process data.', 'my-custom-plugin' ),
        ) );
    }
}
add_action( 'wp_ajax_my_plugin_action', 'my_plugin_ajax_handler' );

/**
 * AJAX handler for non-logged-in users
 */
add_action( 'wp_ajax_nopriv_my_plugin_action', 'my_plugin_ajax_handler' );
```

### Shortcodes

```php
<?php
/**
 * Register shortcode
 */
function my_plugin_shortcode( $atts, $content = null ) {
    // Parse attributes
    $atts = shortcode_atts( array(
        'id'    => 0,
        'type'  => 'default',
        'class' => '',
    ), $atts, 'my_plugin' );
    
    // Sanitize attributes
    $id = absint( $atts['id'] );
    $type = sanitize_text_field( $atts['type'] );
    $class = sanitize_html_class( $atts['class'] );
    
    // Start output buffering
    ob_start();
    
    // Generate output
    ?>
    <div class="my-plugin-shortcode <?php echo esc_attr( $class ); ?>" data-id="<?php echo esc_attr( $id ); ?>">
        <h3><?php echo esc_html( $type ); ?></h3>
        <?php if ( $content ) : ?>
            <div class="content">
                <?php echo wp_kses_post( $content ); ?>
            </div>
        <?php endif; ?>
    </div>
    <?php
    
    // Return output
    return ob_get_clean();
}
add_shortcode( 'my_plugin', 'my_plugin_shortcode' );

// Usage: [my_plugin id="123" type="custom" class="my-class"]Content here[/my_plugin]
```

### Custom Meta Boxes

```php
<?php
/**
 * Add custom meta box
 */
function my_plugin_add_meta_box() {
    add_meta_box(
        'my_plugin_meta_box',                        // ID
        __( 'Custom Settings', 'my-custom-plugin' ), // Title
        'my_plugin_meta_box_callback',               // Callback
        array( 'post', 'page' ),                     // Post types
        'normal',                                     // Context (normal, side, advanced)
        'high'                                        // Priority
    );
}
add_action( 'add_meta_boxes', 'my_plugin_add_meta_box' );

/**
 * Meta box display callback
 */
function my_plugin_meta_box_callback( $post ) {
    // Add nonce field
    wp_nonce_field( 'my_plugin_meta_box', 'my_plugin_meta_box_nonce' );
    
    // Get current value
    $value = get_post_meta( $post->ID, 'my_plugin_field', true );
    ?>
    <p>
        <label for="my_plugin_field">
            <?php esc_html_e( 'Custom Field:', 'my-custom-plugin' ); ?>
        </label>
        <input type="text" 
               id="my_plugin_field" 
               name="my_plugin_field" 
               value="<?php echo esc_attr( $value ); ?>" 
               class="widefat" />
    </p>
    <?php
}

/**
 * Save meta box data
 */
function my_plugin_save_meta_box( $post_id ) {
    // Check nonce
    if ( ! isset( $_POST['my_plugin_meta_box_nonce'] ) || 
         ! wp_verify_nonce( $_POST['my_plugin_meta_box_nonce'], 'my_plugin_meta_box' ) ) {
        return;
    }
    
    // Check autosave
    if ( defined( 'DOING_AUTOSAVE' ) && DOING_AUTOSAVE ) {
        return;
    }
    
    // Check permissions
    if ( ! current_user_can( 'edit_post', $post_id ) ) {
        return;
    }
    
    // Save data
    if ( isset( $_POST['my_plugin_field'] ) ) {
        update_post_meta(
            $post_id,
            'my_plugin_field',
            sanitize_text_field( $_POST['my_plugin_field'] )
        );
    }
}
add_action( 'save_post', 'my_plugin_save_meta_box' );
```

## Best Practices

### DO's

1. **Use Unique Prefixes**: Prefix all functions, classes, and constants
2. **Check Capabilities**: Always verify user permissions
3. **Validate and Sanitize**: All input must be validated and sanitized
4. **Escape Output**: All output must be escaped
5. **Use Nonces**: Implement nonce verification for forms
6. **Hook, Don't Hack**: Use hooks instead of modifying core
7. **Load Conditionally**: Only load assets when needed
8. **Use WordPress APIs**: Leverage built-in APIs
9. **Handle Errors**: Implement proper error handling
10. **Document Code**: Add comments explaining functionality
11. **Use OOP When Appropriate**: Object-oriented for complex plugins
12. **Clean Up on Uninstall**: Remove all plugin data
13. **Version Your Plugin**: Use semantic versioning
14. **Make It Translatable**: Use i18n functions
15. **Test Thoroughly**: Test on different configurations

### DON'Ts

1. **Don't Modify Core**: Never change WordPress core files
2. **Don't Use Global Namespace**: Always use prefixes
3. **Don't Trust User Input**: Always sanitize
4. **Don't Echo Unsanitized**: Always escape output
5. **Don't Skip Nonces**: Always implement nonce verification
6. **Don't Assume Capabilities**: Always check permissions
7. **Don't Load Everywhere**: Conditionally enqueue assets
8. **Don't Use Deprecated Functions**: Check WordPress documentation
9. **Don't Store Sensitive Data**: Use WordPress encryption if needed
10. **Don't Delete Data on Deactivation**: Only delete in uninstall.php
11. **Don't Use extract()**: Can cause variable conflicts
12. **Don't Hardcode Values**: Make things configurable
13. **Don't Forget Error Handling**: Check for errors
14. **Don't Ignore Standards**: Follow WordPress Coding Standards
15. **Don't Skip Testing**: Test before releasing

## Cursor-Specific Instructions

When prompting Cursor AI for plugin development:

```
"Create a WordPress plugin that [describes functionality]. Follow WordPress
plugin standards, include proper security measures (sanitization, escaping,
nonces, capability checks), use WordPress APIs, and make it fully translatable."

"Build a [feature] for WordPress plugin. Use appropriate hooks and filters,
implement proper error handling, follow WordPress coding standards, and
include comments explaining the code."

"Create an admin settings page for WordPress plugin using Settings API.
Include form fields for [describe settings], proper sanitization, and nonce
verification."
```

## Common Pitfalls

1. **Not checking nonces**: Security vulnerability
2. **Not sanitizing input**: SQL injection and XSS risks
3. **Not escaping output**: XSS vulnerability
4. **Not checking capabilities**: Unauthorized access
5. **Using global namespace**: Function name conflicts
6. **Loading assets everywhere**: Performance impact
7. **Not using uninstall.php**: Leaving data behind
8. **Modifying core files**: Updates will break changes
9. **Not handling errors**: Poor user experience
10. **Forgetting translations**: Not accessible to non-English users

## Resources

- [Plugin Handbook](https://developer.wordpress.org/plugins/)
- [Plugin API](https://codex.wordpress.org/Plugin_API)
- [Settings API](https://developer.wordpress.org/plugins/settings/settings-api/)
- [Options API](https://developer.wordpress.org/plugins/settings/options-api/)
- [Shortcode API](https://developer.wordpress.org/plugins/shortcodes/)
- [Plugin Security](https://developer.wordpress.org/plugins/security/)
- [Plugin Internationalization](https://developer.wordpress.org/plugins/internationalization/)

---

**Remember**: A well-built plugin is secure, performant, and follows WordPress standards. Always prioritize user security and data integrity.

