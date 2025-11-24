# Advanced Custom Fields (ACF) Integration Rules

## Purpose

Comprehensive guidelines for using Advanced Custom Fields (ACF) in WordPress development. This document covers field group creation, displaying fields, ACF blocks, flexible content, and best practices.

## Key Principles

### 1. Field Groups Organization

Organize field groups logically by:
- Content type (posts, pages, custom post types)
- Template (specific page templates)
- Functionality (header options, footer options, etc.)

### 2. Consistent Naming

Use consistent, descriptive field names:
- Use underscores for field names
- Use lowercase
- Be descriptive but concise
- Group related fields with prefixes

### 3. Conditional Logic

Leverage ACF's conditional logic to show/hide fields based on other field values.

### 4. Security

Always sanitize and escape ACF field output, even though ACF does some sanitization by default.

## Common Patterns

### Getting Field Values

```php
<?php
// Get field from current post
$value = get_field( 'field_name' );

// Get field from specific post
$value = get_field( 'field_name', $post_id );

// Get field from options page
$value = get_field( 'field_name', 'option' );

// Get field from term
$value = get_field( 'field_name', 'term_' . $term_id );

// Get field from user
$value = get_field( 'field_name', 'user_' . $user_id );

// Get field object (includes metadata)
$field_object = get_field_object( 'field_name' );

// Check if field has value
if ( get_field( 'field_name' ) ) {
    // Field has value
}
```

### Displaying Different Field Types

#### Text Field

```php
<?php
$text = get_field( 'text_field' );
if ( $text ) :
    ?>
    <p><?php echo esc_html( $text ); ?></p>
    <?php
endif;
```

#### Textarea Field

```php
<?php
$textarea = get_field( 'textarea_field' );
if ( $textarea ) :
    ?>
    <div class="content">
        <?php echo wp_kses_post( wpautop( $textarea ) ); ?>
    </div>
    <?php
endif;
```

#### WYSIWYG Editor

```php
<?php
$content = get_field( 'wysiwyg_field' );
if ( $content ) :
    ?>
    <div class="content">
        <?php echo wp_kses_post( $content ); ?>
    </div>
    <?php
endif;
```

#### Image Field

```php
<?php
// Return format: Image Array
$image = get_field( 'image_field' );
if ( $image ) :
    ?>
    <img src="<?php echo esc_url( $image['url'] ); ?>" 
         alt="<?php echo esc_attr( $image['alt'] ); ?>" 
         width="<?php echo esc_attr( $image['width'] ); ?>"
         height="<?php echo esc_attr( $image['height'] ); ?>" />
    
    <!-- Responsive image with srcset -->
    <img src="<?php echo esc_url( $image['url'] ); ?>" 
         srcset="<?php echo esc_attr( $image['sizes']['medium'] ); ?> 768w,
                 <?php echo esc_url( $image['url'] ); ?> 1024w"
         sizes="(max-width: 768px) 100vw, 768px"
         alt="<?php echo esc_attr( $image['alt'] ); ?>" />
    <?php
endif;

// Return format: Image ID
$image_id = get_field( 'image_field' );
if ( $image_id ) :
    echo wp_get_attachment_image( $image_id, 'large', false, array(
        'class' => 'custom-class',
        'alt'   => get_post_meta( $image_id, '_wp_attachment_image_alt', true ),
    ) );
endif;

// Return format: Image URL
$image_url = get_field( 'image_field' );
if ( $image_url ) :
    ?>
    <img src="<?php echo esc_url( $image_url ); ?>" alt="" />
    <?php
endif;
```

#### File Field

```php
<?php
$file = get_field( 'file_field' );
if ( $file ) :
    ?>
    <a href="<?php echo esc_url( $file['url'] ); ?>" 
       download="<?php echo esc_attr( $file['filename'] ); ?>">
        <?php echo esc_html( $file['title'] ); ?>
        (<?php echo esc_html( $file['filesize'] ); ?>)
    </a>
    <?php
endif;
```

#### Gallery Field

```php
<?php
$gallery = get_field( 'gallery_field' );
if ( $gallery ) :
    ?>
    <div class="gallery">
        <?php foreach ( $gallery as $image ) : ?>
            <div class="gallery-item">
                <a href="<?php echo esc_url( $image['url'] ); ?>" 
                   data-lightbox="gallery">
                    <img src="<?php echo esc_url( $image['sizes']['thumbnail'] ); ?>" 
                         alt="<?php echo esc_attr( $image['alt'] ); ?>" />
                </a>
            </div>
        <?php endforeach; ?>
    </div>
    <?php
endif;
```

#### True/False Field

```php
<?php
$enabled = get_field( 'enable_feature' );
if ( $enabled ) :
    ?>
    <div class="feature-enabled">
        <!-- Show content when enabled -->
    </div>
    <?php
endif;
```

#### Select Field

```php
<?php
$selection = get_field( 'select_field' );
if ( $selection ) :
    ?>
    <p>Selected option: <?php echo esc_html( $selection ); ?></p>
    <?php
endif;

// Multiple select (returns array)
$selections = get_field( 'multi_select_field' );
if ( $selections ) :
    ?>
    <ul>
        <?php foreach ( $selections as $selection ) : ?>
            <li><?php echo esc_html( $selection ); ?></li>
        <?php endforeach; ?>
    </ul>
    <?php
endif;
```

#### Radio Button Field

```php
<?php
$choice = get_field( 'radio_field' );
if ( $choice ) :
    ?>
    <p>Selected: <?php echo esc_html( $choice ); ?></p>
    <?php
endif;

// With different layout based on choice
$layout = get_field( 'layout_choice' );
switch ( $layout ) {
    case 'full-width':
        // Full width layout
        break;
    case 'sidebar':
        // Sidebar layout
        break;
    default:
        // Default layout
        break;
}
```

#### Checkbox Field

```php
<?php
$options = get_field( 'checkbox_field' );
if ( $options ) :
    ?>
    <ul>
        <?php foreach ( $options as $option ) : ?>
            <li><?php echo esc_html( $option ); ?></li>
        <?php endforeach; ?>
    </ul>
    <?php
endif;
```

#### Relationship Field

```php
<?php
$posts = get_field( 'related_posts' );
if ( $posts ) :
    ?>
    <div class="related-posts">
        <?php foreach ( $posts as $post ) : ?>
            <?php setup_postdata( $post ); ?>
            <article>
                <h3><a href="<?php the_permalink(); ?>"><?php the_title(); ?></a></h3>
                <?php the_excerpt(); ?>
            </article>
        <?php endforeach; ?>
        <?php wp_reset_postdata(); ?>
    </div>
    <?php
endif;
```

#### Post Object Field

```php
<?php
$post_object = get_field( 'post_object_field' );
if ( $post_object ) :
    $post = $post_object;
    setup_postdata( $post );
    ?>
    <article>
        <h2><a href="<?php the_permalink(); ?>"><?php the_title(); ?></a></h2>
        <?php the_excerpt(); ?>
    </article>
    <?php
    wp_reset_postdata();
endif;
```

#### Page Link Field

```php
<?php
$link = get_field( 'page_link_field' );
if ( $link ) :
    ?>
    <a href="<?php echo esc_url( $link ); ?>">Visit Page</a>
    <?php
endif;
```

#### Link Field

```php
<?php
$link = get_field( 'link_field' );
if ( $link ) :
    $link_url = $link['url'];
    $link_title = $link['title'];
    $link_target = $link['target'] ? $link['target'] : '_self';
    ?>
    <a href="<?php echo esc_url( $link_url ); ?>" 
       target="<?php echo esc_attr( $link_target ); ?>">
        <?php echo esc_html( $link_title ); ?>
    </a>
    <?php
endif;
```

#### Taxonomy Field

```php
<?php
$terms = get_field( 'taxonomy_field' );
if ( $terms ) :
    ?>
    <ul class="terms">
        <?php foreach ( $terms as $term ) : ?>
            <li>
                <a href="<?php echo esc_url( get_term_link( $term ) ); ?>">
                    <?php echo esc_html( $term->name ); ?>
                </a>
            </li>
        <?php endforeach; ?>
    </ul>
    <?php
endif;
```

#### User Field

```php
<?php
$user = get_field( 'user_field' );
if ( $user ) :
    ?>
    <div class="author">
        <?php echo get_avatar( $user['ID'], 64 ); ?>
        <h3><?php echo esc_html( $user['display_name'] ); ?></h3>
        <p><?php echo esc_html( $user['user_email'] ); ?></p>
    </div>
    <?php
endif;
```

#### Google Map Field

```php
<?php
$location = get_field( 'map_field' );
if ( $location ) :
    ?>
    <div class="acf-map" data-zoom="16">
        <div class="marker" 
             data-lat="<?php echo esc_attr( $location['lat'] ); ?>" 
             data-lng="<?php echo esc_attr( $location['lng'] ); ?>">
            <h3><?php echo esc_html( $location['address'] ); ?></h3>
        </div>
    </div>
    <?php
endif;
```

#### Date Picker Field

```php
<?php
$date = get_field( 'date_field' );
if ( $date ) :
    $date_object = DateTime::createFromFormat( 'Ymd', $date );
    ?>
    <time datetime="<?php echo esc_attr( $date_object->format( 'c' ) ); ?>">
        <?php echo esc_html( $date_object->format( 'F j, Y' ) ); ?>
    </time>
    <?php
endif;
```

#### Color Picker Field

```php
<?php
$color = get_field( 'color_field' );
if ( $color ) :
    ?>
    <div style="background-color: <?php echo esc_attr( $color ); ?>;">
        <!-- Content with background color -->
    </div>
    <?php
endif;
```

### Repeater Field

```php
<?php
if ( have_rows( 'repeater_field' ) ) :
    ?>
    <div class="repeater-items">
        <?php while ( have_rows( 'repeater_field' ) ) : the_row(); ?>
            <div class="item">
                <h3><?php echo esc_html( get_sub_field( 'title' ) ); ?></h3>
                <p><?php echo esc_html( get_sub_field( 'description' ) ); ?></p>
                <?php
                $image = get_sub_field( 'image' );
                if ( $image ) :
                    ?>
                    <img src="<?php echo esc_url( $image['url'] ); ?>" 
                         alt="<?php echo esc_attr( $image['alt'] ); ?>" />
                    <?php
                endif;
                ?>
            </div>
        <?php endwhile; ?>
    </div>
    <?php
endif;

// Nested repeater
if ( have_rows( 'parent_repeater' ) ) :
    while ( have_rows( 'parent_repeater' ) ) : the_row();
        echo '<div class="parent-item">';
        echo '<h2>' . esc_html( get_sub_field( 'parent_title' ) ) . '</h2>';
        
        if ( have_rows( 'child_repeater' ) ) :
            echo '<div class="child-items">';
            while ( have_rows( 'child_repeater' ) ) : the_row();
                echo '<div class="child-item">';
                echo '<h3>' . esc_html( get_sub_field( 'child_title' ) ) . '</h3>';
                echo '</div>';
            endwhile;
            echo '</div>';
        endif;
        
        echo '</div>';
    endwhile;
endif;
```

### Flexible Content Field

```php
<?php
if ( have_rows( 'flexible_content_field' ) ) :
    while ( have_rows( 'flexible_content_field' ) ) : the_row();
        
        if ( get_row_layout() == 'text_block' ) :
            ?>
            <section class="text-block">
                <h2><?php echo esc_html( get_sub_field( 'heading' ) ); ?></h2>
                <div class="content">
                    <?php echo wp_kses_post( get_sub_field( 'content' ) ); ?>
                </div>
            </section>
            <?php
        
        elseif ( get_row_layout() == 'image_block' ) :
            $image = get_sub_field( 'image' );
            ?>
            <section class="image-block">
                <?php if ( $image ) : ?>
                    <img src="<?php echo esc_url( $image['url'] ); ?>" 
                         alt="<?php echo esc_attr( $image['alt'] ); ?>" />
                <?php endif; ?>
                <?php if ( get_sub_field( 'caption' ) ) : ?>
                    <p class="caption"><?php echo esc_html( get_sub_field( 'caption' ) ); ?></p>
                <?php endif; ?>
            </section>
            <?php
        
        elseif ( get_row_layout() == 'gallery_block' ) :
            ?>
            <section class="gallery-block">
                <?php if ( have_rows( 'images' ) ) : ?>
                    <div class="gallery">
                        <?php while ( have_rows( 'images' ) ) : the_row(); ?>
                            <?php $img = get_sub_field( 'image' ); ?>
                            <div class="gallery-item">
                                <img src="<?php echo esc_url( $img['sizes']['medium'] ); ?>" 
                                     alt="<?php echo esc_attr( $img['alt'] ); ?>" />
                            </div>
                        <?php endwhile; ?>
                    </div>
                <?php endif; ?>
            </section>
            <?php
        
        endif;
        
    endwhile;
endif;
```

### Group Field

```php
<?php
$group = get_field( 'group_field' );
if ( $group ) :
    ?>
    <div class="group-content">
        <h2><?php echo esc_html( $group['title'] ); ?></h2>
        <p><?php echo esc_html( $group['description'] ); ?></p>
        <?php if ( $group['image'] ) : ?>
            <img src="<?php echo esc_url( $group['image']['url'] ); ?>" 
                 alt="<?php echo esc_attr( $group['image']['alt'] ); ?>" />
        <?php endif; ?>
    </div>
    <?php
endif;

// Alternative syntax
if ( have_rows( 'group_field' ) ) :
    while ( have_rows( 'group_field' ) ) : the_row();
        ?>
        <h2><?php echo esc_html( get_sub_field( 'title' ) ); ?></h2>
        <p><?php echo esc_html( get_sub_field( 'description' ) ); ?></p>
        <?php
    endwhile;
endif;
```

### Options Page

```php
<?php
// Register options page in functions.php
if ( function_exists( 'acf_add_options_page' ) ) {
    
    // Main options page
    acf_add_options_page( array(
        'page_title' => 'Theme Settings',
        'menu_title' => 'Theme Settings',
        'menu_slug'  => 'theme-settings',
        'capability' => 'manage_options',
        'icon_url'   => 'dashicons-admin-generic',
        'position'   => 60,
    ) );
    
    // Sub page
    acf_add_options_sub_page( array(
        'page_title'  => 'Header Settings',
        'menu_title'  => 'Header',
        'parent_slug' => 'theme-settings',
    ) );
    
    acf_add_options_sub_page( array(
        'page_title'  => 'Footer Settings',
        'menu_title'  => 'Footer',
        'parent_slug' => 'theme-settings',
    ) );
}

// Get options page fields
$header_logo = get_field( 'header_logo', 'option' );
$footer_text = get_field( 'footer_text', 'option' );
$social_links = get_field( 'social_links', 'option' );

// Display in template
if ( $header_logo ) :
    ?>
    <img src="<?php echo esc_url( $header_logo['url'] ); ?>" 
         alt="<?php echo esc_attr( get_bloginfo( 'name' ) ); ?>" />
    <?php
endif;
```

### ACF Blocks

```php
<?php
/**
 * Register ACF Block in functions.php
 */
function register_acf_blocks() {
    if ( function_exists( 'acf_register_block_type' ) ) {
        
        acf_register_block_type( array(
            'name'            => 'testimonial',
            'title'           => __( 'Testimonial', 'my-theme' ),
            'description'     => __( 'A custom testimonial block.', 'my-theme' ),
            'render_template' => 'blocks/testimonial.php',
            'category'        => 'formatting',
            'icon'            => 'admin-comments',
            'keywords'        => array( 'testimonial', 'quote' ),
            'mode'            => 'preview',
            'supports'        => array(
                'align'  => true,
                'mode'   => false,
                'jsx'    => true,
            ),
        ) );
        
        acf_register_block_type( array(
            'name'            => 'hero',
            'title'           => __( 'Hero Section', 'my-theme' ),
            'description'     => __( 'A hero section with image and text.', 'my-theme' ),
            'render_callback' => 'render_hero_block',
            'category'        => 'layout',
            'icon'            => 'cover-image',
            'keywords'        => array( 'hero', 'banner' ),
        ) );
    }
}
add_action( 'acf/init', 'register_acf_blocks' );

/**
 * Block template file: blocks/testimonial.php
 */
$quote = get_field( 'quote' );
$author = get_field( 'author' );
$role = get_field( 'role' );
$image = get_field( 'author_image' );

// Create unique ID for block
$block_id = 'testimonial-' . $block['id'];
if ( ! empty( $block['anchor'] ) ) {
    $block_id = $block['anchor'];
}

// Create class attribute
$class_name = 'testimonial-block';
if ( ! empty( $block['className'] ) ) {
    $class_name .= ' ' . $block['className'];
}
if ( ! empty( $block['align'] ) ) {
    $class_name .= ' align' . $block['align'];
}
?>

<div id="<?php echo esc_attr( $block_id ); ?>" class="<?php echo esc_attr( $class_name ); ?>">
    <?php if ( $quote ) : ?>
        <blockquote class="testimonial-quote">
            <?php echo wp_kses_post( $quote ); ?>
        </blockquote>
    <?php endif; ?>
    
    <div class="testimonial-author">
        <?php if ( $image ) : ?>
            <img src="<?php echo esc_url( $image['sizes']['thumbnail'] ); ?>" 
                 alt="<?php echo esc_attr( $author ); ?>" 
                 class="author-image" />
        <?php endif; ?>
        <div class="author-info">
            <?php if ( $author ) : ?>
                <p class="author-name"><?php echo esc_html( $author ); ?></p>
            <?php endif; ?>
            <?php if ( $role ) : ?>
                <p class="author-role"><?php echo esc_html( $role ); ?></p>
            <?php endif; ?>
        </div>
    </div>
</div>

<?php
/**
 * Block render callback function
 */
function render_hero_block( $block, $content = '', $is_preview = false ) {
    $heading = get_field( 'heading' );
    $subheading = get_field( 'subheading' );
    $button_text = get_field( 'button_text' );
    $button_link = get_field( 'button_link' );
    $background_image = get_field( 'background_image' );
    
    $block_id = 'hero-' . $block['id'];
    $class_name = 'hero-block';
    if ( ! empty( $block['className'] ) ) {
        $class_name .= ' ' . $block['className'];
    }
    
    $style = '';
    if ( $background_image ) {
        $style = 'background-image: url(' . esc_url( $background_image['url'] ) . ');';
    }
    ?>
    
    <section id="<?php echo esc_attr( $block_id ); ?>" 
             class="<?php echo esc_attr( $class_name ); ?>" 
             style="<?php echo esc_attr( $style ); ?>">
        <div class="hero-content">
            <?php if ( $heading ) : ?>
                <h1><?php echo esc_html( $heading ); ?></h1>
            <?php endif; ?>
            <?php if ( $subheading ) : ?>
                <p><?php echo esc_html( $subheading ); ?></p>
            <?php endif; ?>
            <?php if ( $button_text && $button_link ) : ?>
                <a href="<?php echo esc_url( $button_link ); ?>" class="btn">
                    <?php echo esc_html( $button_text ); ?>
                </a>
            <?php endif; ?>
        </div>
    </section>
    <?php
}
```

### Registering Field Groups with PHP

```php
<?php
/**
 * Register ACF field group programmatically
 */
if ( function_exists( 'acf_add_local_field_group' ) ) :
    
    acf_add_local_field_group( array(
        'key'    => 'group_page_settings',
        'title'  => 'Page Settings',
        'fields' => array(
            array(
                'key'   => 'field_page_subtitle',
                'label' => 'Subtitle',
                'name'  => 'page_subtitle',
                'type'  => 'text',
            ),
            array(
                'key'          => 'field_hide_title',
                'label'        => 'Hide Title',
                'name'         => 'hide_title',
                'type'         => 'true_false',
                'default_value' => 0,
            ),
            array(
                'key'   => 'field_banner_image',
                'label' => 'Banner Image',
                'name'  => 'banner_image',
                'type'  => 'image',
                'return_format' => 'array',
                'preview_size'  => 'medium',
            ),
        ),
        'location' => array(
            array(
                array(
                    'param'    => 'post_type',
                    'operator' => '==',
                    'value'    => 'page',
                ),
            ),
        ),
        'menu_order'            => 0,
        'position'              => 'normal',
        'style'                 => 'default',
        'label_placement'       => 'top',
        'instruction_placement' => 'label',
        'hide_on_screen'        => '',
    ) );
    
endif;
```

## Best Practices

### DO's

1. **Use get_field() Instead of the_field()**: More control over output
2. **Check Field Existence**: Always check if field has value before displaying
3. **Escape Output**: Use esc_html(), esc_url(), etc.
4. **Use Appropriate Return Formats**: Choose the right return format for your needs
5. **wp_reset_postdata()**: Reset after relationship/post object loops
6. **Use Field Groups**: Organize fields logically
7. **Leverage Conditional Logic**: Show/hide fields based on conditions
8. **Use Options Pages**: For global/theme settings
9. **Name Fields Descriptively**: Use clear, descriptive names
10. **Document Field Groups**: Add instructions for editors
11. **Use ACF Blocks**: For repeatable, flexible content blocks
12. **Version Control Field Groups**: Export/import or use PHP registration
13. **Use Field Prefixes**: Prefix field names to avoid conflicts
14. **Validate Fields**: Use ACF's validation features
15. **Test Thoroughly**: Test with empty, partial, and full data

### DON'Ts

1. **Don't Echo Raw Field Data**: Always escape for security
2. **Don't Use the_field() for Attributes**: Use get_field() and escape
3. **Don't Forget wp_reset_postdata()**: After custom loops
4. **Don't Hardcode Field Keys**: Use field names instead
5. **Don't Overuse Flexible Content**: Can be confusing for editors
6. **Don't Skip Validation**: Validate user input
7. **Don't Create Too Many Field Groups**: Keep organized
8. **Don't Use ACF for Everything**: Some things work better as plugins
9. **Don't Forget Mobile Layouts**: Test on different devices
10. **Don't Ignore Performance**: Large repeaters can slow down admin

## Cursor-Specific Instructions

When prompting Cursor AI for ACF development:

```
"Create an ACF field group for [describe purpose]. Include appropriate field
types, conditional logic, and proper location rules. Follow ACF best practices."

"Display ACF fields for [describe content]. Use get_field(), proper escaping,
check for field existence, and follow WordPress security practices."

"Build an ACF block for [describe block]. Include proper registration, template
file, field handling, and support for block features like alignment and className."

"Create a flexible content layout with ACF that includes [describe layouts].
Use proper conditional checks, escape output, and make it editor-friendly."
```

## Common Pitfalls

1. **Not escaping output**: Security vulnerability
2. **Using the_field() in attributes**: Can break HTML
3. **Forgetting wp_reset_postdata()**: Breaks subsequent WordPress loops
4. **Not checking if field exists**: Can cause PHP notices
5. **Misusing return formats**: Wrong format for the use case
6. **Overcomplicating flexible content**: Too many layout options confuse editors
7. **Not handling empty states**: Should check if field has value
8. **Hardcoding field keys**: Use names for better maintainability
9. **Not using conditional logic**: Fields show when they shouldn't
10. **Performance issues**: Too many fields or large repeaters

## Resources

- [ACF Documentation](https://www.advancedcustomfields.com/resources/)
- [ACF Field Types](https://www.advancedcustomfields.com/resources/#field-types)
- [ACF Functions](https://www.advancedcustomfields.com/resources/#functions)
- [ACF Blocks](https://www.advancedcustomfields.com/resources/blocks/)
- [ACF Options Page](https://www.advancedcustomfields.com/resources/options-page/)
- [ACF Local JSON](https://www.advancedcustomfields.com/resources/local-json/)

---

**Remember**: ACF is powerful but should be used thoughtfully. Always prioritize user experience for content editors and site visitors.

