---
layout: default
title: Basics
chapter: 1
meta-description: Introduction to Magento code hierarchies, modules and configuration.
---

# The Basics

Exam proportion: 6%.

## Fundamentals

Magento uses Object Oriented Programming (OOP) as its programming paradigm which consists of *classes*, *objects* and *methods*.  It also uses the Model-View-Controller (MVC) design pattern.

Magento makes use of an Event Driven Architecture (EDA).  This is where components of the system are either producers or consumers of events.  An EDA allows for loosely coupled, distributed software modules and is, by its nature, a reactionary system that performs actions based on stimuli.


### Design Patterns

There are numerous development designs patterns used in Magento ([Source](http://stackoverflow.com/questions/5041473/magento-design-patterns)).

- Factory

```php
<?php $product = Mage::getModel('catalog/product'); ?>
```

- Singleton

```php
<?php $category = Mage::getSingleton('catalog/session'); ?>
```

- Registry

```php
<?php $currentCategory = Mage::registry('current_category'); ?>
```

- Event/Observer

```php
<?php Mage::dispatchEvent('event_name', array('key'=>$value)); ?>
```

```xml
<config>
    <global>
        <events>
            <event_name>
                <observers>
                    <unique_name>
                        <class>Class_Name</class>
                        <method>methodName</method>
                    </unique_name>
                </observers>
            </event_name>
        </events>
    </global>
</config>
```

- Prototype

```php
<?php Mage::getModel('catalog/product')->getTypeInstance(); ?>
```

- Object Pool

```php
<?php
$id = Mage::objects()->save($object);
$object = Mage::objects($id);
?>
```

- Iterator

```php
<?php Mage::getModel('catalog/product')->getCollection(); ?>
```

- View Helper

```php
<?php Mage::helper('core'); ?>
```

### Constructors

Magento uses public `__construct` methods and, typically, protected `_construct` methods for class initialisation. More information on the difference between the two can be found on [stackoverflow](http://stackoverflow.com/questions/8706352/why-does-magento-have-construct-and-construct-methods).

Arguments can be passed to models created through factories by using a second argument, e.g.

```php
<?php
    $instance = Mage::getModel('prefix/model', array(
        `argument_one` => $value_one,
        `argument_two` => $value_two
    ));
?>
```

The argument parameter to `getModel` will be passed as is.  There is no way to specify more than one argument for the constructor.  To get around this restriction, we use an array.

## High-Level Architecture

<h3 id="code-pools">Code Pools</h3>

Magento is split up into three code pools.

- Core
	- Contains core modules developed by Magento.
- Community
	- The location of third party modules (extensions) that are packaged and released.
- Local
	- Installation specific modules, customisations and overrides.


<h3 id="module-structure">Typical Module Structure</h3>

- Module Declaration

`app/etc/modules/{namespace}_{module name}.xml`

Defines the module, the code pool it is in, any dependencies and whether it is enabled.

- Module Code

`app/code/{codepool}/{namespace}/{module name}`

Contains the module configuration and code.  This includes:

- Blocks
	- `Block/`
- Controllers
	- `controllers/`
- Configuration files
	- `etc/`
- Models
	- `Model/`
- Resource Models
	- `Model/Resource/`
- Helpers
	- `Helper/`
- Database Installation and Upgrade Scripts
	- `sql/`
- Data scripts
	- `data/`


<h3 id="template-layout">Magento Templates and Layout Files</h3>

`app/design/{area}/{package}/{theme}/`

Layout and template files are organised into folders by store area, package (theme namespace) and theme. Inside, the layout files and template files are separated into `layout/` and `template/` directories respectively.


<h3 id="skin-javascript">Magento Skin and JavaScript files location</h3>

`skin/{area}/{package}/{theme}`

As with template files, most Magento assets (js, css and images) are organised by store area, package and theme. Inside, the convention is to put different types of assets into their own folders, but it is not enforced.

`js/`

Contains theme independent JavaScript files and other assets (e.g. css, images), organised by library.

<h3 id="design-areas">Magento Design Areas</h3>

- Adminhtml
	- Contains the files dealing with the presentation of the Magento backend.
- Frontend
	- Contains the layouts, templates and assets for the customer facing areas of Magento.

<h3 id="naming-conventions">Class Naming Conventions and Autoloader</h3>

Magento's include path is set to look for files (`app/code`) and libraries (`lib/`) in each of the code pools (`core`, `community` and `local`). Classes are resolved using the [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) autoloading standard.  That is, the autoloader simply replaces the underscores in the class name with directory separators and adds the .php extension:

```php
<?php
public function autoload($class)
{
    $classFile = str_replace(' ', DIRECTORY_SEPARATOR, ucwords(str_replace('_', ' ', $class)));
    $classFile.= '.php';
    @include $classFile;
}
?>
```


The class naming convention for most Magento classes is:

`{Namespace}_{Module name}_{Object type}_{Object name}`

<h3 id="conflict-resolution">Module Conflict Resolution</h3>

Conflicts can arise in three ways:

- Configuration conflicts
	- Two modules could be extending the same class or making configuration changes in the same areas. You can specify precedence explicitly with a `<depends>` dependency tag in the module declaration; forcing one module to always be loaded before the other.

- Rewrite conflicts
	- Magento models and blocks can only be rewritten once in `config.xml`. But we can sequence rewrites (ensuring that both changes are applied) by extending the first rewrite within the second.

- Theme conflicts
	- Changing the layout (moving, removing and replacing blocks, or changing their templates) can introduce conflicts if other modules are expecting a particular structure.  Investigate the templates and module configuration to locate the causes and resolve them.


## Magento Configuration

<h3 id="config-load">Config Load</h3>

Configuration in Magento is stored as an XML tree.  Magento reads in the config.xml files for each module and merges the configuration into a single tree.  This allows modules to modify existing configuration elements and add new ones.

<h3 id="factory-methods">Class Group Config and Factory Methods</h3>

In Magento models, blocks and helpers are registered in configuration using the following format:

```xml
<config>
	<global>
		<{object type}>
			<{module identifier}>
				<class>{class prefix}</class>
			</{module identifier}>
        </{object type}>
    </global>
</config>
```

These objects are then instantiated in the code using the factory methods (`Mage::getModel()` or `Mage::helper()`) and are referred to using their grouped class name - `{module identifier}/{object identifier}`.  Internally, Magento uses the module identifier (the part before the slash) to find the class name prefix to prepend to the object name (the part after the slash) to find the class that needs to be instantiated.  This allows modules to redefine existing objects and functionality by changing the configuration.

<h3 id="overrides">Class Overrides</h3>

Object classes can be overridden in configuration using the following syntax:

```xml
<config>
    <global>
        <{object type}>
            <{module identifier}>
                <rewrite>
                    <{object name}>{new class}</{object name}>
                </rewrite>
            </{module identifier}>
        </{object type}>
    </global>
</config>
```

This indicates to the factory methods that the specified class should be used instead of the usual class the object would resolve to.

<h3 id="observer">Register an Observer</h3>

Observers are registered in Magento using the following syntax:

```xml
<config>
    <{area}>
        <events>
            <{event name}>
                <observers>
                    <{observer name}>
                        <type>{type}</type>
                        <class>{class}</class>
                        <method>{method}</method>
                    </{observer name}>
                </observers>
            </{event name}>
        </events>
    </{area}>
</config>
```

This requires specifying the `area` of the event listened to (`frontend`, `adminhtml` or `global`), the `event` being listened to, the `observer identifier` (unique name), the observer object `type` (e.g. singleton model), the `class` used and the `method` to call.

<h3 id="automatic-events">Function and use of automatically available events</h3>

Magento dispatches a number of events automatically that can be observed, e.g. `*_save_before` and `*_load_after`.  

This provides us with the ability to modify models at the point before they are saved to the database or to modify a model immediately after its data has been retrieved from the database.

<h3 id="cron-jobs">Cron Jobs</h3>

Crons jobs are defined in Magento using the following syntax:

```xml
<config>
    <crontab>
        <jobs>
            <{cronjob identifier}>
                <schedule>{schedule}</schedule>
                <run>
                    <model>{grouped class name}::{method name}</model>
                </run>
            </{cronjob identifier}>
        </jobs>
    </crontab>
</config>
```

The cron job `schedule` can either be a cron schedule expression (inside a `<cron_expr>` tag) or a configuration path (inside a `config_path` tag) where the crons schedule expression is stored.  The `<model>` tag defines the function to execute in the format `{grouped class name}::{method name}`.

### Loading active modules

Magento creates a central list of installed modules by loading all `*.xml` files within `app/etc/modules`.  It records whether they are active and the location of their full configuration (`<codepool>`).

```xml
<?xml version="1.0" ?>
<!-- File: app/etc/modules/Meanbee_Royalmail.xml -->
<config>
    <modules>
        <Meanbee_Royalmail>
            <active>1</active>
            <codePool>community</codePool>
            <depends>
                <Mage_Shipping />
            </depends>
        </Meanbee_Royalmail>
    </modules>
</config>
```

### Common config load methods

- `Mage::app()->getConfig()->getNode();`
- `Mage::getStoreConfig();`
- `Mage::getStoreConfigFlag();`

### Per-Store Configuration in XML DOM

Per-store configuration values are located in the `stores->{store name}` part of the configuration and are usually retrieved with the `Mage::getStoreConfig()` method.

### Configured Class prefixes

Prefixes are used for blocks, models and helpers.  Using prefixes and not referring to classes directly by name allows rewriting the classes in the configuration and without the need to modify the code that uses them.

<h2 id="internationalization">Internationalisation</h2>

Store views are typically used for internationalisation in Magento, with each one being configured for a particular language.

In template files `$this->__('Translate Me')` method is used to manage translatable text.  This is then handled by `Mage_Core_Helper_Data` and `Mage_Core_Model_Translate`.

Before outputting this string it is checked against a number of locations.  The priority for translations are in this order:

1. Inline
    - The `core_translate` database table
2. Theme
    - `app/design/$area/$package/$theme/locale/`
3. Module
    - `app/locale/`

If developer mode is enabled, Magento doesn't use translations unrelated to module.

Module scope translation:

```
"Namespace_Module::string","translation"
```

<h3 id="subdomains-directories">Sub-domains and Sub-directory Setup</h3>

To make use of the internationalisation, a store view with appropriate configurations should be initialised.  Two ways that this can be handled is by sub-domain e.g. `fr.example.com` and sub-directory, e.g. `example.com/fr`.

There are advantages and disadvantages of each. But in general, sub-domains and sub-directories achieve the same effect.

Sub-domains are slightly easier for linking content. Another advantage of sub-domains is that they can be hosted on separate servers. This can be ideal for geo-locating boxes by country or region according to the language served per store view.

With sub-directories, we strengthen name-recognition for the primary domain. And enjoy the benefit that all store views are contributing to SEO efforts by raising total hits for the store brand. But we sacrifice the opportunity for cleaner URLs with localized domain branding. (e.g., http://amazon.com and http://amazon.de vs. http://amazon.com and http://amazon.com/germany)

See even more info in [Belvg's post](http://blog.belvg.com/magento-certified-developer-exam-internationalization.html) on the subject.

<ul class="navigation">
    <li class="prev"><a href="/">&larr; Home</a></li>
    <li class="next"><a href="/request-flow.html">Request Flow &rarr;</a></li>
</ul>
