
## What is User Metadata?


User metadata in WordPress allows developers to store additional information about users beyond the default fields provided by WordPress. This metadata is stored in the wp_usermeta table and can be accessed, updated, and managed using built-in WordPress functions. This guide covers how to effectively work with user metadata in WordPress.


While WordPress stores basic user information (like username, email, and password) in the wp_users table, metadata is stored in the wp_usermeta table. This metadata can include anything from phone numbers and addresses to custom settings or preferences.

For example, if you’re building a membership site, you might use user metadata to store:

- Subscription status
- Payment history
- Profile preferences
- Social media links

## Understanding User Metadata

User metadata consists of key-value pairs stored in the WordPress database. This data can be used for storing user preferences, profile details, or any other custom information required for user management.

## How to Work with User Metadata in WordPress

There are two primary methods to manage user metadata in WordPress:

1. **Through the User Profile Screen**: Adding custom fields to the user's profile page within the WordPress admin dashboard.

To allow users or administrators to input and view custom metadata directly from the WordPress admin area, you can add custom fields to the user profile screen. This involves hooking into specific action hooks and defining functions to display and save the custom fields.

Example: Adding a Birthday Field

The following example demonstrates how to add a "Birthday" field to user profiles:

```php
/**
 * Display the birthday field on the user profile screen.
 *
 * @param WP_User $user The current WP_User object.
 */
function add_birthday_field_to_profile( $user ) {
    ?>
    <h3><?php esc_html_e( "Additional Information", "your-textdomain" ); ?></h3>
    <table class="form-table">
        <tr>
            <th><label for="birthday"><?php esc_html_e( "Birthday", "your-textdomain" ); ?></label></th>
            <td>
                <input type="date" name="birthday" id="birthday" value="<?php echo esc_attr( get_user_meta( $user->ID, 'birthday', true ) ); ?>" class="regular-text" /><br />
                <span class="description"><?php esc_html_e( "Please enter your birthday.", "your-textdomain" ); ?></span>
            </td>
        </tr>
    </table>
    <?php
}

/**
 * Save the birthday field value.
 *
 * @param int $user_id The ID of the user being saved.
 */
function save_birthday_field( $user_id ) {
    // Check if the current user has permission to edit the user.
    if ( ! current_user_can( 'edit_user', $user_id ) ) {
        return false;
    }
    // Update the user meta with the new birthday value.
    update_user_meta( $user_id, 'birthday', sanitize_text_field( $_POST['birthday'] ) );
}

// Add the birthday field to the user profile and edit screens.
add_action( 'show_user_profile', 'add_birthday_field_to_profile' );
add_action( 'edit_user_profile', 'add_birthday_field_to_profile' );

// Save the birthday field value when the user profile is updated.
add_action( 'personal_options_update', 'save_birthday_field' );
add_action( 'edit_user_profile_update', 'save_birthday_field' );
```

In this example, the add_birthday_field_to_profile function adds a date input field to the user profile page. The save_birthday_field function saves the inputted birthday to the user's metadata when the profile is updated. These functions are hooked into show_user_profile, edit_user_profile, personal_options_update, and edit_user_profile_update to ensure the field appears and saves correctly on both personal and admin profile pages.


2. **Programmatically**: Using WordPress functions to add, retrieve, update, or delete user metadata.

WordPress provides several functions to work with user metadata. Let’s explore how to use them effectively.


### Adding User Metadata

To add metadata for a user, use the add_user_meta() function. This function allows you to store additional data for a user by specifying a unique meta key.

```php
add_user_meta($user_id, 'favorite_color', 'blue', true);
```

- $user_id: The ID of the user.

- favorite_color: The meta key.

- blue: The value associated with the meta key.

- true: Optional parameter to prevent duplicate meta keys.

Example:

```php
function add_user_phone_number($user_id) {
    $phone_number = '123-456-7890'; // Example phone number
    add_user_meta($user_id, 'phone_number', $phone_number);
}
add_action('user_register', 'add_user_phone_number');
```

### Retrieving User Metadata

To retrieve metadata for a user, use the get_user_meta() function. This function takes the user ID and the meta key as parameters and returns the associated value.

```php
$favorite_color = get_user_meta($user_id, 'favorite_color', true);
echo 'Favorite Color: ' . $favorite_color;
```

- If the third parameter(optional) is set to true, it returns a single value.

- If set to false, it returns an array of values.

Example:

```php
function display_user_phone_number($user_id) {
    $phone_number = get_user_meta($user_id, 'phone_number', true);
    if ($phone_number) {
        echo 'Phone Number: ' . esc_html($phone_number);
    } else {
        echo 'No phone number found.';
    }
}
```

### Updating User Metadata

If you need to modify existing user metadata, use update_user_meta().

```php
update_user_meta($user_id, 'favorite_color', 'red');
```

If the meta key already exists, the function updates its value; otherwise, it creates a new entry.

Example:

```php
function update_user_phone_number($user_id) {
    $new_phone_number = '987-654-3210'; // New phone number
    update_user_meta($user_id, 'phone_number', $new_phone_number);
}
```

This function is useful for modifying existing metadata or adding new metadata if it doesn’t already exist.


### Deleting User Metadata

To delete metadata associated with a user, use delete_user_meta().

```php
delete_user_meta($user_id, 'favorite_color');
```

This function removes the metadata entry entirely from the database.

Example:

```php
function delete_user_phone_number($user_id) {
    delete_user_meta($user_id, 'phone_number');
}
```
You can also delete all metadata for a user by omitting the meta key:

```php
delete_user_meta($user_id); // Deletes all metadata for the user
```

## Use Cases for User Metadata

User metadata can be used for various purposes, such as:

- Storing additional profile fields (e.g., social media links, preferences).

- Managing user roles and permissions.

- Saving user settings and configurations.

- Customizing the user experience based on stored data.

## Best Practices for Working with User Metadata

- Sanitize and Validate Data: Always sanitize and validate user input to prevent security vulnerabilities.

- Use Unique Meta Keys: Ensure your meta keys are unique to avoid conflicts with other plugins or themes.

- Avoid Storing Large Data: Metadata is best suited for small pieces of data. For large datasets, consider using custom tables.

- Leverage Hooks and Filters: Use WordPress hooks and filters to modify metadata behavior without modifying core files.

## Conclusion

WordPress provides powerful functions to handle user metadata efficiently. By leveraging these functions, developers can enhance user management and extend the default WordPress user system with custom data. For more details, refer to the official WordPress Developer Handbook: [Working with User Metadata](https://developer.wordpress.org/plugins/users/working-with-user-metadata/).
