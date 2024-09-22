# Activation and Deactivation methods

## Introduction

Depending on your plugin's functionality, you may need to perform certain tasks when the plugin is either activated or deactivated. 

In this lesson, you're going to learn about two methods that WordPress provides to implement these tasks.

## Example code

In order to see these in action, let's take the Extra Option plugin from the Naming Collisions lesson. To keep things simple, we'll use the prefixed option.

If you don't have this code, you can create it by creating a plugin in your local WordPress installation using the following code.

```php
<?php
/**
 * Plugin Name: WP Learn Extra Content
 * Version: 1.0.0
 */

add_action( 'admin_init', 'wp_learn_extra_content_add_option' );

function wp_learn_extra_content_add_option() {
	add_settings_field( 'extra_option', 'Extra Option', 'wp_learn_extra_content_extra_option_field', 'general' );
	register_setting( 'general', 'wp_learn_extra_content_extra_option' );
}

function wp_learn_extra_content_extra_option_field() {
	echo '<input name="wp_learn_extra_content_extra_option" id="wp_learn_extra_content_extra_option" type="text" value="' . esc_html( get_option( 'wp_learn_extra_content_extra_option' ) ) . '" />';
}

add_filter( 'the_content', 'wp_learn_extra_content_add_extra_option' );
function wp_learn_extra_content_add_extra_option( $content ) {
	$extra_option = get_option( 'wp_learn_extra_content_extra_option' );
	if ( ! $extra_option ) {
		new WP_Error( 'wp_learn_extra_content_extra_option', 'Extra content is empty.' );

		return $content;
	}
	$content .= '<p>' . esc_html( $extra_option ) . '</p>';

	return $content;
}
```

Alternatively, you can download the plugin from the [GitHub repository](https://github.com/wptrainingteam/plugin-developer/blob/trunk/wp-learn-extra-content.zip), and install and activate it via your WordPress dashboard.

## Activation Method

In order to perform tasks when the plugin is activated, you can use the `register_activation_hook` function. 

This function takes two parameters: a path to the main plugin file and the function to be executed when the plugin is activated.

If you are calling this function from inside the plugin file, you can use the `__FILE__` constant to get the path to the file.

```php
register_activation_hook( __FILE__, 'activation_callback_function' );
```

Let's add a function to the plugin that will add a default value to the extra option when the plugin is activated.

```php
register_activation_hook( __FILE__, 'wp_learn_extra_content_activation' );
function wp_learn_extra_content_activation() {
    add_option( 'wp_learn_extra_content_extra_option', 'Default extra content' );
}
```

If you activate this plugin on a new WordPress install and browse to the General Settings page, you will see that the Extra Option field has a default value of 'Default extra content'.

> [!NOTE]
> If you have already created and used the Extra Content plugin in your local WordPress install, you might need to remove the option from your options table.

## Deactivation Method

Similar to the activation method, you can use the `register_deactivation_hook` function to perform tasks when the plugin is deactivated.

You could use this method to remove the option from the options table when the plugin is deactivated.

```php
register_deactivation_hook( __FILE__, 'wp_learn_extra_content_deactivation' );
function wp_learn_extra_content_deactivation() {
    delete_option( 'wp_learn_extra_content_extra_option' );
}
```

Let's see this in action. First, take a look in the options table to see that the option is there. 

Then deactivate the plugin and check the options table again. You will see that the option has been removed.

## Plugin deactivation and data cleanup

It is typically not recommended to remove data when a plugin is deactivated. This is because the user may want to reactivate the plugin at a later date and expect the data to still be there. 

However, this does depend on the individual use case and requirements of the plugin's functionality. 

Generally though, data cleanup is to be done when the plugin is uninstalled, which you will learn about in the lesson on plugin Uninstall Methods.

## Different callback implementations

Your plugin's code structure will determine how you use the `register_activation_hook` and `register_deactivation_hook` functions. If your activation and deactivation functions are namespaced, remember to use the fully qualified function name.

```php
register_activation_hook( __FILE__, 'MyPlugin\activation_callback_function' );
```

If your functions are in a class, you can use the class method to call the functions.

```php
register_activation_hook( __FILE__, array( 'MyPlugin', 'activation_callback_function' ) );
``` 

Remember to also always pass the full path to the main plugin file as the first parameter. If you're not registering these hooks in the main plugin file, you'll need to use other methods to get the path to the main plugin file.

One way to do this is to use a constant in the main plugin file and then use that constant in the file where you're registering the hooks.

```php
// In the main plugin file
define( 'MY_PLUGIN_FILE', __FILE__ );
```

```php
// In the file where you're registering the hooks
register_activation_hook( MY_PLUGIN_FILE, 'activation_callback_function' );
```

## WP Dashboard only

One final note, the activation and deactivation hooks are only triggered when the plugin is activated or deactivated from the WordPress dashboard. 

For example, if the plugin files are deleted from the server, the deactivation hook will not be triggered.
