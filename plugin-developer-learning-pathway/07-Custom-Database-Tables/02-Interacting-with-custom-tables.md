# Interacting with Custom Tables

Once you have your own custom table in the WordPress database, you need to know how to interact with it. In this lesson, we’ll cover how to safely query custom tables by using the `global $wpdb` object. You'll also learn how to insert, update, and delete data while following best practices to [avoid common security vulnerabilities](https://learn.wordpress.org/lesson/securely-developing-plugins-and-themes/).

A key aspect of interacting with any database is to ensure the data you send and retrieve is secure and validated. One of the most dangerous threats to your plugin’s security is SQL injection. Let's take a look at how that happens and what you can do to safeguard your tables against it.

## Protecting Against SQL Injections

SQL injection is a security vulnerability where attackers manipulate SQL queries by injecting malicious code into user input fields. If unsanitized input values are used in queries directly, then an attacker could execute arbitrary SQL commands which compromises your data.

![xkcd comic 327](xkcd_exploits_of_a_mom.png)

Consider this example where user input is inserted directly into a query without validation:

```php
global $wpdb;
$name = $_GET['name'];

$results = $wpdb->get_results( "SELECT * FROM {$wpdb->prefix}reviews WHERE customer_name = '$name'" );
```

In this example, an attacker could supply a value like `John'; DROP TABLE wp_reviews;--`, which would delete the entire custom table! Let's see how we can prevent this from happening.

### Using Prepared Statements

To protect against SQL injections, always use **prepared statements**. Prepared statements separate SQL logic from data, ensuring that user inputs are properly sanitized and escaped before being inserted into the query. In WordPress, the [`$wpdb->prepare()`](https://developer.wordpress.org/reference/classes/wpdb/prepare/) method is provided for this purpose. It's used similarly to PHP's [`sprintf()`](https://www.php.net/sprintf) and [`vsprintf()`](https://www.php.net/vsprintf) functions where you use placeholders in your SQL query and pass an array of the corresponding values.

Here’s how you can rewrite the previous query to safely handle user input:

```php
global $wpdb;
$name = $_GET['name'];

$prepared_query = $wpdb->prepare( 
    "SELECT * FROM {$wpdb->prefix}reviews WHERE customer_name = %s", 
    $name 
);

$results = $wpdb->get_results( $prepared_query );
```

In this example:
- `%s` is a placeholder for a string.
- The `prepare()` method ensures that the `$name` value is properly escaped before being included in the SQL query.

Notice that we do not keep the quotes around the `%s` placeholder value. By using the appropriate placeholder, the `prepare()` method automatically handles that for us. The following placeholders per data type are:

- `%d` (integer)
- `%f` (float)
- `%s` (string)
- `%i` (identifier, e.g. table/field names, [introduced in WordPress 6.2.0](https://make.wordpress.org/core/2022/10/08/escaping-table-and-field-names-with-wpdbprepare-in-wordpress-6-1/))

### Validating Data

In addition to properly escaping dynamic values used in SQL queries, it's also important to validate the data. Even honest users can enter incorrect data, so it's important for your plugin to check the data before using it. This improves data integrity and it also gives your plugin a chance to alert users of what's expected if certain criteria isn't met.

Some common [data validation techniques](https://developer.wordpress.org/apis/security/data-validation/) are:

- **Safelist** – Accept data only if it matches a known, trusted value. (e.g. selecting a color option)
- **Blocklist** – Reject data if it matches a known, untrusted value. (e.g. rejecting profanity)
- **Format Detection** – Test that the data matches an accepted format. (e.g. formatting phone numbers)

When comparing untrusted data against the safelist, it’s important to [**use strict type checking**](https://www.php.net/manual/en/language.operators.comparison.php). Otherwise, an attacker could craft input [in a way that will pass validation](https://www.php.net/manual/en/language.types.type-juggling.php) against a safelist, but still have a malicious effect.

```php
$untrusted_input = '1 malicious string';  // Will evaluate to integer 1 during loose comparisons.
$safe_values     = array( 1, 5, 7 ); // Integer 1 is an allowed value.

if ( in_array( $untrusted_input, $safe_values, true ) ) {  // `true` enables strict type checking.
    echo 'Success!';
} else {
    wp_die( 'The value you submitted is not allowed.' ); // The provided string is not allowed!
}
```

## Inserting Data

To store information in your custom table, use the [`$wpdb->insert()`](https://developer.wordpress.org/reference/classes/wpdb/insert/) method to add records to the table. This method helps you securely add data while ensuring input is sanitized and properly formatted for the database.

For example, suppose you’re building a plugin that records customer reviews. Each review includes a customer name, the review text, and a star rating from 1 to 5. The table schema could look like this:

```php
global $wpdb;
$charset_collate = $wpdb->get_charset_collate();

$sql = "CREATE TABLE {$wpdb->prefix}reviews (
  id mediumint(9) NOT NULL AUTO_INCREMENT,
  review_time datetime DEFAULT CURRENT_TIMESTAMP NOT NULL,
  customer_name tinytext NOT NULL,
  review_text text NOT NULL,
  star_rating tinyint(1) NOT NULL,
  PRIMARY KEY (id)
) {$charset_collate};";
```

When a user submits a product review, you can record a new entry securely into the custom table like this:

```php
global $wpdb;

// Sanitize input submitted by the user.
$customer_name = sanitize_text_field( $_POST['customer_name'] );
$review_text   = sanitize_textarea_field( $_POST['review_text'] );
$star_rating   = intval( $_POST['star_rating'] );

// Check that the star rating is between 1 and 5.
if ( $star_rating < 1 || $star_rating > 5 ) {
  wp_die( 'Invalid star rating. Please submit a value between 1 and 5.' );
}

// Add the new entry into the custom table.
$wpdb->insert(
  "{$wpdb->prefix}reviews",
  array(
    'customer_name' => $customer_name,
    'review_text'   => $review_text,
    'star_rating'   => $star_rating,
  ),
  array(
    '%s', // customer_name : string.
    '%s', // review_text : string.
    '%d', // star_rating : integer.
  )
);
```

As you can see, this example demonstrates the data security techniques we previously discussed:

1. **Sanitizing Inputs:** We use [`sanitize_text_field()`](https://developer.wordpress.org/reference/functions/sanitize_text_field/) and [`sanitize_textarea_field()`](https://developer.wordpress.org/reference/functions/sanitize_textarea_field/) to clean user input. The star rating is also sanitized by using [`intval()`](https://www.php.net/manual/en/function.intval.php) which casts it to an integer.

2. **Validating Data:** We ensure that the star rating falls between 1 and 5. If the input fails, the [`wp_die()`](https://developer.wordpress.org/reference/functions/wp_die/) function halts execution and shows an informative error message.

3. **Using** `$wpdb->insert()`**:** This method inserts data into the custom table using three parameters:
   - The table name.
   - An associative array with column names as keys and the corresponding values to be inserted.
   - An optional array specifying the corresponding data type placeholders for each value.

Since `$wpdb->insert()` handles the underlying SQL `INSERT` query, you do not need to use `$wpdb->prepare()` in this instance.

## Querying Data

After successfully storing reviews in the custom table, you’ll want to retrieve them for display on your website. To do this safely and efficiently, you can use the [`$wpdb->get_results()`](https://developer.wordpress.org/reference/classes/wpdb/get_results/) method, which allows you to fetch multiple records at once.

To display the 10 most recent reviews, you can query them from the custom table like this:

```php
global $wpdb;

// Retrieve the 10 latest reviews.
$reviews = $wpdb->get_results( 
  "SELECT customer_name, review_text, star_rating, review_time 
   FROM {$wpdb->prefix}reviews 
   ORDER BY review_time DESC
   LIMIT 10"
);

// Display each customer review.
if ( ! empty( $reviews ) ) {
  foreach ( $reviews as $review ) {
    echo '<div class="review">';
    echo '<h3>' . esc_html( $review->customer_name ) . '</h3>';
    echo '<p>' . esc_html( $review->review_text ) . '</p>';
    echo '<p>Rating: ' . intval( $review->star_rating ) . ' / 5</p>';
    echo '<small>Submitted on: ' . esc_html( $review->review_time ) . '</small>';
    echo '</div>';
  }
} else {
  echo '<p>No reviews found.</p>';
}
```

As you can see, the `get_results()` method returns an array of objects where each object is a single row retrieved from the table. Alternatively, you can provide a second parameter to the function to specify the desired output type. For example, to retrieve rows as associative arrays instead of objects, you can use the `ARRAY_A` constant.

To select values from the database by running generic queries, you can also use any of the following methods within the `wpdb` class depending on the return type you're wanting:

- [`get_var()`](https://developer.wordpress.org/reference/classes/wpdb/get_var/) – Returns a single value.
- [`get_row()`](https://developer.wordpress.org/reference/classes/wpdb/get_row/) – Returns an entire, single row in the desired output type.
- [`get_col()`](https://developer.wordpress.org/reference/classes/wpdb/get_col/) – Returns an entire, single column as a one-dimensional array.

### Dynamic Queries

Let's assume you now want to add a new feature that allows users to filter reviews by their star rating. The SQL query will ultimately be the same except for which star rating the user selects.

Here's how to securely query results dynamically based on the user's request:

```php
$star_rating = intval( $_GET['star_rating'] );

$prepared_query = $wpdb->prepare(
  "SELECT customer_name, review_text, star_rating, review_time 
   FROM {$wpdb->prefix}reviews 
   WHERE star_rating = %d 
   ORDER BY review_time DESC",
  $star_rating
);

$reviews = $wpdb->get_results( $prepared_query );
```

Since user-submitted data is incorporated into this SQL query, we must use the `prepare()` method to prevent SQL injection vulnerabilities. The placeholder `%d` is used because the `star_rating` field is an integer type.

## Updating Data

Sometimes you may need to update records in your custom table, such as letting users correct their customer name or modify their review’s star rating. For this, you can use the [`$wpdb->update()`](https://developer.wordpress.org/reference/classes/wpdb/update/) method. This method allows you to modify specific rows while ensuring that the data is securely handled.

For example, let’s say a user decides to change the review that they initially submitted. You can securely update the review's star rating and text in the custom table by using the following code:

```php
global $wpdb;

// Sanitize and validate the inputs.
$review_id   = intval( $_POST['review_id'] );
$new_rating  = intval( $_POST['new_rating'] );
$new_text    = sanitize_textarea_field( $_POST['new_text'] );

// Ensure the new star rating is between 1 and 5.
if ( $new_rating < 1 || $new_rating > 5 ) {
  wp_die( 'Invalid star rating. Please provide a value between 1 and 5.' );
}

// Update the star rating and review text in the custom table.
$updated = $wpdb->update(
  "{$wpdb->prefix}reviews",       // Table to update.
  array(
    'star_rating' => $new_rating,
    'review_text' => $new_text,
  ),                              // Column values to set.
  array( 'id' => $review_id ),    // WHERE clause column values.
  array( '%d', '%s' ),            // Format for the updated column values.
  array( '%d' )                   // Format for the WHERE clause column values.
);

// Check if the update was successful.
if ( false === $updated ) {
  wp_die( 'Error updating the review. Please try again.' );
} else {
  echo 'Review updated successfully!';
}
```

As always, remember to sanitize and validate the user's request. If it passes validation, then the `$wpdb->update()` method updates the `wp_reviews` custom table by using the following 5 parameters:

1. The name of the table to be updated.
2. An array of column names and their associated values to be set.
3. An array of column names and their associated values to determine which matching records in the table should be updated.
   - When passing multiple column-value pairs, the clauses are joined using logical `AND`.
4. An array of formatting placeholders for the column values to be set.
   - `'%d'` for the `star_rating` integer, then `'%s'` for the `review_text` string
5. An array of formatting placeholders for the `WHERE` clause column values.
   - `'%d'` for the review's `id` integer

The `$wpdb->update()` method returns either the number of rows updated or false if an error occurred. If the update query fails, then we display an appropriate error message to inform the user. Otherwise, we notify the user that the update was successful.

## Deleting Data

In some scenarios, users may need to delete outdated or incorrect records from the custom table. To handle such cases, WordPress provides the [`$wpdb->delete()`](https://developer.wordpress.org/reference/classes/wpdb/delete/) method to safely remove specific rows. Just like other database operations, it’s important to validate and sanitize any data before using it to delete a record.

Let’s say you want to allow users to delete their product reviews. Here’s how you can implement it:

```php
global $wpdb;

// Sanitize and validate the review ID.
$review_id = intval( $_POST['review_id'] );

if ( $review_id <= 0 ) {
  wp_die( 'Invalid review ID.' );
}

// Delete the review from the custom table.
$deleted = $wpdb->delete(
  "{$wpdb->prefix}reviews",    // Table from which to delete.
  array( 'id' => $review_id ), // WHERE clause to specify the row(s) to delete.
  array( '%d' )                // Format for the WHERE clause values.
);

// Check if the deletion was successful.
if ( false === $deleted ) {
  wp_die( 'Error deleting the review. Please try again.' );
} elseif ( 0 === $deleted ) {
  echo 'No matching review found to delete.';
} else {
  echo 'Review deleted successfully!';
}
```

The `$wpdb->delete()` method simply accepts 3 parameters which should look familiar from the previous section:

1. The table from which to delete matching rows.
2. An array of column names and their associated values to determine which matching rows in the table should be deleted.
	 - If we were to pass multiple column-value pairs, the clauses would be joined using logical `AND`.
3. An array of formatting placeholders for the `WHERE` clause column values.
	 - `'%d'` for the review's `id` integer

Finally, the value returned from `$wpdb->delete()` is either the number of rows deleted from the specified table or `false`. Because of this, remember to use the strict comparison operator `===` in PHP to differentiate database errors from `0` rows being deleted.

### Use Caution with Deletions

Deleting data is irreversible, so it’s important to consider:

- Asking for user confirmation before deleting records to ensure users know the action cannot be undone.
- Logging deletion actions, such as who deleted what and when, for historical reference.
- Offering a *soft delete* feature as an alternative, such as marking a record as inactive, to keep the data archived.

## Performing Generic Queries for Advanced Use Cases

While the `wpdb` class's methods like `insert()`, `update()`, and `delete()` cover many common database operations, advanced use cases may require more flexibility. For tasks such as bulk inserts, transactional operations, or table operations, you can use the [`query()`](https://developer.wordpress.org/reference/classes/wpdb/query/) method to run any valid SQL statement. However, you must always carefully consider the risks involved when writing your own custom queries and `prepare()` the dynamic data in your custom queries accordingly.

Let's suppose we want to add a feature that allows a customer to submit a collection of reviews for all the items they recently purchased. Using a single bulk insert statement for this use case will grant better performance and data consistency. Here's how to use the `$wpdb->query()` method to implement this feature:

```php
global $wpdb;

// Collect the user's input.
$customer_name = sanitize_text_field( $_POST['customer_name'] ); // The customer's name for all the reviews.
$reviews       = $_POST['reviews']; // An array of reviews with 'review_text' and 'star_rating'.

// Begin the SQL query for a bulk insert.
$sql = "INSERT INTO {$wpdb->prefix}reviews (customer_name, review_text, star_rating) VALUES ";

$data           = array();
$values_clauses = array();

// Loop through each review and build the query dynamically.
foreach ( $reviews as $i => $review ) {
  
    // Sanitize the review's data to be inserted.
    $review_text = sanitize_textarea_field( $review['review_text'] );
    $star_rating = intval( $review['star_rating'] );

    // Validate the star rating.
    if ( $star_rating < 1 || $star_rating > 5 ) {
        $review_number = $i + 1;
        wp_die( 'Error: Invalid star rating for review ' . intval( $review_number ) . '. Please correct the review and try again.' );
    }

    // Collect the review's data for INSERT with the accompanying placeholders.
    $data[] = $customer_name;
    $data[] = $review_text;
    $data[] = $star_rating;
  
    $values_clauses[] = "(%s, %s, %d)";
}

// Join all placeholders and complete the SQL statement.
$sql .= implode( ', ', $values_clauses );

// Execute the query with the prepared statement using the associated data.
$result = $wpdb->query( $wpdb->prepare( $sql, $data ) );

if ( false === $result ) {
    wp_die( 'Error: Failed to insert reviews.' );
} elseif ( count( $reviews ) !== $result ) {
    echo esc_html( 'Submitted ' . count( $reviews ) . ' of ' . intval( $result ) . ' reviews successfully.' );
} else {
    echo 'All reviews submitted successfully!'; 
}
```

Let's walk through this code step-by-step:

1. First, we collect the user's input from the form submission and initialize a string to begin composing the bulk insertion SQL query.
2. Next, we loop through the reviews to sanitize and validate the `review_text` and `star_rating` values for each one.
3. Once sanitized and validated in the loop, the data for each review record to be inserted is collected along with the associated VALUES clause containing the related placeholders.
4. After collecting each VALUES clause, they are joined together using a comma to complete the SQL statement.
5. The completed SQL statement is then safely formatted using the `$wpdb->prepare()` method with the sanitized and validated data for each review, and the prepared query is then executed using the `$wpdb->query()` method.
6. Since the query is for an `INSERT` statement, the `$wpdb->query()` method returns the number of rows affected or `false` if a database error occurred which we then use to inform the user.
   - For SQL statements that affect entire tables instead of specific rows, such as `TRUNCATE`, the `$wpdb->query()` method will simply return `true` on success.

By using our own custom query and the `$wpdb->query()` method, we're able to optimize the efficiency of our queries and even offer more advanced experiences to our users.

## Summary

To learn more about the properties and methods made available in the `wpdb` class, you can read [the `wpdb` class reference for developers](https://developer.wordpress.org/reference/classes/wpdb/).
