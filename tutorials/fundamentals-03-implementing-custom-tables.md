# WordPress Developer Fundamentals - Custom Database Tables

## Learning Objectives

Upon completion of this lesson the participant will be able to:

## Outline
1. Introduction
2. Where to find information
3. Creating Custom Database Tables
4. Creating the table
5. Inserting data
6. Updating data
7. Selecting data
8. Table updates
9. Cleaning up
10. Conclusion

## Introduction

Hey there, and welcome to Learn WordPress.

In this tutorial, you'll be learning about creating custom database tables for WordPress.

You will learn where to find information about creating custom database tables, how to create custom database tables, and how to interact with them.

## Why create custom database tables?

The default WordPress database schema is typically enough for all content types. The ability to register custom post types and use post meta usually covers most options.

However, in some cases, you may need to store data that doesn't fit into the default schema. 

For example, in an ecommerce store, a custom post type would work for products, as it has similar fields to a post; for example a title, image, and content. Any additional fields it might need can be stored as post meta. 

However, an order does not use the same fields as a post, and therefore it might be useful to store orders in a custom table.

For the occasions where you need to store data that doesn't fit into the default schema, you can create your own custom database tables.

## Where to find information

While the WordPress developer documentation does not include anything around custom database tables, there is an older version of the developer documentation called the Codex that does. You can find everything you need to know about custom database tables in the [Creating Tables with Plugins](https://codex.wordpress.org/Creating_Tables_with_Plugins ) page of the Codex.

## Creating Custom Database Tables

The first thing you'll notice is that this is typically done in a plugin. 

Additionally, it is possible, and recommended, to create custom tables when the plugin is activated, using the `register_activation_hook` [function](https://developer.wordpress.org/reference/functions/register_activation_hook/), and if you delete them to do that when the plugin is deactivated using the `register_deactivation_hook` [function](https://developer.wordpress.org/reference/functions/register_deactivation_hook/). This allows this functionality to be run once, and not every time the plugin is loaded.

## Creating the table

To create a custom table on plugin activation, you need to use a few things.

To start, create a function to manage the table creation:

```php
    function create_database_table() {
    }
```

Then, you need to use the `$wpdb` global [WordPress database object](https://developer.wordpress.org/reference/classes/wpdb/), as it contains all the methods you need to interact with the database. 

This will allow you to set up the new table name, using the WordPress database prefix.

```php
    function create_database_table() {
        global $wpdb;
    
        $table_name = $wpdb->prefix . 'custom_table';
    }
````

It will also allow you to access the `get_charset_collate` [method](https://developer.wordpress.org/reference/classes/wpdb/get_charset_collate/), which will return the correct character set and collation for the database.

```php
    $charset_collate = $wpdb->get_charset_collate();
```

To create a table, you need to know SQL to execute a SQL statement on the database. This is done via the `dbDelta` [function](https://developer.wordpress.org/reference/functions/dbdelta/). `dbDelta` is a function that is generally used during WordPress updates, if default WordPress tables need to be updated or change. It examines the current table structure, compares it to the desired table structure, and either adds or modifies the table as necessary.

In order to use `dbDelta`, you need to write your SQL statement in a specific way. 

You can read more about these requirements in the Creating or Updating the Table section of the [Codex](https://codex.wordpress.org/Creating_Tables_with_Plugins#Creating_or_Updating_the_Table) page.

Once you've created the SQL statement, you need to pass it to the `dbDelta` function. This is done by including the `wp-admin/includes/upgrade.php` file, which contains the function declaration.

```php
	function create_database_table() {
		global $wpdb;

		$table_name = $wpdb->prefix . 'custom_table';

		$charset_collate = $wpdb->get_charset_collate();

		$sql = "CREATE TABLE $table_name (
            id mediumint(9) NOT NULL AUTO_INCREMENT,
            time datetime DEFAULT '0000-00-00 00:00:00' NOT NULL,
            name tinytext NOT NULL,
            text text NOT NULL,
            url varchar(55) DEFAULT '' NOT NULL,
            PRIMARY KEY  (id)
	    ) $charset_collate;";

		require_once ABSPATH . 'wp-admin/includes/upgrade.php';
		dbDelta( $sql );
	}
```

In this example, a new table is being created called `custom_table`. It has 5 fields, an `id`, a `time`, a `name`, a `text`, and a `url`. The `id` is an auto incrementing integer, the `time` is a datetime field, the `name` is a tinytext field, the `text` is a text field, and the `url` is a varchar field. The `id` is the primary key.

Hooking this function into your plugin activation hook will ensure that the table is created when the plugin is activated.

```php
register_activation_hook( __FILE__, 'create_database_table' );
register_activation_hook( __FILE__, 'create_custom_database_table' );
```

## Inserting data 

It's also possible to use the plugin activate hook to insert data into your table on plugin activation.

To do this you can use the `insert` method of the `$wpdb` object, passing an array of field names and values.

Here is an example of what this could look like.

```php
    function insert_record_into_table(){
        global $wpdb;

        $table_name = $wpdb->prefix . 'custom_table';

        $wpdb->insert(
            $table_name,
            array(
                'time' => current_time( 'mysql' ),
                'name' => 'John Doe',
                'text' => 'Hello World!',
                'url'  => 'https://wordpress.org'
            )
        );
    }
```

[INSERT NEW RECORDING]

It would also be a good idea to check that the table exists before attempting to insert data into it. This can be done using the `get_var` method of the `$wpdb` object, passing a SQL statement to check if the table exists.

```php
    function insert_record_into_table(){
        global $wpdb;

        $table_name = $wpdb->prefix . 'custom_table';

        $table_exists = $wpdb->get_var( "SHOW TABLES LIKE '$table_name'" );

        if ( $table_exists ) {
            $wpdb->insert(
                $table_name,
                array(
                    'time' => current_time( 'mysql' ),
                    'name' => 'John Doe',
                    'text' => 'Hello World!',
                    'url'  => 'https://wordpress.org'
                )
            );
        }
    }
```

[END INSERT NEW RECORDING]

## Updating data

To update data in your custom table, use the `update` method of the `$wpdb` object, passing an array of field names and values, as well as an array of field names and values to match.

```php
    function update_record_in_table() {
        global $wpdb;

        $table_name = $wpdb->prefix . 'custom_table';

        $wpdb->update(
            $table_name,
            array(
                'time' => current_time( 'mysql' ),
                'name' => 'John Doe',
                'text' => 'Hello World!',
                'url'  => 'https://wordpress.org'
            ),
            array( 'id' => 1 )
        );
    }
```

## Selecting data

Selecting data from your custom table is done using the `get_results` method of the `$wpdb` object.

```php
    function select_records_from_table() {
        global $wpdb;

        $table_name = $wpdb->prefix . 'custom_table';

        $results = $wpdb->get_results( "SELECT * FROM $table_name" );

        foreach ( $results as $result ) {
            echo $result->name . ' ' . $result->text . '<br>';
        }
    }
```

By default, get_results will return an array of objects, which you can loop through and access the fields as properties.

## Table updates

It's also a good idea to store a db version number as a WordPress option, in case you ever need to update the table. While the process of doing this is outside the scope of this tutorial, the basic process is:

1. Store the version number in the options table
2. Create an upgrade routine that triggers when the plugin is updated, perhaps using something like https://developer.wordpress.org/reference/hooks/upgrader_process_complete/
3. Based on the version, upgrade the table, by creating a separate function which contains the full SQL statement to update the table
4. Once the upgrade has run, update the version number

[START RE-RECORD]

## Cleaning up

It's also possible to delete your custom tables. To do this, you can use the `query` method of the `$wpdb` object, passing a SQL statement to delete the table.

```
    function delete_table() {
        global $wpdb;

        $table_name = $wpdb->prefix . 'custom_table';

        $wpdb->query( "DROP TABLE IF EXISTS $table_name" );
    }
```

`query` will run any SQL query, but it's best to only use it for queries that don't insert or update data, as those functions include built-in sanitization.

You should try to never use the `query` method to insert or update data, but for whatever reason if you must use it to insert or update data, you should always use the [prepare](https://developer.wordpress.org/reference/classes/wpdb/prepare/) method on the SQL Query. This will ensure that the SQL query is sanitized to prevent any security vulnerabilities.

Depending on your requirements, or the requirements of your plugin's users, you could delete the table in two ways.

If your plugin users will not need the data in this table if they deactivate the plugin, you could trigger this on the plugin deactivation hook.

```
register_deactivation_hook( __FILE__, 'delete_table' );
```

However, if the data in that table is important, and your users might want to keep it, even if the plugin is deactivated, you could delete the table using one of the [two uninstall methods](https://developer.wordpress.org/plugins/plugin-basics/uninstall-methods/) available to plugins. For example, if you choose to use the register_uninstall_hook.

```
register_uninstall_hook( __FILE__, 'delete_table');
```

It's generally recommended to check with your user and then use one of the uninstall methods over plugin deactivation.

[END RE-RECORD]

## Conclusion

This tutorial only scratches the surface of what's possible with custom tables, but hopefully it gives you an idea of what's possible.

Happy coding. 