# Debugging in WordPress

# Learning Objectives

Upon completion of this lesson the participant will be able to:

## Outline

- What is debugging?
- Debugging PHP
- Enabling debugging
- Debugging with error_log
- Using the SAVEQUERIES constant
- Using debugging plugins

## Introduction

Hey there, and welcome to Learn WordPress.

In this tutorial, you're going to learn how about debugging in WordPress. 

You'll learn how to enable the built-in WordPress debugging options and how they work, as well as some plugins that can help you debug your page requests. 

## What is debugging?

Debugging is the process of finding and fixing errors in your code. 

As the two primary programming languages of WordPress are PHP and JavaScript, you need to be able to debug both. 

With JavaScript code, which is executed in the browser, it's fairly straightforward to use to console.log() to write messages to the browser console for the purposes of testing and debugging. 

PHP on the other hand, is executed on the server and so you need ways to find out what's happening when things go wrong.

There are a few third party tools you can use for advanced debugging, like Xdebug or Ray, but for the purposes of this tutorial, you'll learn about options that are specific to WordPress, and require no additional software. 

## Debugging PHP

In WordPress, during any WordPress request lifecycle, the wp_debug_mode function is run to set up the debugging environment. This function is located in the wp-includes/load.php file.

If you look at this code, you can see that if the WP_DEBUG constant is set to true, then it sets the PHP error reporting level to E_ALL, which means turn on all error reporting.

Additionally, if WP_DEBUG_DISPLAY is set to true, then it sets the display_errors PHP ini setting to 1, which means turn on display these errors on screen.

Finally, if WP_DEBUG_LOG is set to true, then it sets the error_log PHP ini setting to the wp-content/debug.log file. It is also possible to configure a custom debug.log file location, other than the default.

If this log file is enabled, it will set the PHP log_errors setting to 1, and set the error_log setting to the path of the log file, meaning that all errors will be logged to this file.

Using this knowledge, you can configure your wp-config.php file to enable WordPress debugging.

## Enabling debugging

To enable debugging, open the wp-config.php file and scroll down to where the WP_DEBUG constant is set. 

You can update that section to look like this:

```php
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_DISPLAY', false );
define( 'WP_DEBUG_LOG', true );
```

This configuration will:
1. Enable debugging
2. Disable displaying errors on screen
3. Enable logging errors to the wp-content/debug.log file

Depending on your personal preference, you can enable displaying the errors on screen, but this can lead to the errors either being missed, or overlaying other important content on screen, which is not ideal. 

Additionally, if you're ever debugging an issue on a production site, you don't want to display errors on screen, as this can lead to sensitive information being displayed to the user.

To see this in action, let's look at an example. 

Let's say you've developed a plugin with the following code:

```php
<?php
/**
 * Plugin Name: WP Learn Debugging
 * Plugin Description: A plugin to learn about debugging in WordPress.
 * Plugin URI: https://learn.wordpress.org
 * Version: 1.0.0
 */

/**
 * Set up the required form submissions table
 */
register_activation_hook( __FILE__, 'wp_learn_setup_table' );
function wp_learn_setup_table() {
	global $wpdb;
	$table_name = $wpdb->prefix . 'form_submissions';

	$sql = "CREATE TABLE $table_name (
	  id mediumint(9) NOT NULL AUTO_INCREMENT,
	  name varchar (100) NOT NULL,
	  email varchar (100) NOT NULL,
	  PRIMARY KEY  (id)
	)";

	require_once( ABSPATH . 'wp-admin/includes/upgrade.php' );
	dbDelta( $sql );
}

/**
 * Register the REST API GET route
 */
add_action( 'init', 'wp_learn_register_routes' );
function wp_learn_register_routes() {
	register_rest_route(
		'wp-learn-form-submissions-api/v1',
		'/form-submissions/',
		array(
			'methods'             => 'GET',
			'callback'            => 'wp_learn_get_form_submissions',
			'permission_callback' => '__return_true'
		)
	);
}

/**
 * Fetch the form submissions for the REST API GET Route
 *
 * @return array|object|stdClass[]|null
 */
function wp_learn_get_form_submissions() {
	global $wpdb;
	$table_name = $wpdb->prefix . 'form_submission';

	$results = $wpdb->get_results( "SELECT * FROM $table_name" );

	return $results;
}
```

This plugin creates a form submissions table in the database when it is activated, and then registers a REST API GET route to fetch the form submissions from the database.

For the purposes of testing, you've manually inserted a few records in the form_submissions table.

However, if you visit the REST API GET route, you don't see any form submissions, so you need to start looking for bugs in your code.

By simply enabling debugging, any errors in your code are automatically logged to the wp-content/debug.log file. So if you take a look, you'll see an errors have been logged.

```
[02-Jun-2023 13:51:41 UTC] PHP Notice:  Function register_rest_route was called <strong>incorrectly</strong>. REST API routes must be registered on the <code>rest_api_init</code> action. Please see <a href="https://wordpress.org/documentation/article/debugging-in-wordpress/">Debugging in WordPress</a> for more information. (This message was added in version 5.1.0.) in /home/ubuntu/wp-local-env/sites/learnpress/wp-includes/functions.php on line 5865

```

In this case, two errors are being reported. First us a PHP Notice triggered by WordPress, which is caused by hooking the wp_learn_register_routes funtion on the wrong action. Second is an error related to the database query being run to fetch the form submissions.

It looks like it is querying the table form_submission, not form_submissions.

Once you fix this errors, and visit the REST API GET route, you'll see the form submissions are returned.

## Debugging with error_log

In addition to logging errors to the debug.log file, you can also log messages or variables to the debug.log file using the PHP error_log function. This function accepts 

For example, if you wanted to log a sql query being rung to the debug.log file, you could use the following code:

```php
error_log( $wpdb->last_query );
```

If you refresh the request and view the debug.log file, you see the query being logged to the file. 

```
[02-Jun-2023 13:55:35 UTC] SELECT * FROM wp_form_submissions
```

## Using the SAVEQUERIES constant

In addition to logging the last query, you can also log all queries that are run during a WordPress request lifecycle. 

To do this, you can enable the SAVEQUERIES constant in your wp-config.php file.

```php
define( 'SAVEQUERIES', true );
```

Once you've enabled this constant, you can log all queries by using the following code:

```php
error_log( print_r( $wpdb->queries, true ) );
```

This uses the error_log function combined with the PHP print_r function to log the $wpdb->queries array to the debug.log file. 

This array contains all the queries that have been run during the WordPress request lifecycle, and is only available if the SAVEQUERIES constant is enabled.

Using debugging plugins

In addition to using the built-in debugging tools, there are also a few plugins that can help with debugging.

Query Monitor

The first plugin is Query Monitor, which is a debugging plugin that adds a debug menu to the admin bar, and displays information about the current page request. 

It includes information about the database queries that have been run, the current HTTP Request, the hooks and actions that have been run, loaded scripts and styles, and much more. 

Debug Bar

The second plugin is Debug Bar, which also adds a debug menu to the admin bar, and displays debugging information about the current page request. It focuses mostly on the database queries that have been run, includes data on the WP Query object as well as the HTTP Request, and includes info on the Object cache.

And that wraps up this tutorial on debugging in WordPress. For more information and debugging options, check out the [Debugging in WordPress](https://wordpress.org/documentation/article/debugging-in-wordpress/) page in the WordPress developer documentation. 

Happy debugging.

