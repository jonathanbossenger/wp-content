## How to Check User Capabilities in WordPress

## Introduction

WordPress provides a User Roles and Capabilities system to manage permissions across different users. Each role has specific capabilities that define what actions a user can perform on the site, such as editing posts, managing plugins, or moderating comments. Checking user capabilities is essential for restricting actions based on roles and ensuring security. 

As a developer, you may need to check a user's capabilities before executing specific functionality.  These capabilities are assigned based on user roles (e.g., Administrator, Editor, Author, Contributor, Subscriber). Knowing how to check user capabilities is essential for developers who need to control access to features, content, or settings.

## Understanding User Capabilities in WordPress

WordPress assigns capabilities to roles using a predefined system. Some common capabilities include:

- edit_posts – Allows editing posts.
- publish_posts – Allows publishing posts.
- delete_users – Allows deleting users.
- manage_options – Allows managing site settings.

Each role has a specific set of capabilities, which can be modified using custom code or plugins like User Role Editor.

## Methods to Check User Capabilities

### Using current_user_can() Function

The current_user_can() function is the most commonly used method to check if the current user has a specific capability. It returns true if the user has the capability and false otherwise.

```php
if ( current_user_can( 'edit_posts' ) ) {
    echo "You have permission to edit posts.";
} else {
    echo "Access denied.";
}
```
👉 Use Case: Restrict content editing to users with the required permissions.

Example: Check if the user can manage site settings

````php
if ( current_user_can( 'manage_options' ) ) {
    // Show admin settings
    echo "You have access to manage site settings.";
}
````
👉 Use Case: Display an admin settings panel only for users with the manage_options capability.


### Using user_can() 

The user_can() function allows you to check the capabilities of a specific user (not just the current user). You need to pass the user ID or object and the capability you want to check.

Example: Check Capabilities for a Specific User

```php
$user_id = 2; // Example user ID
if (user_can($user_id, 'delete_posts')) {
    echo 'This user can delete posts.';
} else {
    echo 'This user cannot delete posts.';
}
```
👉 Use Case: Verify permissions for another user in a multi-user environment.


### Using wp_get_current_user()

The wp_get_current_user() function retrieves the current user object, which you can use to check roles and capabilities directly.

Example: Check the Current User’s Role

```php
$current_user = wp_get_current_user();
if (in_array('editor', $current_user->roles)) {
    echo 'You are an Editor.';
}
```

### Checking Multiple Capabilities

You can check multiple capabilities using logical operators:

```php
if ( current_user_can( 'edit_posts' ) && current_user_can( 'publish_posts' ) ) {
    echo 'User can edit and publish posts.';
} else {
    echo 'User lacks the required permissions.';
}

```

### Customizing User Capabilities

If you need to add or remove capabilities for a role, use the add_cap() and remove_cap() functions.

Example: Add a Custom Capability to Editors

```php
function add_custom_capability() {
    $role = get_role( 'editor' );
    if ( $role ) {
        $role->add_cap( 'manage_options' ); // Allow editors to manage settings
    }
}
add_action( 'init', 'add_custom_capability' );
```

👉 Use Case: Give editors permission to change settings without making them administrators.


### Debugging User Capabilities

If you want to debug a user’s roles and capabilities, you can retrieve and print their data:

```php
$user = wp_get_current_user();
echo '<pre>';
print_r( $user->roles );
print_r( $user->allcaps );
echo '</pre>';
```
This will display all roles and capabilities assigned to the logged-in user.

### Example: Restrict Access to a Custom Admin Page

Let’s say you’ve created a custom admin page and want to restrict access to users with the manage_options capability (typically available to Administrators).

````php
function my_custom_admin_page() {
    if (!current_user_can('manage_options')) {
        wp_die('You do not have sufficient permissions to access this page.');
    }
    // Your custom admin page code here
    echo 'Welcome to the custom admin page!';
}
add_action('admin_menu', 'my_custom_admin_page');
````

## Conclusion

Checking user capabilities in WordPress ensures that only authorized users can perform certain actions. The current_user_can() function is the easiest way to check permissions for the logged-in user, while has_cap() allows checking capabilities for any specific user.

By properly managing user capabilities, you can enhance security and ensure that users have the appropriate level of access within your WordPress site.