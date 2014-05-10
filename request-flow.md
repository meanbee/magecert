---
layout: default
title: Request Flow
chapter: 2
meta-description: Learn how Magento bootstraps itself and handles requests. 
---

# Request Flow

Exam proportion: 7%.

## Application Initialization

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

The include path is set up and the autoloader is registered when the `Mage.php` file is included in `index.php`.  Shortly after this occurs the autoloader is registered with `spel_autoloader_register`.

### Loading Magento Module and Database Configuration

The base configuration of Magento is loaded in `Mage_Core_Model_Config::loadBase`. It grabs the glob of `app/etc/*.xml` which contains key config information, e.g. database credentials, the core module, the results of installation.

Database config is stored in `app/etc/local.xml` and `app/etc/config.xml`.

`Mage_Core_Model_Config::loadModules` handles looping over each of the modules that exist in `app/etc/modules/` and merging their own `config.xml` files from their respective module directories.

The order that modules are loaded in is:

1. `Mage_All.xml`
2. `Mage_*.xml`
3. Everything else 

If a module depends on another, Magento makes sure it exists and loads its configuration first. 

Modules are loaded after the base configuration but before store initialisation.


### Setup Script Execution

Module loading and SQL database upgrades are run when `Mage_Core_Model_App::app()` is called but data upgrades are not.  Whereas `Mage_Core_Model_App::run()` completes all of the above.  It does so in two phases.

1. `Mage_Core_Model_Resource_Setup::applyAllUpdates()` - executed immediately after the module configuration is loaded and runs all the SQL install and update scripts.

2. `Mage_Core_Model_Resource_Setup::applyAllDataUpdates()` - executed after the store, locale and request models have been initialized and runs the data install and upgrade scripts.

### Magento Stores and Locale Loading

In the `Mage_Core_Model_App::run()` method, Magento sets up which store to use by:

```php
<?php $this->_initCurrentStore($scopeCode, $scopeType); ?>
```

There are multiple ways to specify the current store.

- Environment variables
- Reorder store priorities
- GET parameter `__store`

The environment variables are checked in `index.php`:

```php
<?php 
	/* Store or website code */
	$mageRunCode = isset($_SERVER['MAGE_RUN_CODE']) ? $_SERVER['MAGE_RUN_CODE'] : '';

	/* Run store or run website */
	$mageRunType = isset($_SERVER['MAGE_RUN_TYPE']) ? $_SERVER['MAGE_RUN_TYPE'] : 'store';
?>
```

### Request and Response Objects

The request object is initialised in `Mage_Core_Model_App` and the `_initRequest()` method.  `getRequest()` and `getResponse` are relevant.


## Front Controller

### Role

The front controller performs the routing of the request to the appropriate controller.  It loops over all of the registered routers, passing the request to each one of them to be matches against a controller capable of handling it.  After the request has been dispatched, the front controller sends the response to the client.

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

```xml
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
```

- By observer the `controller_front_init_before` or `controller_front_init_routers` events and injecting the router into the front controller.

## URL Rewrites

### URL Structure

The URL structure in Magento generally uses the format `{base_url}/{front_name}/{controller}/{action}`.

`Mage_Core_Controller_Varien_Router_Standard` parses the URLs in this format and maps them to a module used and the controller action to be executed. 

### URL Rewrite Process

URL rewrites happen in the Front controller before the routing.  The database rewrites are checked and applied first, followed by the configuration (`glboal->rewrite`) rewrites. Rewrites can either redirect the request using HTTP methods, update the request path (keeping the old one for reference) or completely replace the request path.

### Database URL Rewrites

The most important fields in the `core_url_rewrite` table are `request_path` and `target_path` which map the request to a rewrite.

Magento creates catalog requires using the `catalog_url` indexer.

### Matching requests

The requests are applied by the `Mage_Core_Model_Url_Rewrite_Request` model. The request path is parsed to include any variation (with or without the trailing slash) and then looks up the `request_path` column of the `core_url_rewrite` table using the `Mage_Core_Model_Url_Rewrite::loadByRequestPath()` method.

## Request Routing

Built-in Magento routers (in the order that they are matched):

1. Admin
	- Collects all routes for the administration area
	- Looks within `config.xml` for routes within admin.
2. Standard
	- Superclass of Admin
	- Collects routes from within frontend routes defined in `config.xml`
3. CMS
	- Routes CMS pages from identifiers
4. Default
	- This will always match.
	- Routes error or 404 pages.

The standard router maps a path to an action by splitting it into `base_url/frontname/controller/action`. The `frontname` is then mapped to a module by way of the configuration files. The controller file is then located at `/path/to/module/base/controllers/{Name}Controller.php

Unmapped requests read the Default router where they are rewritten to a 404 page and get mapped by the Standard router on the next iteration.

Before dispatch, request module, controller, action and parameters are set.  Then it is passed to the controller (all within the Standard router).


## Design and layout initialization

The store design (`core/design_package`) is initialised in the controller `preDispatch()` method.  The package and theme configuration is then determined by the `Mage_Core_Model_Design`.

Layout files get read (`$layout->gteUpdate()->load()`) when the controller calls `$this->loadLayout()`.  The same method also compiles the layout (`$layout->generateXml()`) which processes the layout directives.

Output is rendered when the controller calls `$this->renderLayout()`, which calls each of the blocks defined to output data e.g. `output="toHtml"` in the definition, and merged their output into the response body.

To add a layout handle to be processed call

```php
<?php Mage::getLayout()->getUpdate()->addHandle('new_handle'); ?>
```

Behind the scenes `Mage_Core_Model_Layout_Update` loads the layout files and their XML while `Mage_Core_Model_Layout` processes it.

Layout XML gets merged, first from modules, then `local.xml` and then the database.  The next step is to remove blocks or references as directed by the `<remove>` element. 

To add a layout file to be merged, add this to an extensions `config.xml` file:

```xml
<config>
	<{area}>
		<layout>
			<updates>
				<{name}>
					<file>{filename.xml}</file>
				</{name}>
			</updates>
		</layout>
	</{area}>
</config>
```

This file will then be searched for in `app/design/{area}/{package}/{theme}/layout/{filename.xml}`

## Flushing Data (output)

Response content gets set by the `$layout->renderLayout()` method.  After the controller dispatch method returns, the Front Controller send the response.

The `controller_front_send_response_before` event can be used to modify the response before sending.  Subsequently, observing for the `controller_front_send_response_after` event allows for cleaning up after if necessary. 

If the output is not sent in a response object but printed out, it can prevent headers from being sent as it is unbuffered.

### Redirects

There are two types of redirects that can be used in a controller action.

- `_redirect()`
	- This performs a HTTP redirect
- `_forward()`
	- This redirects internally within the application to another controller and/or action.

<ul class="navigation">
    <li class="prev"><a href="/basics.html">&larr; Basics</a>
    <li class="next"><a href="/rendering.html">Rendering &rarr;</a>
</ul>


