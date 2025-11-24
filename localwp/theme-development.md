# WordPress Theme Development Rules

## Purpose

Guidelines and best practices for building custom WordPress themes. This document covers theme structure, template hierarchy, modern theme development including block themes, and best practices for creating maintainable, performant themes.

## Key Principles

### 1. Theme Structure

Every WordPress theme must have at minimum:
- `style.css` - Theme stylesheet with header information
- `index.php` - Main template file

### 2. Template Hierarchy

Understand and leverage WordPress's template hierarchy for proper template organization.

### 3. Modern Standards

- Support block editor (Gutenberg)
- Implement theme.json for global styles
- Follow accessibility guidelines (WCAG)
- Ensure responsive design
- Optimize for performance

### 4. Child Themes

Use child themes for customizations of existing themes to maintain updateability.

## Theme File Structure

### Basic Theme Structure

```
theme-name/
├── style.css              # Main stylesheet (REQUIRED)
├── index.php              # Main template (REQUIRED)
├── functions.php          # Theme functions and setup
├── screenshot.png         # Theme preview (880x660px recommended)
├── header.php             # Header template
├── footer.php             # Footer template
├── sidebar.php            # Sidebar template
├── single.php             # Single post template
├── page.php               # Page template
├── archive.php            # Archive template
├── 404.php                # 404 error template
├── search.php             # Search results template
├── comments.php           # Comments template
├── template-parts/        # Reusable template parts
│   ├── content.php
│   ├── content-single.php
│   └── navigation.php
├── inc/                   # Additional PHP includes
│   ├── customizer.php
│   ├── custom-functions.php
│   └── template-tags.php
├── assets/                # Theme assets
│   ├── css/
│   │   └── custom.css
│   ├── js/
│   │   └── custom.js
│   └── images/
├── languages/             # Translation files
│   └── theme-name.pot
└── theme.json             # Block theme configuration
```

### Block Theme Structure

```
block-theme/
├── style.css
├── theme.json             # Global styles and settings
├── functions.php
├── templates/             # Block templates
│   ├── index.html
│   ├── single.html
│   ├── page.html
│   └── archive.html
├── parts/                 # Block template parts
│   ├── header.html
│   ├── footer.html
│   └── sidebar.html
├── patterns/              # Block patterns
│   └── hero.php
└── assets/
```

## Common Patterns

### style.css Header

```css
/*
Theme Name: My Custom Theme
Theme URI: https://example.com/my-theme
Author: Your Name
Author URI: https://example.com
Description: A brief description of the theme
Version: 1.0.0
Requires at least: 6.0
Tested up to: 6.4
Requires PHP: 7.4
License: GNU General Public License v2 or later
License URI: http://www.gnu.org/licenses/gpl-2.0.html
Text Domain: my-custom-theme
Domain Path: /languages
Tags: one-column, two-columns, right-sidebar, custom-menu, featured-images
*/
```

### functions.php Setup

```php
<?php
/**
 * Theme setup and initialization
 */

if ( ! defined( 'ABSPATH' ) ) {
    exit; // Exit if accessed directly
}

/**
 * Sets up theme defaults and registers support for various WordPress features.
 */
function mytheme_setup() {
    // Add default posts and comments RSS feed links to head
    add_theme_support( 'automatic-feed-links' );

    // Let WordPress manage the document title
    add_theme_support( 'title-tag' );

    // Enable support for Post Thumbnails
    add_theme_support( 'post-thumbnails' );
    set_post_thumbnail_size( 1200, 675, true );

    // Add custom image sizes
    add_image_size( 'mytheme-featured', 800, 450, true );
    add_image_size( 'mytheme-thumbnail', 400, 225, true );

    // Register navigation menus
    register_nav_menus( array(
        'primary' => esc_html__( 'Primary Menu', 'my-custom-theme' ),
        'footer'  => esc_html__( 'Footer Menu', 'my-custom-theme' ),
    ) );

    // Switch default core markup to output valid HTML5
    add_theme_support( 'html5', array(
        'search-form',
        'comment-form',
        'comment-list',
        'gallery',
        'caption',
        'style',
        'script',
    ) );

    // Add theme support for selective refresh for widgets
    add_theme_support( 'customize-selective-refresh-widgets' );

    // Add support for Block Styles
    add_theme_support( 'wp-block-styles' );

    // Add support for full and wide align images
    add_theme_support( 'align-wide' );

    // Add support for editor styles
    add_theme_support( 'editor-styles' );
    add_editor_style( 'assets/css/editor-style.css' );

    // Add support for responsive embeds
    add_theme_support( 'responsive-embeds' );

    // Add support for custom logo
    add_theme_support( 'custom-logo', array(
        'height'      => 100,
        'width'       => 400,
        'flex-height' => true,
        'flex-width'  => true,
    ) );

    // Add support for custom background
    add_theme_support( 'custom-background', array(
        'default-color' => 'ffffff',
    ) );

    // Load text domain for translations
    load_theme_textdomain( 'my-custom-theme', get_template_directory() . '/languages' );
}
add_action( 'after_setup_theme', 'mytheme_setup' );

/**
 * Set the content width
 */
if ( ! isset( $content_width ) ) {
    $content_width = 1200;
}

/**
 * Register widget areas
 */
function mytheme_widgets_init() {
    register_sidebar( array(
        'name'          => esc_html__( 'Sidebar', 'my-custom-theme' ),
        'id'            => 'sidebar-1',
        'description'   => esc_html__( 'Add widgets here.', 'my-custom-theme' ),
        'before_widget' => '<section id="%1$s" class="widget %2$s">',
        'after_widget'  => '</section>',
        'before_title'  => '<h2 class="widget-title">',
        'after_title'   => '</h2>',
    ) );

    register_sidebar( array(
        'name'          => esc_html__( 'Footer', 'my-custom-theme' ),
        'id'            => 'footer-1',
        'description'   => esc_html__( 'Add footer widgets here.', 'my-custom-theme' ),
        'before_widget' => '<div id="%1$s" class="footer-widget %2$s">',
        'after_widget'  => '</div>',
        'before_title'  => '<h3 class="widget-title">',
        'after_title'   => '</h3>',
    ) );
}
add_action( 'widgets_init', 'mytheme_widgets_init' );

/**
 * Enqueue scripts and styles
 */
function mytheme_scripts() {
    // Main stylesheet
    wp_enqueue_style( 
        'mytheme-style', 
        get_stylesheet_uri(), 
        array(), 
        wp_get_theme()->get( 'Version' ) 
    );

    // Custom stylesheet
    wp_enqueue_style( 
        'mytheme-custom', 
        get_template_directory_uri() . '/assets/css/custom.css', 
        array( 'mytheme-style' ), 
        wp_get_theme()->get( 'Version' ) 
    );

    // Main JavaScript
    wp_enqueue_script( 
        'mytheme-script', 
        get_template_directory_uri() . '/assets/js/custom.js', 
        array( 'jquery' ), 
        wp_get_theme()->get( 'Version' ), 
        true 
    );

    // Localize script (pass PHP data to JavaScript)
    wp_localize_script( 'mytheme-script', 'mythemeData', array(
        'ajaxUrl' => admin_url( 'admin-ajax.php' ),
        'nonce'   => wp_create_nonce( 'mytheme-nonce' ),
    ) );

    // Comments reply script
    if ( is_singular() && comments_open() && get_option( 'thread_comments' ) ) {
        wp_enqueue_script( 'comment-reply' );
    }
}
add_action( 'wp_enqueue_scripts', 'mytheme_scripts' );
```

### Template Hierarchy

WordPress follows a specific template hierarchy when displaying content:

```
Single Post:
single-{post-type}-{slug}.php → single-{post-type}.php → single.php → singular.php → index.php

Page:
{custom-template}.php → page-{slug}.php → page-{id}.php → page.php → singular.php → index.php

Archive:
archive-{post-type}.php → archive.php → index.php

Category:
category-{slug}.php → category-{id}.php → category.php → archive.php → index.php

Tag:
tag-{slug}.php → tag-{id}.php → tag.php → archive.php → index.php

Custom Taxonomy:
taxonomy-{taxonomy}-{term}.php → taxonomy-{taxonomy}.php → taxonomy.php → archive.php → index.php

Author:
author-{nicename}.php → author-{id}.php → author.php → archive.php → index.php

Date:
date.php → archive.php → index.php

Search:
search.php → index.php

404:
404.php → index.php

Front Page:
front-page.php → home.php → index.php

Home (Blog):
home.php → index.php
```

### Header Template (header.php)

```php
<!DOCTYPE html>
<html <?php language_attributes(); ?>>
<head>
    <meta charset="<?php bloginfo( 'charset' ); ?>">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <?php wp_head(); ?>
</head>
<body <?php body_class(); ?>>
<?php wp_body_open(); ?>

<div id="page" class="site">
    <a class="skip-link screen-reader-text" href="#primary">
        <?php esc_html_e( 'Skip to content', 'my-custom-theme' ); ?>
    </a>

    <header id="masthead" class="site-header">
        <div class="site-branding">
            <?php
            if ( has_custom_logo() ) {
                the_custom_logo();
            } else {
                ?>
                <h1 class="site-title">
                    <a href="<?php echo esc_url( home_url( '/' ) ); ?>">
                        <?php bloginfo( 'name' ); ?>
                    </a>
                </h1>
                <p class="site-description"><?php bloginfo( 'description' ); ?></p>
                <?php
            }
            ?>
        </div>

        <nav id="site-navigation" class="main-navigation">
            <button class="menu-toggle" aria-controls="primary-menu" aria-expanded="false">
                <?php esc_html_e( 'Menu', 'my-custom-theme' ); ?>
            </button>
            <?php
            wp_nav_menu( array(
                'theme_location' => 'primary',
                'menu_id'        => 'primary-menu',
                'container'      => false,
                'fallback_cb'    => false,
            ) );
            ?>
        </nav>
    </header>

    <div id="content" class="site-content">
```

### Footer Template (footer.php)

```php
    </div><!-- #content -->

    <footer id="colophon" class="site-footer">
        <?php if ( is_active_sidebar( 'footer-1' ) ) : ?>
            <div class="footer-widgets">
                <?php dynamic_sidebar( 'footer-1' ); ?>
            </div>
        <?php endif; ?>

        <div class="site-info">
            <p>
                <?php
                printf(
                    esc_html__( '© %1$s %2$s. All rights reserved.', 'my-custom-theme' ),
                    date( 'Y' ),
                    get_bloginfo( 'name' )
                );
                ?>
            </p>
        </div>
    </footer>
</div><!-- #page -->

<?php wp_footer(); ?>
</body>
</html>
```

### Index Template (index.php)

```php
<?php get_header(); ?>

<main id="primary" class="site-main">
    <?php
    if ( have_posts() ) :
        
        // Check if it's the blog page
        if ( is_home() && ! is_front_page() ) :
            ?>
            <header class="page-header">
                <h1 class="page-title"><?php single_post_title(); ?></h1>
            </header>
            <?php
        endif;

        // Start the Loop
        while ( have_posts() ) :
            the_post();
            
            // Include the template part for the content
            get_template_part( 'template-parts/content', get_post_type() );

        endwhile;

        // Pagination
        the_posts_pagination( array(
            'prev_text' => esc_html__( 'Previous', 'my-custom-theme' ),
            'next_text' => esc_html__( 'Next', 'my-custom-theme' ),
        ) );

    else :
        
        // If no content, include the "No posts found" template
        get_template_part( 'template-parts/content', 'none' );

    endif;
    ?>
</main>

<?php
get_sidebar();
get_footer();
```

### Single Post Template (single.php)

```php
<?php get_header(); ?>

<main id="primary" class="site-main">
    <?php
    while ( have_posts() ) :
        the_post();
        ?>
        
        <article id="post-<?php the_ID(); ?>" <?php post_class(); ?>>
            <header class="entry-header">
                <?php
                if ( is_singular() ) :
                    the_title( '<h1 class="entry-title">', '</h1>' );
                else :
                    the_title( '<h2 class="entry-title"><a href="' . esc_url( get_permalink() ) . '">', '</a></h2>' );
                endif;
                ?>

                <div class="entry-meta">
                    <?php
                    printf(
                        '<span class="posted-on">%s</span>',
                        get_the_date()
                    );
                    
                    printf(
                        '<span class="byline"> by %s</span>',
                        '<span class="author vcard"><a href="' . esc_url( get_author_posts_url( get_the_author_meta( 'ID' ) ) ) . '">' . esc_html( get_the_author() ) . '</a></span>'
                    );
                    ?>
                </div>
            </header>

            <?php if ( has_post_thumbnail() ) : ?>
                <div class="post-thumbnail">
                    <?php the_post_thumbnail( 'large' ); ?>
                </div>
            <?php endif; ?>

            <div class="entry-content">
                <?php
                the_content();

                wp_link_pages( array(
                    'before' => '<div class="page-links">' . esc_html__( 'Pages:', 'my-custom-theme' ),
                    'after'  => '</div>',
                ) );
                ?>
            </div>

            <footer class="entry-footer">
                <?php
                $categories_list = get_the_category_list( ', ' );
                if ( $categories_list ) {
                    printf( '<span class="cat-links">%s</span>', $categories_list );
                }

                $tags_list = get_the_tag_list( '', ', ' );
                if ( $tags_list ) {
                    printf( '<span class="tags-links">%s</span>', $tags_list );
                }
                ?>
            </footer>
        </article>

        <?php
        // If comments are open or there are comments, load the comments template
        if ( comments_open() || get_comments_number() ) :
            comments_template();
        endif;

        // Previous/next post navigation
        the_post_navigation( array(
            'prev_text' => '<span class="nav-subtitle">' . esc_html__( 'Previous:', 'my-custom-theme' ) . '</span> <span class="nav-title">%title</span>',
            'next_text' => '<span class="nav-subtitle">' . esc_html__( 'Next:', 'my-custom-theme' ) . '</span> <span class="nav-title">%title</span>',
        ) );

    endwhile;
    ?>
</main>

<?php
get_sidebar();
get_footer();
```

### Template Parts (template-parts/content.php)

```php
<article id="post-<?php the_ID(); ?>" <?php post_class(); ?>>
    <header class="entry-header">
        <?php
        the_title( sprintf( '<h2 class="entry-title"><a href="%s">', esc_url( get_permalink() ) ), '</a></h2>' );
        ?>
        
        <?php if ( 'post' === get_post_type() ) : ?>
            <div class="entry-meta">
                <span class="posted-on"><?php echo get_the_date(); ?></span>
                <span class="byline">by <?php the_author(); ?></span>
            </div>
        <?php endif; ?>
    </header>

    <?php if ( has_post_thumbnail() ) : ?>
        <div class="post-thumbnail">
            <a href="<?php the_permalink(); ?>">
                <?php the_post_thumbnail( 'medium' ); ?>
            </a>
        </div>
    <?php endif; ?>

    <div class="entry-summary">
        <?php the_excerpt(); ?>
    </div>

    <footer class="entry-footer">
        <a href="<?php the_permalink(); ?>" class="read-more">
            <?php esc_html_e( 'Read More', 'my-custom-theme' ); ?>
        </a>
    </footer>
</article>
```

### Custom Page Templates

```php
<?php
/**
 * Template Name: Full Width
 * Template Post Type: page, post
 * 
 * @package My_Custom_Theme
 */

get_header();
?>

<main id="primary" class="site-main full-width">
    <?php
    while ( have_posts() ) :
        the_post();
        get_template_part( 'template-parts/content', 'page' );
        
        if ( comments_open() || get_comments_number() ) :
            comments_template();
        endif;
    endwhile;
    ?>
</main>

<?php
get_footer();
```

### theme.json (Block Themes)

```json
{
    "$schema": "https://schemas.wp.org/trunk/theme.json",
    "version": 2,
    "settings": {
        "color": {
            "palette": [
                {
                    "slug": "primary",
                    "color": "#007cba",
                    "name": "Primary"
                },
                {
                    "slug": "secondary",
                    "color": "#006ba1",
                    "name": "Secondary"
                },
                {
                    "slug": "foreground",
                    "color": "#333333",
                    "name": "Foreground"
                },
                {
                    "slug": "background",
                    "color": "#ffffff",
                    "name": "Background"
                }
            ]
        },
        "typography": {
            "fontFamilies": [
                {
                    "fontFamily": "-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen-Sans, Ubuntu, Cantarell, 'Helvetica Neue', sans-serif",
                    "slug": "system-font",
                    "name": "System Font"
                }
            ],
            "fontSizes": [
                {
                    "slug": "small",
                    "size": "0.875rem",
                    "name": "Small"
                },
                {
                    "slug": "medium",
                    "size": "1rem",
                    "name": "Medium"
                },
                {
                    "slug": "large",
                    "size": "1.5rem",
                    "name": "Large"
                },
                {
                    "slug": "x-large",
                    "size": "2rem",
                    "name": "Extra Large"
                }
            ]
        },
        "layout": {
            "contentSize": "800px",
            "wideSize": "1200px"
        }
    },
    "styles": {
        "color": {
            "background": "var(--wp--preset--color--background)",
            "text": "var(--wp--preset--color--foreground)"
        },
        "typography": {
            "fontFamily": "var(--wp--preset--font-family--system-font)",
            "fontSize": "var(--wp--preset--font-size--medium)",
            "lineHeight": "1.6"
        },
        "elements": {
            "link": {
                "color": {
                    "text": "var(--wp--preset--color--primary)"
                }
            },
            "h1": {
                "typography": {
                    "fontSize": "var(--wp--preset--font-size--x-large)"
                }
            }
        }
    }
}
```

## Best Practices

### DO's

1. **Use get_template_part()**: Break templates into reusable parts
2. **Enqueue Assets Properly**: Use wp_enqueue_script() and wp_enqueue_style()
3. **Add Theme Support**: Enable WordPress features your theme supports
4. **Make it Translatable**: Use translation functions for all text
5. **Use Template Tags**: Leverage WordPress template tags (the_title(), the_content(), etc.)
6. **Implement Accessibility**: Follow WCAG guidelines
7. **Create Child Themes**: For modifications of existing themes
8. **Use Proper Escaping**: Escape all output with esc_html(), esc_attr(), etc.
9. **Add body_class()**: Include body_class() in body tag
10. **Include wp_head() and wp_footer()**: Required for plugins to work
11. **Test Responsiveness**: Ensure theme works on all devices
12. **Add Schema Markup**: Implement structured data
13. **Optimize Images**: Use responsive images and lazy loading
14. **Version Assets**: Use theme version for cache busting
15. **Document Your Code**: Add comments explaining functionality

### DON'Ts

1. **Don't Hardcode URLs**: Use WordPress functions
2. **Don't Include jQuery from CDN**: Use WordPress's bundled jQuery
3. **Don't Modify Parent Themes**: Use child themes instead
4. **Don't Use Inline Styles/Scripts**: Enqueue them properly
5. **Don't Forget wp_body_open()**: Required since WordPress 5.2
6. **Don't Use Deprecated Functions**: Check WordPress documentation
7. **Don't Hardcode Menus**: Use wp_nav_menu() instead
8. **Don't Echo Unsanitized Data**: Always escape output
9. **Don't Load All Fonts**: Only load fonts you actually use
10. **Don't Use !important in CSS**: Write more specific selectors
11. **Don't Forget wp_reset_postdata()**: Reset after custom queries
12. **Don't Add Functionality**: Keep business logic in plugins
13. **Don't Hardcode Sizes**: Use add_image_size() for thumbnails
14. **Don't Ignore Sidebar Management**: Register sidebars properly
15. **Don't Skip Testing**: Test on different browsers and devices

## Cursor-Specific Instructions

When prompting Cursor AI for theme development:

```
"Create a WordPress theme [describe theme]. Follow WordPress theme standards,
include proper template hierarchy, enqueue assets correctly, make it fully
translatable, and ensure accessibility compliance."

"Build a [page/template] for WordPress theme. Use proper WordPress template
tags, include necessary hooks (wp_head, wp_footer), escape all output, and
follow WordPress coding standards."

"Create a theme.json configuration for a block theme with [describe styling].
Include color palette, typography settings, and layout configurations."
```

## Common Pitfalls

1. **Not calling wp_head() and wp_footer()**: Required for WordPress to work properly
2. **Hardcoding site URLs**: Use home_url(), get_template_directory_uri()
3. **Not making text translatable**: Wrap all strings in translation functions
4. **Enqueueing scripts incorrectly**: Don't add scripts directly in templates
5. **Not using body_class() and post_class()**: These provide useful CSS classes
6. **Forgetting wp_body_open()**: Modern WordPress requirement
7. **Not escaping output**: Security vulnerability
8. **Ignoring template hierarchy**: WordPress has a specific order for templates
9. **Not registering navigation menus**: Always register menus programmatically
10. **Hardcoding widget areas**: Use register_sidebar() and dynamic_sidebar()

## Resources

- [Theme Handbook](https://developer.wordpress.org/themes/)
- [Template Hierarchy](https://developer.wordpress.org/themes/basics/template-hierarchy/)
- [Theme Functions](https://developer.wordpress.org/themes/basics/theme-functions/)
- [Block Themes](https://developer.wordpress.org/block-editor/how-to-guides/themes/block-theme-overview/)
- [theme.json Reference](https://developer.wordpress.org/block-editor/reference-guides/theme-json-reference/)
- [Accessibility Handbook](https://make.wordpress.org/accessibility/handbook/)

---

**Remember**: A great theme is accessible, performant, and follows WordPress standards. Always test your theme thoroughly before deployment.

