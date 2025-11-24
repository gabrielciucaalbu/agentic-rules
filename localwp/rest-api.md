# WordPress REST API Development Rules

## Purpose

Comprehensive guidelines for working with the WordPress REST API, including creating custom endpoints, extending existing endpoints, authentication, and API best practices.

## Key Principles

### 1. RESTful Design

Follow REST principles for endpoint design and HTTP method usage.

### 2. Authentication

Always implement proper authentication and permission checks.

### 3. Validation

Validate all input data before processing.

### 4. Error Handling

Return appropriate HTTP status codes and error messages.

## Common Patterns

### Registering Custom Endpoints

```php
<?php
/**
 * Register custom REST API endpoint
 */
function my_plugin_register_endpoints() {
    register_rest_route( 'my-plugin/v1', '/items', array(
        'methods'             => 'GET',
        'callback'            => 'my_plugin_get_items',
        'permission_callback' => '__return_true', // Public endpoint
    ) );
    
    register_rest_route( 'my-plugin/v1', '/items', array(
        'methods'             => 'POST',
        'callback'            => 'my_plugin_create_item',
        'permission_callback' => 'my_plugin_create_item_permissions',
        'args'                => array(
            'title' => array(
                'required'          => true,
                'validate_callback' => function( $param, $request, $key ) {
                    return is_string( $param );
                },
                'sanitize_callback' => 'sanitize_text_field',
            ),
            'content' => array(
                'required'          => true,
                'validate_callback' => function( $param, $request, $key ) {
                    return is_string( $param );
                },
                'sanitize_callback' => 'sanitize_textarea_field',
            ),
        ),
    ) );
    
    register_rest_route( 'my-plugin/v1', '/items/(?P<id>\d+)', array(
        'methods'             => 'GET',
        'callback'            => 'my_plugin_get_item',
        'permission_callback' => '__return_true',
        'args'                => array(
            'id' => array(
                'validate_callback' => function( $param, $request, $key ) {
                    return is_numeric( $param );
                },
            ),
        ),
    ) );
    
    register_rest_route( 'my-plugin/v1', '/items/(?P<id>\d+)', array(
        'methods'             => 'PUT',
        'callback'            => 'my_plugin_update_item',
        'permission_callback' => 'my_plugin_update_item_permissions',
        'args'                => array(
            'id' => array(
                'validate_callback' => function( $param, $request, $key ) {
                    return is_numeric( $param );
                },
            ),
            'title' => array(
                'sanitize_callback' => 'sanitize_text_field',
            ),
        ),
    ) );
    
    register_rest_route( 'my-plugin/v1', '/items/(?P<id>\d+)', array(
        'methods'             => 'DELETE',
        'callback'            => 'my_plugin_delete_item',
        'permission_callback' => 'my_plugin_delete_item_permissions',
        'args'                => array(
            'id' => array(
                'validate_callback' => function( $param, $request, $key ) {
                    return is_numeric( $param );
                },
            ),
        ),
    ) );
}
add_action( 'rest_api_init', 'my_plugin_register_endpoints' );
```

### Endpoint Callbacks

```php
<?php
/**
 * GET /items - Get all items
 */
function my_plugin_get_items( $request ) {
    // Get query parameters
    $page = $request->get_param( 'page' ) ? absint( $request->get_param( 'page' ) ) : 1;
    $per_page = $request->get_param( 'per_page' ) ? absint( $request->get_param( 'per_page' ) ) : 10;
    $search = $request->get_param( 'search' );
    
    // Prepare args
    $args = array(
        'post_type'      => 'item',
        'posts_per_page' => $per_page,
        'paged'          => $page,
        'post_status'    => 'publish',
    );
    
    if ( $search ) {
        $args['s'] = sanitize_text_field( $search );
    }
    
    // Get items
    $query = new WP_Query( $args );
    $items = array();
    
    foreach ( $query->posts as $post ) {
        $items[] = array(
            'id'      => $post->ID,
            'title'   => $post->post_title,
            'content' => $post->post_content,
            'date'    => $post->post_date,
            'author'  => get_the_author_meta( 'display_name', $post->post_author ),
        );
    }
    
    // Prepare response
    $response = rest_ensure_response( $items );
    
    // Add pagination headers
    $response->header( 'X-WP-Total', $query->found_posts );
    $response->header( 'X-WP-TotalPages', $query->max_num_pages );
    
    return $response;
}

/**
 * GET /items/:id - Get single item
 */
function my_plugin_get_item( $request ) {
    $id = $request->get_param( 'id' );
    
    $post = get_post( $id );
    
    if ( empty( $post ) || $post->post_type !== 'item' ) {
        return new WP_Error(
            'not_found',
            __( 'Item not found.', 'my-plugin' ),
            array( 'status' => 404 )
        );
    }
    
    $data = array(
        'id'      => $post->ID,
        'title'   => $post->post_title,
        'content' => $post->post_content,
        'date'    => $post->post_date,
        'author'  => array(
            'id'   => $post->post_author,
            'name' => get_the_author_meta( 'display_name', $post->post_author ),
        ),
        'meta'    => array(
            'custom_field' => get_post_meta( $post->ID, 'custom_field', true ),
        ),
    );
    
    return rest_ensure_response( $data );
}

/**
 * POST /items - Create item
 */
function my_plugin_create_item( $request ) {
    // Get parameters (already sanitized by args)
    $title = $request->get_param( 'title' );
    $content = $request->get_param( 'content' );
    
    // Create post
    $post_id = wp_insert_post( array(
        'post_title'   => $title,
        'post_content' => $content,
        'post_type'    => 'item',
        'post_status'  => 'publish',
        'post_author'  => get_current_user_id(),
    ) );
    
    if ( is_wp_error( $post_id ) ) {
        return new WP_Error(
            'create_failed',
            __( 'Failed to create item.', 'my-plugin' ),
            array( 'status' => 500 )
        );
    }
    
    // Get created item
    $post = get_post( $post_id );
    
    $data = array(
        'id'      => $post->ID,
        'title'   => $post->post_title,
        'content' => $post->post_content,
        'date'    => $post->post_date,
    );
    
    $response = rest_ensure_response( $data );
    $response->set_status( 201 ); // Created
    
    return $response;
}

/**
 * PUT /items/:id - Update item
 */
function my_plugin_update_item( $request ) {
    $id = $request->get_param( 'id' );
    
    $post = get_post( $id );
    
    if ( empty( $post ) || $post->post_type !== 'item' ) {
        return new WP_Error(
            'not_found',
            __( 'Item not found.', 'my-plugin' ),
            array( 'status' => 404 )
        );
    }
    
    // Prepare update data
    $update_data = array(
        'ID' => $id,
    );
    
    if ( $request->has_param( 'title' ) ) {
        $update_data['post_title'] = $request->get_param( 'title' );
    }
    
    if ( $request->has_param( 'content' ) ) {
        $update_data['post_content'] = $request->get_param( 'content' );
    }
    
    // Update post
    $result = wp_update_post( $update_data );
    
    if ( is_wp_error( $result ) ) {
        return new WP_Error(
            'update_failed',
            __( 'Failed to update item.', 'my-plugin' ),
            array( 'status' => 500 )
        );
    }
    
    // Get updated item
    $post = get_post( $id );
    
    $data = array(
        'id'      => $post->ID,
        'title'   => $post->post_title,
        'content' => $post->post_content,
        'date'    => $post->post_date,
    );
    
    return rest_ensure_response( $data );
}

/**
 * DELETE /items/:id - Delete item
 */
function my_plugin_delete_item( $request ) {
    $id = $request->get_param( 'id' );
    
    $post = get_post( $id );
    
    if ( empty( $post ) || $post->post_type !== 'item' ) {
        return new WP_Error(
            'not_found',
            __( 'Item not found.', 'my-plugin' ),
            array( 'status' => 404 )
        );
    }
    
    // Delete post
    $result = wp_delete_post( $id, true ); // true = force delete, skip trash
    
    if ( ! $result ) {
        return new WP_Error(
            'delete_failed',
            __( 'Failed to delete item.', 'my-plugin' ),
            array( 'status' => 500 )
        );
    }
    
    return rest_ensure_response( array(
        'deleted' => true,
        'id'      => $id,
    ) );
}
```

### Permission Callbacks

```php
<?php
/**
 * Permission callback for creating items
 */
function my_plugin_create_item_permissions( $request ) {
    // Check if user is logged in
    if ( ! is_user_logged_in() ) {
        return new WP_Error(
            'rest_forbidden',
            __( 'You must be logged in.', 'my-plugin' ),
            array( 'status' => 401 )
        );
    }
    
    // Check user capability
    if ( ! current_user_can( 'edit_posts' ) ) {
        return new WP_Error(
            'rest_forbidden',
            __( 'You do not have permission to create items.', 'my-plugin' ),
            array( 'status' => 403 )
        );
    }
    
    return true;
}

/**
 * Permission callback for updating items
 */
function my_plugin_update_item_permissions( $request ) {
    $id = $request->get_param( 'id' );
    
    if ( ! is_user_logged_in() ) {
        return new WP_Error(
            'rest_forbidden',
            __( 'You must be logged in.', 'my-plugin' ),
            array( 'status' => 401 )
        );
    }
    
    // Check if user can edit this specific post
    if ( ! current_user_can( 'edit_post', $id ) ) {
        return new WP_Error(
            'rest_forbidden',
            __( 'You do not have permission to edit this item.', 'my-plugin' ),
            array( 'status' => 403 )
        );
    }
    
    return true;
}

/**
 * Permission callback for deleting items
 */
function my_plugin_delete_item_permissions( $request ) {
    $id = $request->get_param( 'id' );
    
    if ( ! is_user_logged_in() ) {
        return new WP_Error(
            'rest_forbidden',
            __( 'You must be logged in.', 'my-plugin' ),
            array( 'status' => 401 )
        );
    }
    
    if ( ! current_user_can( 'delete_post', $id ) ) {
        return new WP_Error(
            'rest_forbidden',
            __( 'You do not have permission to delete this item.', 'my-plugin' ),
            array( 'status' => 403 )
        );
    }
    
    return true;
}
```

### Extending Existing Endpoints

```php
<?php
/**
 * Add custom fields to post response
 */
function my_plugin_add_custom_fields_to_post() {
    register_rest_field(
        'post', // Post type
        'custom_field', // Field name in API
        array(
            'get_callback'    => 'my_plugin_get_custom_field',
            'update_callback' => 'my_plugin_update_custom_field',
            'schema'          => array(
                'description' => __( 'Custom field description.', 'my-plugin' ),
                'type'        => 'string',
                'context'     => array( 'view', 'edit' ),
            ),
        )
    );
}
add_action( 'rest_api_init', 'my_plugin_add_custom_fields_to_post' );

/**
 * Get custom field value
 */
function my_plugin_get_custom_field( $object, $field_name, $request ) {
    return get_post_meta( $object['id'], 'custom_field', true );
}

/**
 * Update custom field value
 */
function my_plugin_update_custom_field( $value, $object, $field_name ) {
    if ( ! $value || ! is_string( $value ) ) {
        return;
    }
    
    return update_post_meta( $object->ID, 'custom_field', sanitize_text_field( $value ) );
}

/**
 * Add author name to post response
 */
function my_plugin_add_author_name() {
    register_rest_field(
        array( 'post', 'page' ),
        'author_name',
        array(
            'get_callback' => function( $object ) {
                return get_the_author_meta( 'display_name', $object['author'] );
            },
            'schema'       => array(
                'description' => __( 'Author display name.', 'my-plugin' ),
                'type'        => 'string',
                'context'     => array( 'view', 'edit' ),
            ),
        )
    );
}
add_action( 'rest_api_init', 'my_plugin_add_author_name' );
```

### Authentication

#### Application Passwords (Built-in)

```bash
# Use application passwords (WordPress 5.6+)
curl -X GET https://example.com/wp-json/wp/v2/posts \
  --user "username:xxxx xxxx xxxx xxxx xxxx xxxx"
```

#### JWT Authentication (Plugin Required)

```php
<?php
/**
 * Custom JWT authentication example
 * Requires JWT authentication plugin
 */

// Generate token endpoint
register_rest_route( 'my-plugin/v1', '/token', array(
    'methods'             => 'POST',
    'callback'            => 'my_plugin_generate_token',
    'permission_callback' => '__return_true',
    'args'                => array(
        'username' => array(
            'required' => true,
        ),
        'password' => array(
            'required' => true,
        ),
    ),
) );

function my_plugin_generate_token( $request ) {
    $username = $request->get_param( 'username' );
    $password = $request->get_param( 'password' );
    
    $user = wp_authenticate( $username, $password );
    
    if ( is_wp_error( $user ) ) {
        return new WP_Error(
            'invalid_credentials',
            __( 'Invalid username or password.', 'my-plugin' ),
            array( 'status' => 401 )
        );
    }
    
    // Generate token (example - use proper JWT library)
    $token = wp_generate_password( 32, false );
    set_transient( 'api_token_' . $user->ID, $token, HOUR_IN_SECONDS );
    
    return rest_ensure_response( array(
        'token'      => $token,
        'user_id'    => $user->ID,
        'user_email' => $user->user_email,
    ) );
}
```

#### Custom Nonce Authentication

```php
<?php
/**
 * Nonce authentication for logged-in users
 */
function my_plugin_verify_nonce( $request ) {
    $nonce = $request->get_header( 'X-WP-Nonce' );
    
    if ( ! $nonce ) {
        return new WP_Error(
            'missing_nonce',
            __( 'Nonce is required.', 'my-plugin' ),
            array( 'status' => 401 )
        );
    }
    
    if ( ! wp_verify_nonce( $nonce, 'wp_rest' ) ) {
        return new WP_Error(
            'invalid_nonce',
            __( 'Invalid nonce.', 'my-plugin' ),
            array( 'status' => 401 )
        );
    }
    
    return true;
}
```

### Making API Requests (JavaScript)

```javascript
// GET request
fetch('/wp-json/my-plugin/v1/items')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error('Error:', error));

// GET with parameters
const params = new URLSearchParams({
    page: 1,
    per_page: 10,
    search: 'query'
});

fetch(`/wp-json/my-plugin/v1/items?${params}`)
    .then(response => response.json())
    .then(data => console.log(data));

// POST request with authentication
fetch('/wp-json/my-plugin/v1/items', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-WP-Nonce': wpApiSettings.nonce // WordPress nonce
    },
    body: JSON.stringify({
        title: 'New Item',
        content: 'Item content'
    })
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('Error:', error));

// Using wp.apiFetch (in WordPress admin)
wp.apiFetch({
    path: '/my-plugin/v1/items',
    method: 'POST',
    data: {
        title: 'New Item',
        content: 'Item content'
    }
}).then(data => {
    console.log(data);
}).catch(error => {
    console.error(error);
});

// PUT request
fetch('/wp-json/my-plugin/v1/items/123', {
    method: 'PUT',
    headers: {
        'Content-Type': 'application/json',
        'X-WP-Nonce': wpApiSettings.nonce
    },
    body: JSON.stringify({
        title: 'Updated Title'
    })
})
.then(response => response.json())
.then(data => console.log(data));

// DELETE request
fetch('/wp-json/my-plugin/v1/items/123', {
    method: 'DELETE',
    headers: {
        'X-WP-Nonce': wpApiSettings.nonce
    }
})
.then(response => response.json())
.then(data => console.log(data));
```

### Making API Requests (PHP)

```php
<?php
/**
 * Make internal API request
 */
function my_plugin_internal_request() {
    $request = new WP_REST_Request( 'GET', '/my-plugin/v1/items' );
    $request->set_query_params( array(
        'per_page' => 5,
    ) );
    
    $response = rest_do_request( $request );
    $server = rest_get_server();
    $data = $server->response_to_data( $response, false );
    
    return $data;
}

/**
 * Make external API request using wp_remote_get
 */
function my_plugin_external_request() {
    $response = wp_remote_get( 'https://example.com/wp-json/wp/v2/posts' );
    
    if ( is_wp_error( $response ) ) {
        return false;
    }
    
    $body = wp_remote_retrieve_body( $response );
    $data = json_decode( $body, true );
    
    return $data;
}

/**
 * Make authenticated request
 */
function my_plugin_authenticated_request() {
    $args = array(
        'headers' => array(
            'Authorization' => 'Bearer ' . get_option( 'api_token' ),
        ),
    );
    
    $response = wp_remote_get( 'https://example.com/wp-json/custom/v1/data', $args );
    
    if ( is_wp_error( $response ) ) {
        return false;
    }
    
    return json_decode( wp_remote_retrieve_body( $response ), true );
}
```

### Response Schemas

```php
<?php
/**
 * Define response schema
 */
function my_plugin_item_schema() {
    $schema = array(
        '$schema'    => 'http://json-schema.org/draft-04/schema#',
        'title'      => 'item',
        'type'       => 'object',
        'properties' => array(
            'id' => array(
                'description' => __( 'Unique identifier for the item.', 'my-plugin' ),
                'type'        => 'integer',
                'context'     => array( 'view', 'edit' ),
                'readonly'    => true,
            ),
            'title' => array(
                'description' => __( 'The title of the item.', 'my-plugin' ),
                'type'        => 'string',
                'context'     => array( 'view', 'edit' ),
                'required'    => true,
            ),
            'content' => array(
                'description' => __( 'The content of the item.', 'my-plugin' ),
                'type'        => 'string',
                'context'     => array( 'view', 'edit' ),
            ),
            'date' => array(
                'description' => __( 'The date the item was created.', 'my-plugin' ),
                'type'        => 'string',
                'format'      => 'date-time',
                'context'     => array( 'view', 'edit' ),
                'readonly'    => true,
            ),
        ),
    );
    
    return $schema;
}

/**
 * Use schema in endpoint
 */
register_rest_route( 'my-plugin/v1', '/items/(?P<id>\d+)', array(
    array(
        'methods'             => 'GET',
        'callback'            => 'my_plugin_get_item',
        'permission_callback' => '__return_true',
    ),
    'schema' => 'my_plugin_item_schema',
) );
```

## Best Practices

### DO's

1. **Use Namespace and Version**: my-plugin/v1 format
2. **Validate Input**: Always validate parameters
3. **Sanitize Data**: Sanitize all input
4. **Check Permissions**: Implement proper permission callbacks
5. **Return Appropriate Status Codes**: Use correct HTTP codes
6. **Use WP_Error**: For error responses
7. **Document Endpoints**: Provide clear documentation
8. **Use Schemas**: Define response schemas
9. **Paginate Results**: For large data sets
10. **Cache Responses**: When appropriate
11. **Version Your API**: Allow for breaking changes
12. **Test Endpoints**: Test all scenarios
13. **Rate Limiting**: Implement if needed
14. **Log Errors**: Track API errors
15. **CORS Headers**: Configure when needed

### DON'Ts

1. **Don't Skip Validation**: Security risk
2. **Don't Trust Input**: Always validate
3. **Don't Expose Sensitive Data**: Filter responses
4. **Don't Use Direct Database Access**: Use WP functions
5. **Don't Skip Permission Checks**: Security risk
6. **Don't Return All Data**: Paginate large results
7. **Don't Forget Error Handling**: Handle all errors
8. **Don't Use Wrong HTTP Methods**: Follow REST standards
9. **Don't Hardcode URLs**: Use proper URL building
10. **Don't Skip Authentication**: For protected endpoints

## Cursor-Specific Instructions

When prompting Cursor AI for REST API development:

```
"Create a WordPress REST API endpoint for [describe functionality]. Include proper
validation, sanitization, permission checks, error handling, and follow REST API
best practices."

"Build custom REST API endpoints to [describe purpose]. Use appropriate HTTP
methods, validate input, check user permissions, return proper status codes, and
include response schemas."

"Extend the WordPress REST API for [post type/functionality]. Add custom fields,
implement proper get and update callbacks, validate data, and maintain security."
```

## Common Pitfalls

1. **Missing permission callbacks**: Security vulnerability
2. **Not validating input**: Data integrity issues
3. **Wrong HTTP status codes**: Confusing responses
4. **Exposing sensitive data**: Privacy/security risk
5. **Not sanitizing input**: SQL injection risk
6. **Missing error handling**: Poor user experience
7. **No rate limiting**: Resource exhaustion
8. **Overly complex responses**: Performance issues
9. **Not versioning API**: Breaking changes affect clients
10. **Missing documentation**: Hard to use API

## Resources

- [REST API Handbook](https://developer.wordpress.org/rest-api/)
- [REST API Reference](https://developer.wordpress.org/rest-api/reference/)
- [Extending the REST API](https://developer.wordpress.org/rest-api/extending-the-rest-api/)
- [REST API Authentication](https://developer.wordpress.org/rest-api/using-the-rest-api/authentication/)
- [Global Parameters](https://developer.wordpress.org/rest-api/using-the-rest-api/global-parameters/)
- [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

---

**Remember**: A well-designed API is secure, well-documented, and follows REST principles. Always prioritize security and validation.

