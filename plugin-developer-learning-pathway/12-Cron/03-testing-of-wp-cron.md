# Testing of WP Cron

Like any code, it is important to test it.

However, as the way to execute that code is not straight forward, we will detail how we can do this.

First, we need to differentiate two things:
- The logic executed inside the event.
- The scheduling from the action itself.

## Testing the event

### The lazy way

The event itself is actually an action like another, so it is possible to test it by executing it at WordPress initialization, so it runs each time the page is reloaded:

```php
function my_plugin_run_my_action() {
    do_action('init', 'my_plugin_my_action');
}

add_action('init', 'my_plugin_run_my_action');
```

Then, once this is done, it is actually really easy to test the logic within that method but don't forget to remove that code after you finished.

### The clean way

It is possible to use [WP-CLI](https://wp-cli.org/) to make it execute the schedule for you.

For that first you need to install WP-CLI and configure it.

Then it will be possible to run any cron job with the command `wp cron event run {job name}`.

For example, with the following cron job:

```php
add_action( 'wp_learn_trigger_event', 'wp_learn_trigger_event' );

function wp_learn_trigger_event() {
    // event logic here
}

add_action( 'init', 'wp_learn_schedule_event' );

function wp_learn_schedule_event() {
    wp_schedule_event( time(), 'hourly', 'wp_learn_trigger_event' );
}

```

It is possible to execute it using the following command `wp cron event run wp_learn_trigger_event`.

## Testing the scheduling

Then, if we want to test the full scheduling, we can use [WP-CLI](https://wp-cli.org/).

To test the scheduling, we need to use the following command `wp cron event list` which going to provide the list of the next scheduled events:

```shell
+-------------------+---------------------+---------------------+------------+
| hook              | next_run_gmt        | next_run_relative   | recurrence |
+-------------------+---------------------+---------------------+------------+
| wp_version_check  | 2016-05-31 22:15:13 | 11 hours 57 minutes | 12 hours   |
| wp_update_plugins | 2016-05-31 22:15:13 | 11 hours 57 minutes | 12 hours   |
| wp_update_themes  | 2016-05-31 22:15:14 | 11 hours 57 minutes | 12 hours   |
+-------------------+---------------------+---------------------+------------+
```

That you can check in a second if everything is scheduled according to plan.


