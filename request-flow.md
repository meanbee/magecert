---
layout: default
---

# Request Flow

## Application Initilization

1. Check compilier configuration
2. Include `Mage.php`
	1. Setup core functions and autoloader
	2. Register the autoloader
3. `Mage::run()`
	1. Instantiate `Mage_Core_Model_App`
	2. Instantiate the Config model
	3. `$app-run()`
		1. `baseInit()` - load the base config and initialise cache.
		2. `initModules()` - load module configuration.
		3. Run all the required SQL install and upgrade scripts
		4. Setup the locale
		5. `initCurrentStore()` - load store configuration and instantiate the store model
		6. `initRequest()` - load the request information into the model
		7. Run all the required data install and upgrade scripts
		8. Dispatch the request

### Include Path & Autoloader Registration

The include path is set up and the autoloader is registered when the `Mage.php` file is included in `index.php`.  Shortly after this occus the autoloader is regsitered with `spel_autoloader_register`.

### Loading Magento Module and Database Configuration

The base configuration of Magento is loaded in `Mage_Core_Model_Config::loadBase`. It grabs the glob of `app/etc/*.xml` which contains key config information, e.g. database credentials, the core module, the results of installation.

`Mage_Core_Model_Config::loadModules` handles looping ver each of the modules (as defined in `app/etc/modules/`) and merging their own `config.xml` files from their respective module directories.

Database config is stored in `app/etc/local.xml` and `app/etc/config.xml`.

### Setup Script Execution

The setup scripts are executed within the `Mage_Core_Model_App::run()` method in two phases.

1. `Mage_Core_Model_Resource_Setup::applyAllUpdates()` - executed immediately after the module configuration is loadewd and runs all the SQL install and update scripts.

2. `Mage_Core_Model_Resource_Setup::applyAllDataUpdates()` - executed after the store, locale and request models have been initilized and runs the data install and upgrade scripts.

### Magento Stores and Locale Loading

In the `Mage_Core_Model_App::run()` method, Magento sets up which store to use by:

{% highlight php %}
	$this->_initCurrentStore($scopeCode, $scopeType);
{% endhighlight %}

There are multiple ways to specify the current store.

- Environment variables
- Reorder store priorities
- GET parameter `__store`

The environment variables are checked in `index.php`:

{% highlight php %}
	/* Store or website code */
	$mageRunCode = isset($_SERVER['MAGE_RUN_CODE']) ? $_SERVER['MAGE_RUN_CODE'] : '';

	/* Run store or run website */
	$mageRunType = isset($_SERVER['MAGE_RUN_TYPE']) ? $_SERVER['MAGE_RUN_TYPE'] : 'store';
{% endhighlight %}

### Request and Response Objects

The request object is initialised in `Mage_Core_Model_App` and the `_initRequest()` method.  `getRequest()` and `getResponse` are relevant.


## Front Controller

### Role

The front controller performs the routing of the request to the appropriate controller.  It loops over all of the registered routers, pasing the request to each one of them to be matches against a controller capable of handling it.  After the request has been dispatched, the front controller sends the response to the client.

### Events

- `controller_front_init_before`
	- Fired before adding the routers.  It is useful for adding a router that takes precedence over any others.
- `controller_front_init_routers`
	- Fired after adding the routers, but before the default router is added.  It is useful for adding general routers or modifying existing ones.
- `controller_front_send_response`
	- Fired before the response is sent out.  It is useful for modifying the response data after the dispatch.
- `controller_front_send_response_after`
	- Fired after the response is sent out.  It is useful for performing any tear down operations after the request has been dealt with.

### Adding Router Classes

There are two ways to add routes. 

- Using configuration

{% highlight xml %}
<config>
    <default>
        <web>
            <routers>
                <{name}>
                    <area></area>
                    <class></class>
                </{name}>
            </routers>
        </web>
    </default>
</config>
{% endhighlight %}

- By observer the `controller_front_init_before` or `controller_front_init_routers` events and injecting the router into the front controller.

## URL Rewrites

### URL Structure

The URL structure in Magento generally uses the format `{base_url}/{front_name}/{controller}/{action}`.

`Mage_Core_Controller_Varien_Router_Standard` parses the URLs in this format and maps them to a module used and the controller action to be executed. 

### URL Rewrite Process

URL rewrites happen in the Front controllerm before the routing.  The database reqrites are checked and applied first, followed by the configuration (`glboal->rewrite`) rewrites. Rewrites can either redirect the request using HTTP methods, updte the request path (keeping the old on efor reference) or completely replace the request path.

### Database URL Rewrites

The most important fields in the `core_url_rewrite` table are `request_path` and `target_path` which map the request to a rewrite.

Magento creates catalog requres using the `catalog_url` indexer.

### Matching requests

The requres are applied by the `Mage_Core_Model_Url_Rewrite_Request` model. The request path is parsed to include any variation (with or without the trailing slash) and then looks up the `request_path` column of the `core_url_rewrite` table using the `Mage_Core_Model_Url_Rewrite::loadByRequestPath()` method.

## Request Routing



