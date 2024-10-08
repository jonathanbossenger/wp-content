# Creating and Managing Custom Tables

As a plugin developer, knowing how to extend WordPress functionality beyond standard post types and taxonomies is a valuable skill to master.

In this lesson, we will explore how to create, manage, and delete custom database tables in WordPress. 

By the end of this lesson, you’ll understand why you might need a custom table, how to create and version the table schema, and how to safely remove a table when it’s no longer needed.

## Why Create a Custom Table?

In most cases, WordPress's default tables are already enough to store your plugin's data. You can use [custom post types](link to cpt lesson) to manage different types of content, store user meta to further personalize users' experiences, manage custom settings in the options table, and so much more!

Even still, there are times when creating a custom table can be more effective. When thinking about data your plugin will be managing, consider the following:

- **Performance**: If you’re dealing with large datasets or specific relationships between data points, custom tables can improve query performance by giving you control over the data schema and indexing behavior.
- **Data structure**: When your data doesn’t fit the traditional WordPress structure, it can be much easier to write queries if the data is stored in custom tables which are tailored to how the data will be used.
- **Scalability**: As your plugin and the surrounding application grow, you can easily add, update, or remove custom tables as needs change.

Now that you can identify when custom database tables are useful, let's learn how to add one!

In this example, you're going to create a custom table to store form submissions, which will have a field for the name and a message. 

## The `WPDB` class

To start, you need to first establish a connection to the WordPress database. Thankfully, WordPress already does this automatically by using the `WPDB` class.

The `WPDB` class is used to manage a database connection while also providing helpful methods to perform queries and retrieve data from the connected database in a structured way. 

During PHP execution, WordPress creates a `$wpdb` global object variable as an instance of this class by using the `DB_HOST`, `DB_NAME`, `DB_USER`, and `DB_PASSWORD` constants defined in your `wp-config.php` file.

Because the `$wpdb` object exists in the global namespace, you can access it in PHP like this:

```php
global $wpdb;
```

The `$wpdb` instance also contains crucial information about the WordPress database which you'll need to properly create and manage our own custom tables. Let's see how as we create our first one.

## Creating a Custom Table

To start, create a new plugin to store the custom table code. 

```
<?php
/**
 * Plugin Name: Form submissions
 * Version:     1.0.0
 * Author:      Learn WordPress
 * Author URI:  https: //learn.wordpress.org/
 *
 * @package wp-learn-form-submissions
 */
 ```

Then, register an activation hook for your plugin to create the custom table. Using this hook ensures the table immediately exists once your plugin is activated so that any following code can safely query against it.

When choosing a name for the custom table, be sure to use the configured table prefix by prepending the table name with the `$wpdb->prefix` object property. Also, use `$wpdb->get_charset_collate()` method to ensure the custom table's collation matches the rest of the tables in WordPress for optimum data integrity, compatibility, and query performance.

To create a custom table, use the `dbDelta()` function included in WordPress. This function handles both table creation and future updates to the table's structure.

```php
register_activation_hook( __FILE__, 'wp_learn_custom_table_create_submissions_table' );
function wp_learn_custom_table_create_submissions_table() {
	global $wpdb;
	$charset_collate = $wpdb->get_charset_collate();

	$sql = "CREATE TABLE {$wpdb->prefix}submissions (
	id mediumint(9) NOT NULL AUTO_INCREMENT,
	time datetime DEFAULT '0000-00-00 00:00:00' NOT NULL,
	name tinytext NOT NULL,
	message text NOT NULL,
	PRIMARY KEY  (id)
	) {$charset_collate};";

	require_once ABSPATH . 'wp-admin/includes/upgrade.php';
	dbDelta( $sql );
}
```

Notice that you must load the `wp-admin/includes/upgrade.php` file included in WordPress in order to use the `dbDelta()` function. WordPress only uses this function for update processes, so it is not always included like other PHP functions are in common execution.

The SQL query must follow the `dbDelta()` function's specific requirements, such as putting each field on its own line and putting two spaces after the words `PRIMARY KEY`. Please refer to the [*Creating Tables with Plugins* section of the WordPress Plugin Developer Handbook](https://developer.wordpress.org/plugins/creating-tables-with-plugins/#creating-or-updating-the-table) for all formatting requirements.

If you use this code in a custom plugin, activate it, and use a database tool, you'll see the new table listed in your local WordPress database, with the table columns you defined. 

## Updating the Table Schema

As your plugin evolves, you may need to modify the structure of your custom table. For example, let's say you realise you also want to offer the user an option to enter their site URL in the form submission.

The `dbDelta()` function can also help you modify existing tables without losing data. By following the previously mentioned formatting requirements, it can automatically figure out how to alter the installed table into the new schema.

Before we update the custom table's schema, though, let's ensure `dbDelta()` is only checking the schema when it makes sense.

### Table Versioning

When a plugin is updated, its activation function registered with `register_activation_hook()` is not called. This is why we'll need to use another hook to check if the database needs to be upgraded.

The `plugins_loaded` action hook is a great choice for this because it is triggered right after all active plugins have been loaded. This ensures the database tables are updated before any other code tries to query against them.

To track which database version is already installed, you can store a simple database version number for the plugin in the options table.

```php
$wp_learn_custom_table_db_version = '1.0';
```

```
add_action( 'plugins_loaded', 'wp_learn_custom_table_update_db_check' );
function wp_learn_custom_table_update_db_check() {
	global $wp_learn_custom_table_db_version;
	$current_custom_table_db_version = get_option( 'wp_learn_custom_table_db_version', '1.0' );
	if ( version_compare( $current_custom_table_db_version, $wp_learn_custom_table_db_version ) ) {
		wp_learn_custom_table_create_submissions_table();
	}
}
```

Whenever the plugin's table schema is changed in the code, the version number should be increased so that the tables are checked and updated by the `dbDelta()` function.

```php
$wp_learn_custom_table_db_version = '1.0';

register_activation_hook( __FILE__, 'wp_learn_custom_table_create_submissions_table' );
function wp_learn_custom_table_create_submissions_table() {
	global $wpdb;
	$charset_collate = $wpdb->get_charset_collate();

	$sql = "CREATE TABLE {$wpdb->prefix}submissions (
	id mediumint(9) NOT NULL AUTO_INCREMENT,
	time datetime DEFAULT '0000-00-00 00:00:00' NOT NULL,
	name tinytext NOT NULL,
	message text NOT NULL,
    url varchar(55) DEFAULT '' NOT NULL,
	PRIMARY KEY  (id)
	) {$charset_collate};";

	require_once ABSPATH . 'wp-admin/includes/upgrade.php';
	dbDelta( $sql );

	// Set or update the currently installed version.
	global $wp_learn_custom_table_db_version;
	update_option( 'wp_learn_custom_table_db_version', $wp_learn_custom_table_db_version );
}

add_action( 'plugins_loaded', 'wp_learn_custom_table_update_db_check' );
function wp_learn_custom_table_update_db_check() {
	global $wp_learn_custom_table_db_version;
	$current_custom_table_db_version = get_option( 'wp_learn_custom_table_db_version', '1.0' );
	if ( version_compare( $current_custom_table_db_version, $wp_learn_custom_table_db_version ) ) {
		wp_learn_custom_table_create_submissions_table();
	}
}
```

## Deleting a Custom Table

When a user deactivates or uninstalls your plugin, it’s good practice to clean up any custom tables you created to avoid cluttering the database. However, remember to only delete tables when you’re certain the user wants to remove all plugin data.

In most cases, plugin data is removed during uninstallation because it indicates the user no longer needs the plugin. As you learned in Module 1, you can use the `register_uninstall_hook()` function to do this.

You can execute the SQL query to delete the table by using the `$wpdb` global variable, which you'll learn more about in a future lesson.

```php
register_uninstall_hook( __FILE__, 'wp_learn_custom_table_delete_submissions_table' );
function wp_learn_custom_table_delete_submissions_table() {
	global $wpdb;
	$wpdb->query( "DROP TABLE IF EXISTS {$wpdb->prefix}submissions" );
	delete_option( 'wp_learn_custom_table_db_version' );
}
```

## Conclusion

Now you know how to create, update, and delete a custom table in the WordPress database.

For more information on working with custom tables, make sure to read the [Creating Tables with Plugins section of WordPress Plugin Handbook](https://developer.wordpress.org/plugins/creating-tables-with-plugins/). It's also useful to read the class reference for the `WPDB` class, as well as the function reference for `dbDelta`.