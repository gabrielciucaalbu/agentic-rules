# WooCommerce Development Rules

## Purpose

Comprehensive guidelines for WooCommerce development including product customization, cart and checkout modifications, custom product types, hooks and filters, and e-commerce best practices.

## Key Principles

### 1. Use WooCommerce Hooks

Never modify WooCommerce core files. Always use hooks and filters to customize functionality.

### 2. Child Themes for Templates

Override WooCommerce templates using child themes, not by modifying the plugin files.

### 3. Maintain Compatibility

Ensure customizations work with WooCommerce updates and maintain backward compatibility.

### 4. Performance

Optimize queries and caching for better store performance.

## Common Patterns

### Template Override

```
theme/
└── woocommerce/
    ├── single-product.php
    ├── archive-product.php
    ├── cart/
    │   ├── cart.php
    │   └── cart-empty.php
    ├── checkout/
    │   ├── form-checkout.php
    │   └── thankyou.php
    ├── myaccount/
    │   └── my-account.php
    └── content-product.php
```

### Declaring WooCommerce Support

```php
<?php
/**
 * Add WooCommerce support in functions.php
 */
function mytheme_add_woocommerce_support() {
    add_theme_support( 'woocommerce' );
    
    // Optional: Add product gallery features
    add_theme_support( 'wc-product-gallery-zoom' );
    add_theme_support( 'wc-product-gallery-lightbox' );
    add_theme_support( 'wc-product-gallery-slider' );
}
add_action( 'after_setup_theme', 'mytheme_add_woocommerce_support' );

/**
 * Set image dimensions
 */
function mytheme_woocommerce_image_dimensions() {
    $catalog = array(
        'width'  => 300,
        'height' => 300,
        'crop'   => 1,
    );
    
    $single = array(
        'width'  => 600,
        'height' => 600,
        'crop'   => 1,
    );
    
    $thumbnail = array(
        'width'  => 180,
        'height' => 180,
        'crop'   => 1,
    );
    
    update_option( 'shop_catalog_image_size', $catalog );
    update_option( 'shop_single_image_size', $single );
    update_option( 'shop_thumbnail_image_size', $thumbnail );
}
add_action( 'after_setup_theme', 'mytheme_woocommerce_image_dimensions' );
```

### Product Information

```php
<?php
// Get product object
global $product;

// Or get by ID
$product = wc_get_product( $product_id );

// Product properties
$product_id = $product->get_id();
$product_name = $product->get_name();
$product_slug = $product->get_slug();
$product_type = $product->get_type(); // simple, variable, grouped, external

// Prices
$regular_price = $product->get_regular_price();
$sale_price = $product->get_sale_price();
$price = $product->get_price();
$price_html = $product->get_price_html();

// Stock
$stock_quantity = $product->get_stock_quantity();
$stock_status = $product->get_stock_status(); // instock, outofstock, onbackorder
$is_in_stock = $product->is_in_stock();
$is_on_sale = $product->is_on_sale();

// Images
$image_id = $product->get_image_id();
$image_url = wp_get_attachment_image_url( $image_id, 'full' );
$gallery_ids = $product->get_gallery_image_ids();

// Categories and tags
$categories = $product->get_category_ids();
$tags = $product->get_tag_ids();

// Attributes
$attributes = $product->get_attributes();

// SKU
$sku = $product->get_sku();

// Weight and dimensions
$weight = $product->get_weight();
$length = $product->get_length();
$width = $product->get_width();
$height = $product->get_height();

// Reviews
$rating_count = $product->get_rating_count();
$average_rating = $product->get_average_rating();
$review_count = $product->get_review_count();

// Purchasing
$is_purchasable = $product->is_purchasable();
$is_virtual = $product->is_virtual();
$is_downloadable = $product->is_downloadable();

// Permalinks
$permalink = $product->get_permalink();
$add_to_cart_url = $product->add_to_cart_url();
```

### Variable Products

```php
<?php
// Get variable product
$product = wc_get_product( $product_id );

if ( $product->is_type( 'variable' ) ) {
    // Get variations
    $variations = $product->get_available_variations();
    
    foreach ( $variations as $variation ) {
        $variation_id = $variation['variation_id'];
        $variation_obj = wc_get_product( $variation_id );
        
        $attributes = $variation['attributes'];
        $price = $variation_obj->get_price();
        $sku = $variation_obj->get_sku();
    }
    
    // Get default attributes
    $default_attributes = $product->get_default_attributes();
    
    // Get variation attributes
    $attributes = $product->get_variation_attributes();
}
```

### Cart Operations

```php
<?php
// Get cart object
$cart = WC()->cart;

// Add to cart
WC()->cart->add_to_cart( $product_id, $quantity = 1, $variation_id = 0, $variation = array(), $cart_item_data = array() );

// Get cart contents
$cart_items = WC()->cart->get_cart();

foreach ( $cart_items as $cart_item_key => $cart_item ) {
    $product = $cart_item['data'];
    $product_id = $cart_item['product_id'];
    $quantity = $cart_item['quantity'];
    $line_total = $cart_item['line_total'];
    $line_subtotal = $cart_item['line_subtotal'];
}

// Update cart item quantity
WC()->cart->set_quantity( $cart_item_key, $quantity );

// Remove from cart
WC()->cart->remove_cart_item( $cart_item_key );

// Empty cart
WC()->cart->empty_cart();

// Get cart totals
$cart_subtotal = WC()->cart->get_cart_subtotal();
$cart_total = WC()->cart->get_cart_total();
$cart_tax = WC()->cart->get_cart_tax();
$shipping_total = WC()->cart->get_shipping_total();

// Get cart count
$cart_count = WC()->cart->get_cart_contents_count();

// Check if cart is empty
$is_empty = WC()->cart->is_empty();

// Get cart URL
$cart_url = wc_get_cart_url();

// Get checkout URL
$checkout_url = wc_get_checkout_url();
```

### Order Information

```php
<?php
// Get order object
$order = wc_get_order( $order_id );

// Order properties
$order_id = $order->get_id();
$order_key = $order->get_order_key();
$order_number = $order->get_order_number();
$order_status = $order->get_status();
$order_date = $order->get_date_created();
$order_total = $order->get_total();
$order_subtotal = $order->get_subtotal();
$order_tax = $order->get_total_tax();
$shipping_total = $order->get_shipping_total();

// Customer information
$customer_id = $order->get_customer_id();
$billing_first_name = $order->get_billing_first_name();
$billing_last_name = $order->get_billing_last_name();
$billing_email = $order->get_billing_email();
$billing_phone = $order->get_billing_phone();
$billing_address = $order->get_billing_address_1();
$billing_city = $order->get_billing_city();
$billing_state = $order->get_billing_state();
$billing_postcode = $order->get_billing_postcode();
$billing_country = $order->get_billing_country();

// Shipping information
$shipping_first_name = $order->get_shipping_first_name();
$shipping_last_name = $order->get_shipping_last_name();
$shipping_address = $order->get_shipping_address_1();
$shipping_city = $order->get_shipping_city();
$shipping_state = $order->get_shipping_state();
$shipping_postcode = $order->get_shipping_postcode();
$shipping_country = $order->get_shipping_country();

// Order items
$items = $order->get_items();

foreach ( $items as $item_id => $item ) {
    $product = $item->get_product();
    $product_id = $item->get_product_id();
    $variation_id = $item->get_variation_id();
    $quantity = $item->get_quantity();
    $subtotal = $item->get_subtotal();
    $total = $item->get_total();
    $tax = $item->get_total_tax();
}

// Update order status
$order->update_status( 'completed', 'Order completed.', true );

// Add order note
$order->add_order_note( 'Custom note for this order.' );

// Get order notes
$notes = $order->get_customer_order_notes();
```

### WooCommerce Hooks

#### Product Page Hooks

```php
<?php
/**
 * Single product page hooks
 */

// Before single product
add_action( 'woocommerce_before_single_product', 'custom_before_product' );

// Product title
remove_action( 'woocommerce_single_product_summary', 'woocommerce_template_single_title', 5 );
add_action( 'woocommerce_single_product_summary', 'custom_product_title', 5 );

// Product price
remove_action( 'woocommerce_single_product_summary', 'woocommerce_template_single_price', 10 );
add_action( 'woocommerce_single_product_summary', 'custom_product_price', 10 );

// Product excerpt
remove_action( 'woocommerce_single_product_summary', 'woocommerce_template_single_excerpt', 20 );

// Add to cart button
remove_action( 'woocommerce_single_product_summary', 'woocommerce_template_single_add_to_cart', 30 );

// Product meta (SKU, categories, tags)
remove_action( 'woocommerce_single_product_summary', 'woocommerce_template_single_meta', 40 );

// Product sharing
remove_action( 'woocommerce_single_product_summary', 'woocommerce_template_single_sharing', 50 );

// Add custom content after title
add_action( 'woocommerce_single_product_summary', 'custom_product_content', 6 );

function custom_product_content() {
    echo '<div class="custom-content">Custom content here</div>';
}

// After single product
add_action( 'woocommerce_after_single_product', 'custom_after_product' );

// Related products
remove_action( 'woocommerce_after_single_product_summary', 'woocommerce_output_related_products', 20 );

// Change related products count
function custom_related_products_args( $args ) {
    $args['posts_per_page'] = 4;
    $args['columns'] = 4;
    return $args;
}
add_filter( 'woocommerce_output_related_products_args', 'custom_related_products_args' );
```

#### Shop/Archive Page Hooks

```php
<?php
/**
 * Shop page hooks
 */

// Before shop loop
add_action( 'woocommerce_before_shop_loop', 'custom_before_shop_loop' );

// Remove result count
remove_action( 'woocommerce_before_shop_loop', 'woocommerce_result_count', 20 );

// Remove ordering
remove_action( 'woocommerce_before_shop_loop', 'woocommerce_catalog_ordering', 30 );

// Products per page
add_filter( 'loop_shop_per_page', 'custom_products_per_page', 20 );

function custom_products_per_page( $cols ) {
    return 12;
}

// Product loop hooks
remove_action( 'woocommerce_before_shop_loop_item_title', 'woocommerce_template_loop_product_thumbnail', 10 );
remove_action( 'woocommerce_shop_loop_item_title', 'woocommerce_template_loop_product_title', 10 );
remove_action( 'woocommerce_after_shop_loop_item_title', 'woocommerce_template_loop_rating', 5 );
remove_action( 'woocommerce_after_shop_loop_item_title', 'woocommerce_template_loop_price', 10 );
remove_action( 'woocommerce_after_shop_loop_item', 'woocommerce_template_loop_add_to_cart', 10 );

// Change columns
add_filter( 'loop_shop_columns', 'custom_shop_columns' );

function custom_shop_columns() {
    return 4; // 4 columns
}

// After shop loop
add_action( 'woocommerce_after_shop_loop', 'custom_after_shop_loop' );
```

#### Cart Page Hooks

```php
<?php
/**
 * Cart page hooks
 */

// Before cart
add_action( 'woocommerce_before_cart', 'custom_before_cart' );

// Before cart table
add_action( 'woocommerce_before_cart_table', 'custom_before_cart_table' );

// After cart table
add_action( 'woocommerce_after_cart_table', 'custom_after_cart_table' );

// Before cart totals
add_action( 'woocommerce_before_cart_totals', 'custom_before_cart_totals' );

// Cart totals
add_action( 'woocommerce_cart_totals_before_order_total', 'custom_cart_total_info' );

// Proceed to checkout
add_action( 'woocommerce_proceed_to_checkout', 'custom_checkout_button' );

// After cart
add_action( 'woocommerce_after_cart', 'custom_after_cart' );

// Empty cart message
add_action( 'woocommerce_cart_is_empty', 'custom_empty_cart_message' );

// Add to cart validation
add_filter( 'woocommerce_add_to_cart_validation', 'custom_add_to_cart_validation', 10, 3 );

function custom_add_to_cart_validation( $passed, $product_id, $quantity ) {
    // Custom validation logic
    if ( $quantity > 5 ) {
        wc_add_notice( 'You cannot add more than 5 items.', 'error' );
        return false;
    }
    return $passed;
}

// Modify cart item
add_filter( 'woocommerce_add_cart_item_data', 'add_custom_cart_item_data', 10, 2 );

function add_custom_cart_item_data( $cart_item_data, $product_id ) {
    $cart_item_data['custom_data'] = 'custom value';
    return $cart_item_data;
}

// Display custom cart item data
add_filter( 'woocommerce_get_item_data', 'display_custom_cart_item_data', 10, 2 );

function display_custom_cart_item_data( $item_data, $cart_item ) {
    if ( isset( $cart_item['custom_data'] ) ) {
        $item_data[] = array(
            'name'  => 'Custom Field',
            'value' => sanitize_text_field( $cart_item['custom_data'] ),
        );
    }
    return $item_data;
}
```

#### Checkout Page Hooks

```php
<?php
/**
 * Checkout page hooks
 */

// Before checkout form
add_action( 'woocommerce_before_checkout_form', 'custom_before_checkout_form' );

// Checkout fields
add_filter( 'woocommerce_checkout_fields', 'custom_checkout_fields' );

function custom_checkout_fields( $fields ) {
    // Add custom field
    $fields['billing']['billing_custom_field'] = array(
        'type'        => 'text',
        'label'       => __( 'Custom Field', 'woocommerce' ),
        'placeholder' => __( 'Enter value', 'woocommerce' ),
        'required'    => false,
        'class'       => array( 'form-row-wide' ),
        'priority'    => 25,
    );
    
    // Remove field
    unset( $fields['billing']['billing_company'] );
    
    // Make field optional
    $fields['billing']['billing_phone']['required'] = false;
    
    return $fields;
}

// Validate checkout fields
add_action( 'woocommerce_checkout_process', 'custom_checkout_field_validation' );

function custom_checkout_field_validation() {
    if ( isset( $_POST['billing_custom_field'] ) && empty( $_POST['billing_custom_field'] ) ) {
        wc_add_notice( 'Please fill in the custom field.', 'error' );
    }
}

// Save custom field
add_action( 'woocommerce_checkout_update_order_meta', 'save_custom_checkout_field' );

function save_custom_checkout_field( $order_id ) {
    if ( ! empty( $_POST['billing_custom_field'] ) ) {
        update_post_meta( $order_id, 'custom_field', sanitize_text_field( $_POST['billing_custom_field'] ) );
    }
}

// Display custom field in admin
add_action( 'woocommerce_admin_order_data_after_billing_address', 'display_custom_field_in_admin', 10, 1 );

function display_custom_field_in_admin( $order ) {
    $custom_field = get_post_meta( $order->get_id(), 'custom_field', true );
    if ( $custom_field ) {
        echo '<p><strong>' . esc_html__( 'Custom Field', 'woocommerce' ) . ':</strong> ' . esc_html( $custom_field ) . '</p>';
    }
}

// After checkout form
add_action( 'woocommerce_after_checkout_form', 'custom_after_checkout_form' );

// Order received (thank you page)
add_action( 'woocommerce_thankyou', 'custom_thankyou_page' );

function custom_thankyou_page( $order_id ) {
    $order = wc_get_order( $order_id );
    echo '<div class="custom-thankyou">';
    echo '<p>Thank you for your order #' . esc_html( $order_id ) . '</p>';
    echo '</div>';
}
```

### Custom Product Type

```php
<?php
/**
 * Register custom product type
 */
class WC_Product_Custom extends WC_Product {
    
    public function __construct( $product ) {
        $this->product_type = 'custom';
        parent::__construct( $product );
    }
    
    public function get_type() {
        return 'custom';
    }
    
    // Add custom methods
    public function custom_method() {
        return 'Custom functionality';
    }
}

/**
 * Add custom product type to WooCommerce
 */
function add_custom_product_type( $types ) {
    $types['custom'] = __( 'Custom Product', 'woocommerce' );
    return $types;
}
add_filter( 'product_type_selector', 'add_custom_product_type' );

/**
 * Register custom product class
 */
function register_custom_product_class( $classname, $product_type ) {
    if ( $product_type === 'custom' ) {
        $classname = 'WC_Product_Custom';
    }
    return $classname;
}
add_filter( 'woocommerce_product_class', 'register_custom_product_class', 10, 2 );
```

### Custom Email Notifications

```php
<?php
/**
 * Add custom email class
 */
class WC_Custom_Email extends WC_Email {
    
    public function __construct() {
        $this->id             = 'custom_email';
        $this->title          = __( 'Custom Email', 'woocommerce' );
        $this->description    = __( 'Custom email notification.', 'woocommerce' );
        $this->heading        = __( 'Custom Email Heading', 'woocommerce' );
        $this->subject        = __( 'Custom Email Subject', 'woocommerce' );
        $this->template_html  = 'emails/custom-email.php';
        $this->template_plain = 'emails/plain/custom-email.php';
        
        // Triggers
        add_action( 'custom_email_trigger', array( $this, 'trigger' ) );
        
        parent::__construct();
    }
    
    public function trigger( $order_id ) {
        if ( ! $order_id ) {
            return;
        }
        
        $this->object = wc_get_order( $order_id );
        $this->recipient = $this->object->get_billing_email();
        
        if ( ! $this->is_enabled() || ! $this->get_recipient() ) {
            return;
        }
        
        $this->send( 
            $this->get_recipient(), 
            $this->get_subject(), 
            $this->get_content(), 
            $this->get_headers(), 
            $this->get_attachments() 
        );
    }
    
    public function get_content_html() {
        return wc_get_template_html( 
            $this->template_html, 
            array(
                'order'         => $this->object,
                'email_heading' => $this->get_heading(),
                'sent_to_admin' => false,
                'plain_text'    => false,
                'email'         => $this,
            )
        );
    }
}

/**
 * Add custom email to WooCommerce emails
 */
function add_custom_woocommerce_email( $emails ) {
    $emails['WC_Custom_Email'] = new WC_Custom_Email();
    return $emails;
}
add_filter( 'woocommerce_email_classes', 'add_custom_woocommerce_email' );

// Trigger the custom email
do_action( 'custom_email_trigger', $order_id );
```

### Payment Gateway

```php
<?php
/**
 * Custom payment gateway
 */
class WC_Gateway_Custom extends WC_Payment_Gateway {
    
    public function __construct() {
        $this->id                 = 'custom_gateway';
        $this->icon               = '';
        $this->has_fields         = false;
        $this->method_title       = __( 'Custom Payment', 'woocommerce' );
        $this->method_description = __( 'Custom payment gateway.', 'woocommerce' );
        
        $this->init_form_fields();
        $this->init_settings();
        
        $this->title       = $this->get_option( 'title' );
        $this->description = $this->get_option( 'description' );
        $this->enabled     = $this->get_option( 'enabled' );
        
        add_action( 'woocommerce_update_options_payment_gateways_' . $this->id, array( $this, 'process_admin_options' ) );
    }
    
    public function init_form_fields() {
        $this->form_fields = array(
            'enabled' => array(
                'title'   => __( 'Enable/Disable', 'woocommerce' ),
                'type'    => 'checkbox',
                'label'   => __( 'Enable Custom Payment', 'woocommerce' ),
                'default' => 'yes',
            ),
            'title' => array(
                'title'       => __( 'Title', 'woocommerce' ),
                'type'        => 'text',
                'description' => __( 'Payment method title.', 'woocommerce' ),
                'default'     => __( 'Custom Payment', 'woocommerce' ),
                'desc_tip'    => true,
            ),
            'description' => array(
                'title'       => __( 'Description', 'woocommerce' ),
                'type'        => 'textarea',
                'description' => __( 'Payment method description.', 'woocommerce' ),
                'default'     => __( 'Pay with custom method.', 'woocommerce' ),
            ),
        );
    }
    
    public function process_payment( $order_id ) {
        $order = wc_get_order( $order_id );
        
        // Mark as processing or complete
        $order->payment_complete();
        
        // Add order note
        $order->add_order_note( __( 'Payment completed via custom gateway.', 'woocommerce' ) );
        
        // Empty cart
        WC()->cart->empty_cart();
        
        // Redirect to thank you page
        return array(
            'result'   => 'success',
            'redirect' => $this->get_return_url( $order ),
        );
    }
}

/**
 * Add custom gateway to WooCommerce
 */
function add_custom_payment_gateway( $gateways ) {
    $gateways[] = 'WC_Gateway_Custom';
    return $gateways;
}
add_filter( 'woocommerce_payment_gateways', 'add_custom_payment_gateway' );
```

## Best Practices

### DO's

1. **Use WooCommerce Hooks**: Never modify core files
2. **Override Templates**: Use child theme template overrides
3. **Check for WooCommerce**: Verify WooCommerce is active before using functions
4. **Use WC Objects**: Use WC_Product, WC_Order classes
5. **Validate Input**: Sanitize and validate user input
6. **Test Thoroughly**: Test checkout process completely
7. **Mobile Optimization**: Ensure mobile-friendly checkout
8. **Use Actions for Side Effects**: Don't use filters for side effects
9. **Version Compatibility**: Test with WooCommerce updates
10. **Performance**: Optimize queries and caching
11. **Security**: Follow WooCommerce security practices
12. **Documentation**: Document custom functionality
13. **Child Themes**: Always use child themes
14. **Error Handling**: Handle errors gracefully
15. **Backup**: Always backup before customizations

### DON'Ts

1. **Don't Modify Core**: Never edit WooCommerce plugin files
2. **Don't Override All Templates**: Only override what you need
3. **Don't Forget Mobile**: Test on mobile devices
4. **Don't Skip Testing**: Test entire purchase flow
5. **Don't Ignore Updates**: Keep WooCommerce updated
6. **Don't Hardcode**: Make settings configurable
7. **Don't Break Checkout**: Test checkout thoroughly
8. **Don't Forget Taxes**: Consider tax implications
9. **Don't Skip Validation**: Validate all user input
10. **Don't Ignore Performance**: Optimize for speed

## Cursor-Specific Instructions

When prompting Cursor AI for WooCommerce development:

```
"Create WooCommerce customization for [describe feature]. Use WooCommerce hooks
and filters, don't modify core files, include proper validation and security,
and follow WooCommerce best practices."

"Build a custom [product/checkout/cart] feature for WooCommerce that [describes
functionality]. Use appropriate WooCommerce actions and filters, validate input,
and maintain compatibility with WooCommerce updates."

"Implement a WooCommerce payment gateway for [payment method]. Include proper
form fields, validation, order processing, and follow WooCommerce gateway standards."
```

## Common Pitfalls

1. **Modifying core files**: Updates will break changes
2. **Not checking if WooCommerce is active**: Causes errors
3. **Breaking checkout flow**: Poor testing
4. **Ignoring mobile users**: Desktop-only testing
5. **Not validating custom fields**: Security risk
6. **Forgetting to empty cart**: After successful order
7. **Not updating order status**: Orders stuck in pending
8. **Hardcoding values**: Not configurable
9. **Poor error handling**: User confusion
10. **Not testing with real payment**: Payment issues in production

## Resources

- [WooCommerce Documentation](https://woocommerce.com/documentation/)
- [WooCommerce Code Reference](https://woocommerce.github.io/code-reference/)
- [WooCommerce Hooks](https://woocommerce.github.io/code-reference/hooks/hooks.html)
- [WooCommerce REST API](https://woocommerce.github.io/woocommerce-rest-api-docs/)
- [Template Structure](https://woocommerce.com/document/template-structure/)
- [Payment Gateway API](https://woocommerce.com/document/payment-gateway-api/)

---

**Remember**: WooCommerce customizations must maintain compatibility with updates and provide a seamless shopping experience.

