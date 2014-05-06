---
layout: default
title: Basics |
---

# The Basics

Exam proportion: 6%.

## Fundamentals

Magento uses Object Oriented Programming (OOP) as its programming paradigm which consists of *classes*, *objects* and *methods*.  It also uses the Model-View-Controller (MVC) design pattern.

Magento makes use of an Event Driven Architecture (EDA).  This is were components of the system or either producers or consumers of events.  An EDA allows for loosely coupled, distributed software modules and is, by its nature, a reactionary system that performs actions based on stimuli.


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
<?php `$currentCategory = Mage::registry('current_category');` ?>
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
<?php Mage:getModel('catalog/product')->getTypeInstance(); ?>
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

Magento uses public `__construct` methohods and, typically, protected `_construct` methods for class initialisation. More information on the difference between the two can be found on [stackoverflow](http://stackoverflow.com/questions/8706352/why-does-magento-have-construct-and-construct-methods). 

Arguments can be passed to modesl created through factories by using a second argument, e.g.

```php
<?php
    $instance = Mage::getModel('prefix/model', array(
        `argument_one` => $value_one,
        `argument_one` => $value_two
    ));
?>
```

The argument parameter to `getModel` will be passed as is.  There is no way to specify more than one argument for the constructor.  To get around this restriction, we use an array.

## High-Level Architecture

### Code Pools

Magento is split up into three code pools. 

- Core
	- Contains core modules developed by Magento.
- Community
	- The location of third party modules (extension) that are packaged and released.
- Local
	- Installation specific modules, customisations and overrides.


### Typical Module Structure

- Module Declaration

`app/etc/modules/{namespace}_{module name}.xml`

Defines the module, the codepool it is in, any dependencies and whether it is enabled.

- Module Code

`app/code/{codepool}/{namespace}/{module name}`

Contains the module configuration and code.  This includes:

- Blocks 
	- `Block/`
- Controllers
	- `controllers/`
- Models
	- `Model`
- Resource Models
	- `Model/Resource/`
- Helpers
	- `Helpers`
- Database Installation and Upgrade Scripts
	- `sql/`
- Data scripts 
	- `data/`


### Magento Templates and Layout Files

`app/design/{area}/{package}/{theme}/`

Layout and template files are organised into folders by store area, package (theme namespace) and theme. Inside the layout files and template files are separated into `layout/` and `template/` directories respectively.


### Magento Skin and JavaScript files location

`skin/{area}/{package}/{theme}`

Same as with template files, most Magento assets (js, css and images) are organised by store area, package and theme. Inside, the convention is to put different types of assets into their own folders, but it is not enforced.

`js/`

Contains theme independent JavaScript files and other assets (e.g. css, images), organised by library.

### Magento Design Areas

- Adminhtml
	- Contains the files dealing with the presentation of the Magento backend.
- Frontend
	- Contains the layouts, templates and assets for the customer facing areas of Magento.

### Class Naming Conventions and Autoloader

Magento include path is set to look for files in all of the codepools (`core`, `community` and `local` inside `app/code`) and libraries (`lib/` directory). Classes are resoled using the [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) autoloading standard.  That is, the autoloader simply replaces the underscores in the class name with directory separators and adds the .php extension:

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


Therefore the class naming convention for most Magento classes is:

`{Namespace}_{Module name}_{Object type}_{Object name}`

### Module Conflict Resolution

Magento modules can conflict in three different ways:

- Configuration conflicts
	- Two modules could be extending the same class or making configuration changes in the same areas. You can force one to take precedence over the other with a `<depends>` dependency tag in the module declaration, forcing on module to always be loaded before the other.

- Rewrite conflicts
	- Magento models and blocks can only be rewritten once in `config.xml`. If a module is rewriting a model that has already been rewritten, the first rewrite will be eliminated. This can be resolved by extending the first rewrite within the second one to make sure the changes are still being applied.

- Theme conflicts
	- Modules can make changes to the layout, moving, removing and replacing blocks or changing their templates.  Investigate the templates and module configuration to find the cause of the conflict.


## Magento Configuration

### Config Load

Configuration in Magento is stored as an XML tree.  Magento reads in the config.xml files for each module and merges the configuration into a single tree.  This allows modules to modify existing configuration elements and add new ones.

### Class Group Config and Factory Methods

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

These objects are then instantiated in the code using the factory methods (`Mage::getModel()` or `Mage::helper()`) and are referred to using their grouped class name - `{module identifier}/{object identifier}`.  Internally, Magento uses the module identifier (the part before the slash) to find the class name prefix to prepent to the object name (the part after the slash) to find the class that needs to be instantiated.  This allows modules to redefine existing objects and functionality by changing the configuration.

### Class Overrides

Object classes can be overriden iun configuration using the following syntax:

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

This indicates to the factory methods that the specified class should be used instead of the usual class the object should resolve to.

### Register an Observer

Observers are registerd in Magento using the following syntax:

```xml
<config>
    <{area}>
        <events>
            <{event name}>
                <observers>
                    <{observer name}>
                        <type></type>
                        <class></class>
                        <method></method>
                    </{observer name}>
                </observers>
            </{event name}>
        </events>
    </{area}>
</config>
```

This requires specifying the area of the event listented to (`frontend`, `adminhtml` or `global`), the event being listened to, the observer identifier (unique name), the observer object type (e.g. singleton model), the class used and the method to call.

### Function and use of automatically available events

e.g. `*_load_after`.


### Cron Jobs

Crons jobs are defined in Magento using the following syntax:

```xml
<config>
    <crontab>
        <jobs>
            <{cronjob identifier}>
                <schedule></schedule>
                <run>
                    <model></model>
                </run>
            </{cronjob identifier}>
        </jobs>
    </crontab>
</config>
```

The cron job schedule can eitehr be a cron schedule expression (inside a `<cron_expr>` tag) or a configuration path (inside a `config_path` tag) where the crons schedule expression is stored.  The `<model>` tag defines the function to execute in the format `{grouped class name}::{method name}`.

### Loading active modules

Magento loads all `*.xml` files from `app/etc/modules` to find a list of installed modules.  It identifies whether they are active and where their full configuration is located (`<codepool>`).

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

Prefixes are used for blocks, models and helpers.  Using prefixes and not referring to classes directly by naem allows rewriting the classes in the configuration and without the need to modify the code that uses them.

## Internationalisation

`$this->__('Translate Me')` method is used to manage translatable text.  Before outputing this string it is checked against a number of locations for transactions.  The priority for translations are in this order:

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





<ul class="navigation">
    <li class="prev"><a href="/">&larr; Home</a>
    <li class="next"><a href="/request-flow.html">Request Flow &rarr;</a>
</ul>
