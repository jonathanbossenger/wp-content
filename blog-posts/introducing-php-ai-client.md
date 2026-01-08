AI generated draft

---

# Introducing the PHP AI Client for WordPress

**By [Your Name]**

*Categories: Common APIs, Plugins, Updates*

Since the [WordPress Core AI team](https://wordpress.org/news/2025/05/announcing-the-formation-of-the-wordpress-ai-team/) was formed, one of the most impactful projects they've been developing is the [PHP AI Client](https://github.com/WordPress/php-ai-client). This provider-agnostic SDK is part of the broader [AI Building Blocks for WordPress](https://make.wordpress.org/ai/2025/07/17/ai-building-blocks/) initiative, and it's designed to transform how developers integrate AI capabilities into their plugins and themes.

If you've ever wanted to add AI features to your WordPress plugin but felt overwhelmed by the complexity of managing multiple AI providers, API credentials, and inconsistent interfaces—this post is for you.

## The problem it solves

Today, every WordPress plugin with AI features essentially rebuilds the same infrastructure: provider integrations, API key management, response normalization, and error handling. Users enter the same API credentials multiple times across different plugins and deal with inconsistent experiences. When providers change their APIs, every plugin breaks separately.

The PHP AI Client centralizes this complexity. One integration point handles all providers. One credential system serves all plugins. When providers update or new ones emerge, the changes happen once and benefit everyone.

Think of it like the WordPress HTTP API, but for AI. Instead of every plugin implementing its own HTTP client, WordPress provides a unified interface. The PHP AI Client does the same for AI providers like Anthropic, Google, and OpenAI.

## Two packages, maximum flexibility

The PHP AI Client SDK consists of two Composer packages:

1. **[PHP AI Client](https://github.com/WordPress/php-ai-client)** — A platform-agnostic PHP library that provides the unified AI interface. It's WordPress-agnostic and benefits the broader PHP ecosystem.

2. **[WordPress AI Client](https://github.com/WordPress/wp-ai-client)** — A WordPress-specific wrapper that adds an admin settings screen for API credentials, REST API endpoints, and a JavaScript API for client-side usage.

For most WordPress developers, you'll want to use the WordPress AI Client package, which provides the full experience with admin UI and WordPress-specific integrations.

## Getting started

### Installation

Install the WordPress AI Client via Composer:

```bash
composer require wordpress/wp-ai-client
```

### Configuration

First, initialize the client on the WordPress `init` hook:

```php
add_action( 'init', function() {
    if ( class_exists( 'WordPress\AI_Client\AI_Client' ) ) {
        WordPress\AI_Client\AI_Client::init();
    }
});
```

Then configure your API credentials by navigating to **Settings > AI Credentials** in the WordPress Admin and entering your API keys for the providers you intend to use.

## Building AI-powered features

The SDK provides a fluent `Prompt_Builder` interface that makes constructing AI requests intuitive. Let's look at some practical examples.

### Text generation

Here's a simple example of generating text:

**PHP:**
```php
use WordPress\AI_Client\AI_Client;

$response = AI_Client::prompt()
    ->with_user_message( 'Suggest three blog post titles about WordPress development.' )
    ->for_text_generation()
    ->generate();

$generated_text = $response->get_text();
```

**JavaScript:**
```javascript
const response = await wp.ai.prompt()
    .withUserMessage( 'Suggest three blog post titles about WordPress development.' )
    .forTextGeneration()
    .generate();

const generatedText = response.getText();
```

### Image generation

Creating images is just as straightforward:

**PHP:**
```php
$response = AI_Client::prompt()
    ->with_user_message( 'A serene mountain landscape at sunset' )
    ->for_image_generation()
    ->generate();

$image_url = $response->get_image();
```

### Advanced usage with JSON output

For structured responses, you can request JSON output:

**PHP:**
```php
$response = AI_Client::prompt()
    ->with_user_message( 'List 5 SEO improvements for this content...' )
    ->with_temperature( 0.7 )
    ->for_text_generation()
    ->with_json_output()
    ->generate();
```

### Multimodal output

The SDK even supports multimodal operations—generating both text and images in a single request:

**PHP:**
```php
$response = AI_Client::prompt()
    ->with_user_message( 'Create a product description and promotional image for...' )
    ->for_text_generation()
    ->for_image_generation()
    ->generate();

$text = $response->get_text();
$image = $response->get_image();
```

## Best practices

### Let the SDK choose the model

By default, the SDK automatically selects a suitable model based on your prompt's requirements and the configured providers on the site. This makes your plugin **provider-agnostic**, allowing it to work on any site regardless of which AI provider the admin has configured:

```php
// This works regardless of whether the site uses OpenAI, Anthropic, or Google
$response = AI_Client::prompt()
    ->with_user_message( 'Summarize this article...' )
    ->for_text_generation()
    ->generate();
```

### Feature detection

Before exposing AI features in your plugin, always check if the required capabilities are available:

**PHP:**
```php
$prompt = AI_Client::prompt()
    ->with_user_message( 'Generate an image...' )
    ->for_image_generation();

if ( $prompt->is_supported_for_image_generation() ) {
    // Show the AI image generation feature to users
    $response = $prompt->generate();
} else {
    // Hide or disable the feature gracefully
}
```

This ensures your plugin gracefully adapts to environments that may not have AI providers configured.

### Error handling

The SDK offers two approaches for handling errors:

**Exception-based (default):**
```php
try {
    $response = AI_Client::prompt()
        ->with_user_message( 'Generate content...' )
        ->for_text_generation()
        ->generate();
} catch ( Exception $e ) {
    // Handle the error
    error_log( 'AI generation failed: ' . $e->getMessage() );
}
```

**WP_Error-based:**
```php
$response = AI_Client::prompt_with_wp_error()
    ->with_user_message( 'Generate content...' )
    ->for_text_generation()
    ->generate();

if ( is_wp_error( $response ) ) {
    // Handle the WP_Error
}
```

## Ideas for AI-powered features

The real magic happens when you think beyond chat interfaces. Here are some features that become surprisingly simple to build with the PHP AI Client:

- **Alternative text suggestions** — Select a paragraph block and generate multiple rephrasing options
- **Automatic featured images** — Generate images based on post content
- **SEO optimization** — Analyze content and suggest keyword improvements
- **Audio versions** — Generate natural-sounding narration of blog posts
- **Smart media editing** — Add or remove objects from images in the Media Library
- **Content summarization** — Automatically create excerpts or social media snippets

All of these features would have been complex to build before. With the PHP AI Client, some are nearly trivial to implement.

## Connecting to the bigger picture

The PHP AI Client is designed to work seamlessly with the other AI Building Blocks:

- **[Abilities API](https://developer.wordpress.org/news/2025/11/introducing-the-wordpress-abilities-api/)** — Register your plugin's AI-powered features as discoverable abilities
- **[MCP Adapter](https://github.com/WordPress/mcp-adapter)** — Allow external AI assistants like Claude and ChatGPT to interact with your WordPress site
- **[AI Experiments Plugin](https://github.com/WordPress/ai)** — Test and explore all the building blocks together

## Where to learn more and get involved

- Explore the [PHP AI Client GitHub repository](https://github.com/WordPress/php-ai-client)
- Check out the [WordPress AI Client GitHub repository](https://github.com/WordPress/wp-ai-client)
- Read the [architecture documentation](https://github.com/WordPress/php-ai-client/blob/trunk/docs/ARCHITECTURE.md)
- Join discussions in the [#ai-building-blocks](https://wordpress.slack.com/archives/C08TJ8BPULS) channel on WordPress Slack
- Follow the [Core AI team blog](https://make.wordpress.org/ai/) for updates

## This is just the beginning

The PHP AI Client puts the user first by being provider-agnostic—site administrators choose their preferred AI provider, configure credentials in one place, and all compatible plugins just work. For developers, it means you can focus on building amazing AI features instead of wrestling with infrastructure.

As AI becomes increasingly essential to WordPress, this SDK provides a sustainable foundation that's designed to grow with emerging capabilities—new modalities, advanced features, and novel deployment models.

I'm excited to see what you build with it.

*Props to [@juanmaguitar](https://profiles.wordpress.org/juanmaguitar/), [@bph](https://profiles.wordpress.org/bph/), and [@felixarntz](https://profiles.wordpress.org/flavor/) for feedback and review on this article.*

---

**Categories:** Common APIs, Plugins, Updates

**Tags:** PHP AI Client, AI, Block development, Extenders

---

This blog post follows the same engaging style as the Abilities API introduction, with:
- A clear explanation of the problem being solved
- Practical code examples in both PHP and JavaScript
- Best practices guidance
- Creative ideas for implementation
- Links to resources and community involvement opportunities