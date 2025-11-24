# Gutenberg Blocks Development Rules

## Purpose

Comprehensive guidelines for developing custom Gutenberg (Block Editor) blocks, including modern block development with block.json, dynamic blocks, block patterns, and best practices.

## Key Principles

### 1. Use block.json

Modern block development uses `block.json` for block registration and metadata.

### 2. React-Based Development

Blocks are built using React for the editor interface.

### 3. Accessibility First

Ensure blocks are accessible and follow WCAG guidelines.

### 4. Performance

Optimize blocks for performance, especially in the editor.

## Block Development Setup

### Project Structure

```
plugin-name/
├── plugin-name.php
├── build/                      # Compiled files
│   ├── index.js
│   ├── index.asset.php
│   └── style-index.css
├── src/                        # Source files
│   ├── blocks/
│   │   ├── my-block/
│   │   │   ├── block.json
│   │   │   ├── edit.js
│   │   │   ├── save.js
│   │   │   ├── style.scss
│   │   │   └── editor.scss
│   │   └── another-block/
│   │       └── ...
│   └── index.js
├── package.json
└── webpack.config.js
```

### Package.json Setup

```json
{
  "name": "my-blocks",
  "version": "1.0.0",
  "scripts": {
    "build": "wp-scripts build",
    "start": "wp-scripts start",
    "format": "wp-scripts format",
    "lint:css": "wp-scripts lint-style",
    "lint:js": "wp-scripts lint-js"
  },
  "devDependencies": {
    "@wordpress/scripts": "^26.0.0"
  }
}
```

## Common Patterns

### block.json Configuration

```json
{
  "$schema": "https://schemas.wp.org/trunk/block.json",
  "apiVersion": 3,
  "name": "my-plugin/my-block",
  "version": "1.0.0",
  "title": "My Custom Block",
  "category": "widgets",
  "icon": "smiley",
  "description": "A custom block for my plugin.",
  "keywords": ["custom", "block"],
  "textdomain": "my-plugin",
  "attributes": {
    "content": {
      "type": "string",
      "source": "html",
      "selector": "p"
    },
    "alignment": {
      "type": "string",
      "default": "left"
    },
    "backgroundColor": {
      "type": "string"
    }
  },
  "supports": {
    "html": false,
    "align": true,
    "alignWide": true,
    "color": {
      "background": true,
      "text": true
    },
    "spacing": {
      "margin": true,
      "padding": true
    },
    "typography": {
      "fontSize": true,
      "lineHeight": true
    }
  },
  "styles": [
    {
      "name": "default",
      "label": "Default",
      "isDefault": true
    },
    {
      "name": "bordered",
      "label": "Bordered"
    }
  ],
  "editorScript": "file:./index.js",
  "editorStyle": "file:./editor.css",
  "style": "file:./style.css"
}
```

### Registering Blocks (PHP)

```php
<?php
/**
 * Register custom blocks
 */
function my_plugin_register_blocks() {
    // Register block from block.json
    register_block_type( __DIR__ . '/build/blocks/my-block' );
    
    // Register multiple blocks
    $blocks = array(
        'my-block',
        'another-block',
        'third-block',
    );
    
    foreach ( $blocks as $block ) {
        register_block_type( __DIR__ . '/build/blocks/' . $block );
    }
}
add_action( 'init', 'my_plugin_register_blocks' );
```

### Simple Static Block (edit.js)

```javascript
import { __ } from '@wordpress/i18n';
import { useBlockProps, RichText } from '@wordpress/block-editor';
import './editor.scss';

export default function Edit({ attributes, setAttributes }) {
    const { content } = attributes;
    const blockProps = useBlockProps();

    return (
        <div {...blockProps}>
            <RichText
                tagName="p"
                value={content}
                onChange={(content) => setAttributes({ content })}
                placeholder={__('Enter your text...', 'my-plugin')}
            />
        </div>
    );
}
```

### Simple Static Block (save.js)

```javascript
import { useBlockProps, RichText } from '@wordpress/block-editor';

export default function Save({ attributes }) {
    const { content } = attributes;
    const blockProps = useBlockProps.save();

    return (
        <div {...blockProps}>
            <RichText.Content tagName="p" value={content} />
        </div>
    );
}
```

### Block with InspectorControls

```javascript
import { __ } from '@wordpress/i18n';
import {
    useBlockProps,
    InspectorControls,
    RichText,
    ColorPalette,
} from '@wordpress/block-editor';
import {
    PanelBody,
    SelectControl,
    ToggleControl,
    RangeControl,
} from '@wordpress/components';

export default function Edit({ attributes, setAttributes }) {
    const {
        content,
        alignment,
        backgroundColor,
        fontSize,
        showBorder,
    } = attributes;

    const blockProps = useBlockProps({
        style: {
            backgroundColor: backgroundColor,
            textAlign: alignment,
            fontSize: fontSize + 'px',
        },
    });

    return (
        <>
            <InspectorControls>
                <PanelBody title={__('Settings', 'my-plugin')} initialOpen={true}>
                    <SelectControl
                        label={__('Alignment', 'my-plugin')}
                        value={alignment}
                        options={[
                            { label: 'Left', value: 'left' },
                            { label: 'Center', value: 'center' },
                            { label: 'Right', value: 'right' },
                        ]}
                        onChange={(alignment) => setAttributes({ alignment })}
                    />
                    
                    <RangeControl
                        label={__('Font Size', 'my-plugin')}
                        value={fontSize}
                        onChange={(fontSize) => setAttributes({ fontSize })}
                        min={12}
                        max={48}
                    />
                    
                    <ToggleControl
                        label={__('Show Border', 'my-plugin')}
                        checked={showBorder}
                        onChange={(showBorder) => setAttributes({ showBorder })}
                    />
                </PanelBody>
                
                <PanelBody title={__('Color Settings', 'my-plugin')} initialOpen={false}>
                    <ColorPalette
                        value={backgroundColor}
                        onChange={(backgroundColor) => setAttributes({ backgroundColor })}
                    />
                </PanelBody>
            </InspectorControls>

            <div {...blockProps}>
                <RichText
                    tagName="p"
                    value={content}
                    onChange={(content) => setAttributes({ content })}
                    placeholder={__('Enter your text...', 'my-plugin')}
                />
            </div>
        </>
    );
}
```

### Block with BlockControls

```javascript
import { __ } from '@wordpress/i18n';
import {
    useBlockProps,
    BlockControls,
    AlignmentToolbar,
    RichText,
} from '@wordpress/block-editor';
import {
    ToolbarGroup,
    ToolbarButton,
} from '@wordpress/components';
import { formatBold, formatItalic } from '@wordpress/icons';

export default function Edit({ attributes, setAttributes }) {
    const { content, alignment, isBold } = attributes;
    const blockProps = useBlockProps();

    return (
        <>
            <BlockControls>
                <AlignmentToolbar
                    value={alignment}
                    onChange={(alignment) => setAttributes({ alignment })}
                />
                
                <ToolbarGroup>
                    <ToolbarButton
                        icon={formatBold}
                        label={__('Bold', 'my-plugin')}
                        isPressed={isBold}
                        onClick={() => setAttributes({ isBold: !isBold })}
                    />
                </ToolbarGroup>
            </BlockControls>

            <div {...blockProps}>
                <RichText
                    tagName="p"
                    value={content}
                    onChange={(content) => setAttributes({ content })}
                    style={{ textAlign: alignment, fontWeight: isBold ? 'bold' : 'normal' }}
                />
            </div>
        </>
    );
}
```

### Dynamic Block (Server-Side Rendering)

#### block.json

```json
{
  "apiVersion": 3,
  "name": "my-plugin/dynamic-block",
  "title": "Dynamic Block",
  "category": "widgets",
  "attributes": {
    "postsToShow": {
      "type": "number",
      "default": 5
    },
    "category": {
      "type": "string"
    }
  },
  "editorScript": "file:./index.js",
  "render": "file:./render.php"
}
```

#### edit.js

```javascript
import { __ } from '@wordpress/i18n';
import { useBlockProps, InspectorControls } from '@wordpress/block-editor';
import { PanelBody, RangeControl, SelectControl } from '@wordpress/components';
import { useSelect } from '@wordpress/data';
import ServerSideRender from '@wordpress/server-side-render';

export default function Edit({ attributes, setAttributes }) {
    const { postsToShow, category } = attributes;
    const blockProps = useBlockProps();

    // Get categories for the select control
    const categories = useSelect((select) => {
        return select('core').getEntityRecords('taxonomy', 'category', {
            per_page: -1,
        });
    }, []);

    const categoryOptions = [
        { label: __('All Categories', 'my-plugin'), value: '' },
    ];

    if (categories) {
        categories.forEach((cat) => {
            categoryOptions.push({
                label: cat.name,
                value: cat.id,
            });
        });
    }

    return (
        <>
            <InspectorControls>
                <PanelBody title={__('Settings', 'my-plugin')}>
                    <RangeControl
                        label={__('Number of Posts', 'my-plugin')}
                        value={postsToShow}
                        onChange={(postsToShow) => setAttributes({ postsToShow })}
                        min={1}
                        max={10}
                    />
                    
                    <SelectControl
                        label={__('Category', 'my-plugin')}
                        value={category}
                        options={categoryOptions}
                        onChange={(category) => setAttributes({ category })}
                    />
                </PanelBody>
            </InspectorControls>

            <div {...blockProps}>
                <ServerSideRender
                    block="my-plugin/dynamic-block"
                    attributes={attributes}
                />
            </div>
        </>
    );
}
```

#### save.js (Dynamic blocks don't need save)

```javascript
export default function Save() {
    return null; // Dynamic block, rendered by PHP
}
```

#### render.php

```php
<?php
/**
 * Server-side rendering for dynamic block
 * 
 * @param array $attributes Block attributes
 * @param string $content Block content
 * @return string Block HTML
 */

$posts_to_show = isset( $attributes['postsToShow'] ) ? absint( $attributes['postsToShow'] ) : 5;
$category = isset( $attributes['category'] ) ? absint( $attributes['category'] ) : 0;

$args = array(
    'post_type'      => 'post',
    'posts_per_page' => $posts_to_show,
    'post_status'    => 'publish',
);

if ( $category ) {
    $args['cat'] = $category;
}

$query = new WP_Query( $args );

if ( ! $query->have_posts() ) {
    return '<p>' . esc_html__( 'No posts found.', 'my-plugin' ) . '</p>';
}

$output = '<div class="my-dynamic-block">';

while ( $query->have_posts() ) {
    $query->the_post();
    $output .= '<article class="post-item">';
    $output .= '<h3><a href="' . esc_url( get_permalink() ) . '">' . esc_html( get_the_title() ) . '</a></h3>';
    $output .= '<div class="excerpt">' . wp_kses_post( get_the_excerpt() ) . '</div>';
    $output .= '</article>';
}

$output .= '</div>';

wp_reset_postdata();

echo $output;
```

### InnerBlocks (Nested Blocks)

```javascript
import { __ } from '@wordpress/i18n';
import { useBlockProps, InnerBlocks } from '@wordpress/block-editor';

const ALLOWED_BLOCKS = ['core/heading', 'core/paragraph', 'core/image'];

const TEMPLATE = [
    ['core/heading', { level: 2, placeholder: 'Enter heading...' }],
    ['core/paragraph', { placeholder: 'Enter content...' }],
];

export default function Edit() {
    const blockProps = useBlockProps();

    return (
        <div {...blockProps}>
            <InnerBlocks
                allowedBlocks={ALLOWED_BLOCKS}
                template={TEMPLATE}
            />
        </div>
    );
}

// save.js
export default function Save() {
    const blockProps = useBlockProps.save();

    return (
        <div {...blockProps}>
            <InnerBlocks.Content />
        </div>
    );
}
```

### Block with Media Upload

```javascript
import { __ } from '@wordpress/i18n';
import {
    useBlockProps,
    MediaUpload,
    MediaUploadCheck,
} from '@wordpress/block-editor';
import { Button } from '@wordpress/components';

export default function Edit({ attributes, setAttributes }) {
    const { imageId, imageUrl, imageAlt } = attributes;
    const blockProps = useBlockProps();

    const onSelectImage = (media) => {
        setAttributes({
            imageId: media.id,
            imageUrl: media.url,
            imageAlt: media.alt,
        });
    };

    const removeImage = () => {
        setAttributes({
            imageId: null,
            imageUrl: null,
            imageAlt: null,
        });
    };

    return (
        <div {...blockProps}>
            <MediaUploadCheck>
                <MediaUpload
                    onSelect={onSelectImage}
                    allowedTypes={['image']}
                    value={imageId}
                    render={({ open }) => (
                        <>
                            {!imageUrl ? (
                                <Button onClick={open} variant="primary">
                                    {__('Upload Image', 'my-plugin')}
                                </Button>
                            ) : (
                                <div className="image-wrapper">
                                    <img src={imageUrl} alt={imageAlt} />
                                    <div className="image-controls">
                                        <Button onClick={open} variant="secondary">
                                            {__('Replace Image', 'my-plugin')}
                                        </Button>
                                        <Button onClick={removeImage} variant="link" isDestructive>
                                            {__('Remove Image', 'my-plugin')}
                                        </Button>
                                    </div>
                                </div>
                            )}
                        </>
                    )}
                />
            </MediaUploadCheck>
        </div>
    );
}
```

### Block with API Data

```javascript
import { __ } from '@wordpress/i18n';
import { useBlockProps } from '@wordpress/block-editor';
import { useSelect } from '@wordpress/data';
import { Spinner } from '@wordpress/components';
import { store as coreStore } from '@wordpress/core-data';

export default function Edit() {
    const blockProps = useBlockProps();

    const { posts, isResolving } = useSelect((select) => {
        const { getEntityRecords, isResolving: checkResolving } = select(coreStore);
        
        return {
            posts: getEntityRecords('postType', 'post', {
                per_page: 5,
                _embed: true,
            }),
            isResolving: checkResolving('getEntityRecords', [
                'postType',
                'post',
                { per_page: 5 },
            ]),
        };
    }, []);

    return (
        <div {...blockProps}>
            {isResolving ? (
                <Spinner />
            ) : (
                <div className="posts-list">
                    {posts && posts.length > 0 ? (
                        posts.map((post) => (
                            <article key={post.id} className="post-item">
                                <h3>{post.title.rendered}</h3>
                                <div
                                    dangerouslySetInnerHTML={{
                                        __html: post.excerpt.rendered,
                                    }}
                                />
                            </article>
                        ))
                    ) : (
                        <p>{__('No posts found.', 'my-plugin')}</p>
                    )}
                </div>
            )}
        </div>
    );
}
```

### Block Variations

```javascript
import { registerBlockVariation } from '@wordpress/blocks';

registerBlockVariation('core/quote', {
    name: 'fancy-quote',
    title: 'Fancy Quote',
    description: 'A stylized quote block',
    icon: 'format-quote',
    attributes: {
        className: 'fancy-quote',
    },
    isActive: (blockAttributes) => {
        return blockAttributes.className === 'fancy-quote';
    },
});
```

### Block Patterns

```php
<?php
/**
 * Register block patterns
 */
function my_plugin_register_block_patterns() {
    // Register pattern category
    register_block_pattern_category(
        'my-plugin',
        array(
            'label' => __( 'My Plugin Patterns', 'my-plugin' ),
        )
    );
    
    // Register pattern
    register_block_pattern(
        'my-plugin/hero-section',
        array(
            'title'       => __( 'Hero Section', 'my-plugin' ),
            'description' => __( 'A hero section with heading and button', 'my-plugin' ),
            'categories'  => array( 'my-plugin' ),
            'content'     => '<!-- wp:group {"align":"full","backgroundColor":"primary"} -->
                <div class="wp-block-group alignfull has-primary-background-color">
                    <!-- wp:heading {"textAlign":"center","level":1} -->
                    <h1 class="has-text-align-center">Welcome to Our Site</h1>
                    <!-- /wp:heading -->
                    
                    <!-- wp:paragraph {"align":"center"} -->
                    <p class="has-text-align-center">This is a hero section pattern.</p>
                    <!-- /wp:paragraph -->
                    
                    <!-- wp:buttons {"layout":{"type":"flex","justifyContent":"center"}} -->
                    <div class="wp-block-buttons">
                        <!-- wp:button -->
                        <div class="wp-block-button"><a class="wp-block-button__link">Learn More</a></div>
                        <!-- /wp:button -->
                    </div>
                    <!-- /wp:buttons -->
                </div>
                <!-- /wp:group -->',
        )
    );
}
add_action( 'init', 'my_plugin_register_block_patterns' );
```

### Block Styles

```php
<?php
/**
 * Register block styles
 */
function my_plugin_register_block_styles() {
    // Register style for existing block
    register_block_style(
        'core/quote',
        array(
            'name'  => 'fancy-quote',
            'label' => __( 'Fancy Quote', 'my-plugin' ),
        )
    );
    
    register_block_style(
        'core/button',
        array(
            'name'         => 'outline-button',
            'label'        => __( 'Outline', 'my-plugin' ),
            'inline_style' => '.wp-block-button.is-style-outline-button .wp-block-button__link {
                background-color: transparent;
                border: 2px solid currentColor;
            }',
        )
    );
}
add_action( 'init', 'my_plugin_register_block_styles' );

/**
 * Unregister block style
 */
function my_plugin_unregister_block_styles() {
    unregister_block_style( 'core/quote', 'large' );
}
add_action( 'init', 'my_plugin_unregister_block_styles' );
```

### Block Transforms

```javascript
import { createBlock } from '@wordpress/blocks';

const transforms = {
    from: [
        {
            type: 'block',
            blocks: ['core/paragraph'],
            transform: (attributes) => {
                return createBlock('my-plugin/my-block', {
                    content: attributes.content,
                });
            },
        },
    ],
    to: [
        {
            type: 'block',
            blocks: ['core/paragraph'],
            transform: (attributes) => {
                return createBlock('core/paragraph', {
                    content: attributes.content,
                });
            },
        },
    ],
};

// Add to block registration
registerBlockType('my-plugin/my-block', {
    // ... other properties
    transforms,
});
```

## Best Practices

### DO's

1. **Use block.json**: Modern block registration standard
2. **Follow Naming Conventions**: Use plugin-name/block-name format
3. **Make Blocks Accessible**: Follow WCAG guidelines
4. **Support Core Features**: Align, colors, typography when appropriate
5. **Use InspectorControls**: For block settings
6. **Use BlockControls**: For inline formatting toolbar
7. **Sanitize Output**: Escape and sanitize in save and render
8. **Test in Editor**: Test block behavior in editor
9. **Mobile Responsive**: Ensure blocks work on mobile
10. **Document Blocks**: Add clear descriptions and examples
11. **Use Icons**: Provide recognizable block icons
12. **Version Blocks**: Use deprecation for breaking changes
13. **Localize Strings**: Use i18n functions
14. **Optimize Performance**: Avoid heavy operations in editor
15. **Provide Defaults**: Set sensible default attributes

### DON'Ts

1. **Don't Use jQuery**: Use vanilla JS or React
2. **Don't Modify Core Blocks**: Extend instead
3. **Don't Skip Sanitization**: Security risk
4. **Don't Forget Mobile**: Test on devices
5. **Don't Overcomplicate**: Keep blocks focused
6. **Don't Skip save()**: Unless dynamic block
7. **Don't Hardcode Values**: Make configurable
8. **Don't Ignore Accessibility**: Must be accessible
9. **Don't Break Updates**: Use deprecation properly
10. **Don't Forget Performance**: Optimize for editor

## Cursor-Specific Instructions

When prompting Cursor AI for Gutenberg block development:

```
"Create a Gutenberg block for [describe block]. Use modern block development with
block.json, include InspectorControls for settings, make it accessible, and follow
WordPress block development best practices."

"Build a dynamic Gutenberg block that [describes functionality]. Use server-side
rendering, include appropriate controls, sanitize output, and optimize for
performance."

"Implement a custom block with [features]. Include BlockControls for inline
formatting, InspectorControls for settings, support core features like alignment
and colors, and ensure mobile responsiveness."
```

## Common Pitfalls

1. **Not using block.json**: Outdated registration method
2. **Forgetting save function**: For static blocks
3. **Not sanitizing output**: Security vulnerability
4. **Breaking block validation**: Causes block errors
5. **Poor accessibility**: Excludes users
6. **Not supporting mobile**: Poor mobile experience
7. **Overcomplicated UI**: Confusing for users
8. **No default attributes**: Undefined values
9. **Heavy editor operations**: Slows down editor
10. **Not testing variations**: Blocks break in different scenarios

## Resources

- [Block Editor Handbook](https://developer.wordpress.org/block-editor/)
- [Block API Reference](https://developer.wordpress.org/block-editor/reference-guides/block-api/)
- [Components Reference](https://developer.wordpress.org/block-editor/reference-guides/components/)
- [Block Patterns](https://developer.wordpress.org/block-editor/reference-guides/block-api/block-patterns/)
- [@wordpress/scripts](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-scripts/)
- [Block Development Examples](https://github.com/WordPress/block-development-examples)

---

**Remember**: Modern block development prioritizes accessibility, performance, and user experience. Always test blocks thoroughly in different scenarios.

