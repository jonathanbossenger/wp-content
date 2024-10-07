
# Determining the correct hook

## Introduction

At the day of today, WordPress has more than 3000 hooks inside its core.

As you might guess, there is a hook for everything, but the main issue will be to be fine what is the name of that hook.

In that lesson, we will learn some of the important hooks, but most importantly, we will learn how to find a new hook using the WordPress documentation.


## Finding the correct type

The first thing to understand is the type of hook you will be searching for.

For that, you need to understand the difference between an action and a filter.

WordPress plugin programming is an event-based development, and due to that it is respecting its rules and principles.

Once of them is called command query responsibility segregation or in short CQRS.

It states that any operation can be divided into two types of operations:
- The command that will be changing the state without returning any result.
- The query that will fetch data without changing the application state.

As you might get it, WordPress is also respecting that rule:
- An action is the equivalent of the command, and this is the type of hook you will be searching to modify the state from WordPress.
- A filter will be the equivalent of the query, and it is the type of hook you will be searching for changing a value used by WordPress.

## Finding a hook inside WordPress documentation 

It is possible to find a list of all WordPress hooks at this page: https://developer.wordpress.org/reference/hooks/

Then from there, it is possible to either navigate manually or using the search bar.

## Important hooks

All hooks are not equally used, and some are used more often than others.

In this part, we're going to pay attention to some of the most important ones.

### The plugins_loaded hook

This event fires once all plugins are loaded.

You often want to register your plugin initialization to that hook because all other plugin code will be loaded which gonna help to check for third party compatibilities or if your plugin relies on another plugin logic to execute.

### The init hook

This event is executed when WordPress finished loading.

It is an important hook as it is where you want to add logic to configure tools as such as registering cron.

Another hook `admin_init` exists to register logic to add only to administration pages.

### The wp_enqueue_scripts hook

When you need to enqueue new styles or script, it is possible to use this hook.

Inside the callbacks from this hook it is possible to use the functions `wp_enqueue_style` and `wp_enqueue_script`.

```php

function enqueue_assets() {
    wp_enqueue_style( 'my-theme', 'style.css', false );

    wp_enqueue_script( 'my-js', 'filename.js', false );
}

add_action('wp_enqueue_scripts', 'enqueue_assets');

```

## What about outside WordPress core?

Welcome to the jungle. 

When you go out of WordPress, you then have to rely on what the plugin maintainer has to offer to you and in most cases, that will be nothing.

Due to that I will advise you to directly search for `do_action` and `apply_filters` inside their codebase to find hooks available on a community plugin.
