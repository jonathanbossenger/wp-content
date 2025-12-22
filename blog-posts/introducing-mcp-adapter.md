# From Abilities to AI Agents: Introducing the WordPress MCP Adapter

The [Abilities API introduced in WordPress 6.9](https://make.wordpress.org/core/2025/11/10/abilities-api-in-wordpress-6-9/) makes it possible to register WordPress functionality that is **standardized, discoverable, typed, and executable**. It provides a solid foundation on which WordPress developers can build and extend across the WordPress ecosystem.

It's also a major step in making WordPress ready for AI automation and workflows. With Abilities, WordPress is positioned to take advantage of any current and future developments in Generative AI. 

One of the biggest of these recent developments is the [Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro), or MCP. 

With MCP, it's possible to provide additional context to the models which power AI tools. Let's say you were using an AI tool to help you draft a report of all sales on your WordPress powered ecommerce site. Imagine if it was possible to give the AI secure access to all your orders for the year. If WordPress supported MCP, it would be.

Fortunately, the Core AI team have already thought of this, with the release of the [MCP Adapter](https://make.wordpress.org/ai/2025/07/17/mcp-adapter/). This adapter implements the Model Context Protocol in the scope of a WordPress site, and lets AI tools (like Claude Desktop, Claude Code, Cursor, and VS Code) **discover and call WordPress Abilities directly**.

So let's dive into the MCP Adapter, learn how to install and use it in your WordPress plugins and themes, expose your existing abilities as **MCP tools**, and connect AI clients to your WordPress enabled MCP site to make use of the new protocol.

## Quick recap: Abilities as the foundation

If this is the first time you're reading about WordPress Abilities, it might be worthwhile to read the Introducing the [WordPress Abilities API](https://developer.wordpress.org/news/2025/11/introducing-the-wordpress-abilities-api/) post. However, if you don't have the time to read that, here's a quick recap. 

The Abilities API gives WordPress a **first-class, cross-context functional API** that standardizes how core, plugins, and themes expose what they can do.

You define an ability once with:

- A unique name (`namespace/ability-name`)
- A typed **input schema** and **output schema**
- A **permission_callback** that enforces capabilities
- An **execute_callback** that performs the actual functionality

The functionality triggered in the `execute_callback` can be anything from fetching data, updating posts, running diagnostics, or any other discrete unit of work.

Once registered, that ability is discoverable and executable from PHP, JavaScript, and the REST API.

WordPress 6.9 ships with 3 default abilities:

- `core/get-site-info`: Returns site information configured in WordPress. By default, returns all fields, or optionally a filtered subset.
- `core/get-user-info`: Returns basic profile details for the current authenticated user to support personalization, auditing, and access-aware behavior.
- `core/get-environment-info`: Returns core details about the site's runtime context for diagnostics and compatibility (environment, PHP runtime, database server info, WordPress version).

While only a small set of Core Abilities, they provide a foundation you can use to test the MCP adapter. 

## What is the WordPress MCP Adapter?

The **WordPress MCP Adapter** is an official package in the _AI Building Blocks for WordPress_ initiative. Its job is to adapt Abilities registered by the **Abilities API** into the [primitives](https://modelcontextprotocol.io/docs/learn/architecture#primitives) supported by the **Model Context Protocol (MCP)** so that AI agents can discover and execute site functionality as **MCP tools** and read WordPress data as **MCP resources**

In practice, this means: **if your code already registers abilities, you are one step away from letting an AI agent use them.**

### A primer on MCP tools, resources, and prompts

The Model Context Protocol organizes interactions into three main primitives: **tools**, which are executable functions the AI calls to perform actions; **resources**, which are passive data sources (like files or database rows) the AI reads for context; and **prompts**, which are pre-configured templates to guide specific workflows.

With the MCP adapter, Abilities are generally exposed as **tools** because they represent executable logic—fetching data, updating posts, or running diagnostics. However, the adapter is flexible: if an Ability simply provides read-only data, such as a debug log or a static site configuration, it can also be configured as a **resource**, allowing the AI to ingest that information as background context without needing to actively "call" it.

## Installing the MCP Adapter

The quickest way to get started with the MCP Adapter is to download and install it as a plugin from the [Releases page of the GitHub repository](https://github.com/WordPress/mcp-adapter/releases).

Once the plugin is installed and activated, it will register a **default MCP server** named `mcp-adapter-default-server`, and three custom abilities. 

- `mcp-adapter/discover-abilities`
- `mcp-adapter/get-ability-info`
- `mcp-adapter/execute-ability`

These abilities are also automatically exposed as MCP tools:

- `mcp-adapter-discover-abilities`
- `mcp-adapter-get-ability-info`
- `mcp-adapter-execute-ability`

These three tools offer AI agents a [layered approach](https://engineering.block.xyz/blog/build-mcp-tools-like-ogres-with-layers) to accessing WordPress Abilities. Agents can **discover** which Abilities are available, **get** ability information, and **execute** abilities. 

## Enabling Abilities for the MCP Adapter default server

By default, Abilities are only available via the MCP Adapter default server if they are explicitly marked as public for MCP access. To do this, you need to add a `meta.mcp.public` flag to the ability registration arguments when you register your ability with `wp_register_ability()`.

```php
'meta' => array(
    'mcp' => array(
        'public' => true,  // Required for MCP default server access
    ),
)
```

In the case of any Core Abilities, you can hook into the `wp_register_ability_args` filter, to update their registration arguments to include the `meta.mcp.public` flag.

```php
<?php
/**
 * Plugin Name: Enable core abilities
 * Version: 1.0.0
 *
 * @package enable-core-abilities
 */

add_filter( 'wp_register_ability_args', 'myplugin_enable_core_abilities_mcp_access', 10, 2 );
/**
 * Enable MCP access for core abilities.
 *
 * @param array  $args        Ability registration arguments.
 * @param string $ability_name  Ability ID.
 * @return array Modified ability registration arguments.
 */
function myplugin_enable_core_abilities_mcp_access( array $args, string $ability_name ) {
	// Enable MCP access for the three current core abilities.
	$core_abilities = array(
		'core/get-site-info',
		'core/get-user-info',
		'core/get-environment-info',
	);
	if ( in_array( $ability_name, $core_abilities, true ) ) {
		$args['meta']['mcp']['public'] = true;
	}

	return $args;
}
```

With this in place, you can start connecting AI clients to your WordPress site via the MCP Adapter and start calling these core abilities via the default server's MCP tools.

## Connecting AI applications

To communicate with an MCP server, there are [two transport mechanisms](https://modelcontextprotocol.io/docs/learn/architecture#transport-layer): **STDIO** and **HTTP**.

For local WordPress environments, the most straightforward way to connect is using **STDIO** transport. The MCP Adapter makes this possible via [WP-CLI](https://wp-cli.org/), so you need to have WP-CLI installed locally. 

For any public environments, or if you don't want to use STDIO, you can set up an **HTTP** connection using the [`@automattic/mcp-wordpress-remote`](https://www.npmjs.com/package/@automattic/mcp-wordpress-remote) remote proxy. This requires you to have [Node.js](https://nodejs.org/en) installed, and to set up authentication using either [WordPress application passwords](https://make.wordpress.org/core/2020/11/05/application-passwords-integration-guide/) or a custom OAuth implementation. 

Regardless of which transport or authentication methods you choose, you'll need to configure your AI application to connect to your MCP server. This is typically done via a JSON configuration file that specifies the command to start the MCP server, any necessary arguments, and environment variables for authentication.

For the STDIO transport, at minimum you need to configure the following to connect to your MCP enabled WordPress site:

```json
    "wordpress-mcp-server": {
      "command": "wp",
      "args": [
        "--path=/path/to/your/wordpress/installation",
        "mcp-adapter",
        "serve",
        "--server=mcp-adapter-default-server",
        "--user={admin_user}"
      ]
    }
```

- the server name in this case is `wordpress-mcp-server`. This can be any name you choose.
- the command is `wp`, which is the WP-CLI command-line tool
- the args array includes:
    - `--path` pointing to your WordPress installation 
    - `mcp-adapter serve` to start the MCP Adapter server
    - `--server` specifying the MCP server to use (in this case, the default server)
    - `--user` specifying the WordPress user to authenticate as (in this case the `admin` user)

If you're using the HTTP transport, your minimum configuration should look like this:

```json
    "wordpress-mcp-server": {
      "command": "npx",
      "args": ["-y", "@automattic/mcp-wordpress-remote@latest"],
      "env": {
        "WP_API_URL": "https://yoursite.example/wp-json/mcp/mcp-adapter-default-server",
        "WP_API_USERNAME": "{admin_user}",
        "WP_API_PASSWORD": "{application-password}"
      }
    }
  }
```

- the server name
- the command is `npx`, which executes Node.js packages
- the args array includes:
    - `-y` to automatically agree to install the package
    - `@automattic/mcp-wordpress-remote@latest` to use the latest version of the remote MCP proxy package
- the env object includes:
    - `WP_API_URL` pointing to the MCP endpoint on your WordPress site
    - `WP_API_USERNAME` specifying the WordPress user to authenticate as
    - `WP_API_PASSWORD` specifying the application password for the user

For local environments connecting via the remote proxy, the `mcp-wordpress-remote` package [includes some troubleshooting tips](https://github.com/Automattic/mcp-wordpress-remote/blob/trunk/Docs/troubleshooting.md) if you run into any issues connecting. Usually these issues are related to having multiple versions of Node.js installed, or issues related to local SSL certificates.

Now let's look at where to configure your MCP server in the most popular AI applications, Claude Desktop, VS Code, Cursor, and Claude Code.

> **Note:** For all of the examples below, I'm using a Studio site named 'WordPress MCP' located at `/Users/jonathanbossenger/Studio/wordpress-mcp` and browsable via http://localhost:8885/, with an admin user named `admin`. Make sure to replace these values with your own WordPress site path and user.

### Claude Desktop

Being an Anthropic product, Claude Desktop was one of the first apps with built-in support for MCP servers. To add MCP servers to Claude Desktop, navigate to the **Developer** tab (_Claude → Settings → Developer_). Under Local MCP servers click **Edit config**. 

[!Claude Desktop MCP Config](images/mcp-01-claude-developer-settings.png);

This will open a file browser to the location of the `claude_desktop_config.json` file, where you can add your MCP server configurations.

MCP Servers are added to this file in a `mcpServers` object.

```json
{
  "mcpServers": {
  }
}
```

Here's what the configuration looks like for connecting via STDIO transport:

```json
{
  "mcpServers": {
    "wordpress-mcp-server": {
      "command": "wp",
      "args": [
        "--path=/Users/jonathanbossenger/Studio/wordpress-mcp",
        "mcp-adapter",
        "serve",
        "--server=mcp-adapter-default-server",
        "--user=admin"
      ]
    }
  }
}
```

Here's what the configuration looks like for connecting via HTTP transport:

```json
{
  "mcpServers": {
    "wordpress-mcp-server": {
      "command": "npx",
      "args": ["-y", "@automattic/mcp-wordpress-remote@latest"],
      "env": {
        "WP_API_URL": "http://localhost:8885/wp-json/mcp/mcp-adapter-default-server",
        "WP_API_USERNAME": "admin",
        "WP_API_PASSWORD": "2SEB qW5j D7CW fpsh pbmN RGva"
      }
    }
  }
}
```

Once you save the configuration file, you will have to restart Claude Desktop, as it only reads the MCP server configurations on startup.

You should now see your MCP server listed in the **Developer** tab under **Local MCP servers**. If you see the `running` status next to your server name, you're ready to start using it in your conversations.

[!Claude Desktop MCP Server](images/mcp-03-claude-mcp-server.png);

### Cursor

In Cursor, navigate to the **Settings** tab (_Cursor → Settings → Cursor Settings_), then select the **Tools and MCP** section. 

[!Cursor](images/mcp-04-cursor.png);

Click on **Add Custom MCP** button, which will open the mcp.json configuration file for Cursor.

The configuration for Cursor is the same as for Claude Desktop. Once you've added your MCP server configuration, save the file and navigate back to the **Tools and MCP** section in Cursor settings. You should see your MCP server listed there, and you can enable it for use in your coding sessions.

[!Cursor MCP](images/mcp-05-cursor-mcp.png);

## Claude Code

To add MCP servers to Claude Code, you can either add the `mcpServers` object with the relevant server configs to the `.claude.json` file on your home directory, or create a `.mcp.json` file in your project directory. Adding the MCP servers to the project directory allows you to have different MCP server configurations for different projects, whereas adding them to the home directory makes them available globally across all projects.

Either way, you can use the same configuration format as Cursor or Claude Desktop.

### VS Code

Configuring VS Code to connect to an MCP server requires setting up a [JSON configuration file that describes the MCP server details](https://code.visualstudio.com/docs/copilot/customization/mcp-servers). This file is usually named `mcp.json` and should be placed in a `.vscode` directory inside your project workspace.

The only difference between configuring VS Code and Claude Desktop is that you define your MCP servers in a `servers` object not an `mcpServers` object. The rest of the configuration is the same.

```json
{
  "servers": {
    // MCP server definitions go here
  }
}
```

Once you create this file in your project workspace, VS Code displays an MCP control toolbar, where you can start, stop and restart the MCP server.

[!VS Code](images/mcp-06-vscode.png);

When the sever has started correctly, it will also show you how many tools are available for the AI to use, in this case three.

## Using MCP tools

With your MCP server connected to your AI application of choice, you can now start using the MCP tools exposed by the MCP Adapter.

For example, in Claude Desktop, you can start a new conversation asking Claude to "Get the site info from my WordPress site".

[!Claude Desktop Query](images/mcp-03-claude-query.png);

It will determine that there is an available MCP server and call the `mcp-adapter-discover-abilities` tool to see what abilities are available. It will then determine that the `core/get-site-info` Ability will fulfill the request, and call the `mcp-adapter-execute-ability` tool, passing it the `core/get-site-info` Ability name. This will return the site info data, and the application will "answer" with the site information.

## Configuring custom MCP Servers for your plugins

While the MCP Adapter default server should cover most requirements, you may want to create a custom MCP server for your plugin or theme. This allows you to have more control over how your abilities are exposed as MCP tools.

Implementing this requires installing the MCP Adapter package via Composer, and creating and registering a custom MCP server.

From your plugin or theme directory, run the `composer require` command:

```bash
composer require wordpress/mcp-adapter
```

Then, make sure to load Composer’s autoloader in your main plugin file or theme's `functions.php`:

```php
if ( file_exists( __DIR__ . '/vendor/autoload.php' ) ) {
	require_once __DIR__ . '/vendor/autoload.php';
}
```

If it's possible that multiple plugins on a site might depend on the MCP Adapter or Abilities API, the official documentation [recommends using the Jetpack Autoloader](https://github.com/WordPress/mcp-adapter/blob/trunk/docs/getting-started/installation.md#using-jetpack-autoloader-highly-recommended) to avoid version conflicts.

The next step is to initialize the MCP Adapter in your plugin or theme:

```php
<?php
if ( ! class_exists( WP\MCP\Core\McpAdapter::class ) ) {
    // check if the MCP Adapter class is available, if not show some sort of error or admin notice
    return;
}

// Initialize MCP Adapter and its default server.
WP\MCP\Core\McpAdapter::instance();
```

Finally, you can create a custom MCP server by hooking into the `mcp_adapter_init` action. The action callback function receives the `McpAdapter` instance. The adapter's `create_server()` method is used to define the custom server, with the desired configuration.

```php
add_action( 'mcp_adapter_init', 'myplugin_create_custom_mcp_server' );
function myplugin_create_custom_mcp_server( $adapter ) {
    $adapter = WP\MCP\Core\McpAdapter::instance();
    $adapter->create_server(
        'custom-mcp-server', // Unique server identifier.
        'custom-mcp-server', // REST API namespace.
        'mcp',               // REST API route.
        'Custom MCP Server', // Server name.
        'Custom MCP Server', // Server description.
        'v1.0.0',            // Server version.
        array(               // Transport methods.
            \WP\MCP\Transport\HttpTransport::class,  // Recommended: MCP 2025-06-18 compliant.
        ),
        \WP\MCP\Infrastructure\ErrorHandling\ErrorLogMcpErrorHandler::class, // Error handler.
        \WP\MCP\Infrastructure\Observability\NullMcpObservabilityHandler::class, // Observability handler.
        array( 'namespace/ability-name' ), // Abilities to expose as tools
        array(),                           // Resources (optional).
        array(),                           // Prompts (optional).
    );
}
```

The important parameters to note here are:
- The first parameter is the unique server identifier. This is used when starting the MCP server via WP-CLI.
- The second and third parameters define the REST API namespace and route for the MCP server.
- The fourth and fifth parameters are the server name and description, which are displayed in AI applications when listing available MCP servers.
- The sixth parameter is the server version.
- The tenth parameter is an array of ability names that you want to expose as MCP tools. You can list multiple abilities here.
- The rest of the parameters define the transport methods, error handling, and observability handlers. In this example, you're using the HTTP transport, error logging, and a null observability handlers from the core MCP Adapter package. However, it is possible to create your own custom handlers if you want to integrate with your own transport, logging or monitoring systems.

## Adding an MCP server to List All URLs

To show you an example of creating a custom MCP server, let's take the [List All URLs plugin](https://github.com/wptrainingteam/list-all-urls) from the Abilities API post, and add a custom MCP server to it. 

Before you do this, deactivate the MCP Adapter plugin if you have it activated.

Next, clone the List All URLs GitHub repository inside your WordPress plugins directory:

```bash
cd wp-content/plugins
git clone git@github.com:wptrainingteam/list-all-urls.git
```

You'll also need to switch to the branch that includes the Abilities API implementation:

```bash
cd list-all-urls
git checkout abilities
```

The plugin already uses Composer for dependency management, so run `composer install` to install the required packages.

```bash
composer install
```

Next require the mcp-adapter package:

```bash
composer require wordpress/mcp-adapter
```

Now, open the main plugin file `list-all-urls.php`, and add the following code at the bottom of the file to initialize the MCP Adapter and create a custom MCP server:

```php
<?php
if ( ! class_exists( WP\MCP\Core\McpAdapter::class ) ) {
	return;
}

// Initialize MCP Adapter and its default server.
WP\MCP\Core\McpAdapter::instance();

add_action( 'mcp_adapter_init', 'list_all_urls_create_custom_mcp_server' );
/**
 * Create a custom MCP server for the List All URLs plugin.
 *
 * @param object $adapter WP\MCP\Core\McpAdapter The MCP Adapter instance.
 * @return void
 */
function list_all_urls_create_custom_mcp_server( $adapter ) {
	$adapter = WP\MCP\Core\McpAdapter::instance();
	$adapter->create_server(
		'list-all-urls-mcp-server',
		'list-all-urls-mcp-server',
		'mcp',
		'List All URLS MCP Server',
		'Custom MCP Server for the List All URLs plugin. Currently exposes only the list-all-urls/urls ability as an MCP Tool.',
		'v1.0.0',
		array(
			\WP\MCP\Transport\HttpTransport::class,
		),
		\WP\MCP\Infrastructure\ErrorHandling\ErrorLogMcpErrorHandler::class,
		\WP\MCP\Infrastructure\Observability\NullMcpObservabilityHandler::class,
		array( 'list-all-urls/urls' ),
	);
}
```

Notice that you don't need to enable the `meta.mcp.public` flag for the `list-all-urls/urls` ability, because you're explicitly exposing it via the custom MCP server.

Now activate the List All URLs plugin from the WordPress admin dashboard. 

Once the plugin is activated, update your AI application's MCP server configuration to use the new custom MCP server. 

Here's an example of the updated VS Code configuration to include both the default MCP server and the new custom MCP server from the List All URLs plugin, both using STDIO transport:

```json
{
  "servers": {
    "wordpress-mcp-server": {
      "command": "wp",
      "args": [
        "--path=/Users/jonathanbossenger/Studio/wordpress-mcp",
        "mcp-adapter",
        "serve",
        "--server=mcp-adapter-default-server",
        "--user=admin"
      ]
    },
    "list-all-urls-mcp-server": {
      "command": "wp",
      "args": [
        "--path=/Users/jonathanbossenger/Studio/wordpress-mcp",
        "mcp-adapter",
        "serve",
        "--server=list-all-urls-mcp-server",
        "--user=admin"
      ]
    }
  }
}
```

It's possible to have multiple MCP servers configured in the same AI application. This allows you to switch between different WordPress sites or plugins that expose different sets of abilities.

Whatever AI application you're using, make sure to either restart the application, or start the MCP server in the application after updating the MCP server configuration. You should see the new MCP server listed and be able to use the `list-all-urls/urls` ability as an MCP tool. You can then ask the AI to "List all URLs on my WordPress site", and it will call the `list-all-urls-urls` tool via the MCP Adapter.

## Security and best practices

Because MCP clients act as **logged-in WordPress users**, always treat them as part of your application surface area by following these best practices:

- **Use `permission_callback` carefully**
    - Each ability should check the minimum capability needed (`manage_options`, `edit_posts`, etc.).
    - Avoid `__return_true` for destructive operations such as deleting content.
- **Use dedicated users for MCP access**
    - Especially in production, create a specific role/user with limited capabilities.
    - Do not expose powerful abilities to unaudited AI clients.
- **Prefer read-only abilities for public MCP endpoints**
    - For HTTP transports exposed over the internet, focus on read-only diagnostics, reporting, and content access.
- **Implement custom authentication if needed**
    - The default authentication uses application passwords, but you can implement OAuth or other methods for better security.
- **Monitor and log usage**
    - Use custom error and observability handlers to integrate with your logging/monitoring stack.

## How to start experimenting today

If you want to get started experimenting with the MCP adapter, a minimal “hello AI” path for a WordPress developer only requires you to register an ability, require and initialize the MCP Adapter, and connect an MCP-aware AI client.

If you already have plugins using the Abilities API, the MCP Adapter turns them into **AI-ready APIs** with very little additional work.

As with any new technology, start small. Begin by exposing a few non-destructive, read-only abilities as MCP tools. Test them with local AI clients like Claude Desktop or Cursor. Gradually expand to more complex abilities and workflows as you gain confidence. Be prepared to hit roadblocks, and engage with the WordPress AI and developer communities for support and collaboration. The documentation for both the [Abilities API](https://developer.wordpress.org/apis/abilities/) and the [MCP Adapter](https://github.com/WordPress/mcp-adapter) are great resources to help you along the way.

This combination of Abilities and the MCP Adapter gives WordPress developers a powerful path to build AI-assisted admin tools, automation, and workflows for both clients and teams. 