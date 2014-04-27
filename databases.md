---
layout: default
---

# Databases

## Models, Resource Models and Collections

### Basic Concepts

A **Model** is used to store and manipulate data about a particular object.

A **Resource Model** is used to interact with the database on behalf of the *Model*.  The Resouce Model actually performs the CRUD operations.

A **Collection Model** handles working with groups of models and performing (CRUD) operations on groups of models.  

There's a basic ActiceRecord-like/one-object-one-table model, as well as an Entity Attribute Value (EAV) model.

### Database Connection

Databse connections are configured in `config.xml`:

{% highlight xml %}
<config>
    <global>
        <resources>
            <{identifier}>
                <connection>
                    <host>{host}</host>
                    <username>{username}</username>
                    <password>{password}</password>
                    <dbname>{database}</dbname>
                    <model>{model}</model>
                    <initStatements>{init commands}</initStatements>
                    <type>{adapter type}</type>
                    <active>{0|1}</active>
                </connection>
            </{identifier}>
        </resources>
    </global>
</config>
{% endhighlight %}

The core Magento connections (`default_setup`, `default_read`, `default_write`, `core_setup`, `core_read`, `core_write`) are configured in `app/etc/config.xml` with the database credentials stored in `app/etc/local.xml`.

### Working with Database Tables

Magento used the *Resource Models* to interact with the database tables.  When a *Model* is loaded or saved, it cals its resource model to perform thee operation (executing the database queries).  Database table names are configured in `config.xml` and resource models retrieve them using look-up mehtods, which allows for the table names to be customised, e.g. adding a prefix to all table names.

### Group Save Operations

When several save operations have to be performed for an entity, Magento uses database transactions to ensure that the data stays in a consistent state in the database.

### Zend\_Db\_Select

Magento uses the Zend database abstraction classes like `Zend_Db_Select` to perform database operations.  These classes allow building and executing database queries without having to use the syntax of the specific database engine being used. 

### Collection Interface

*Collection Models* provide a consistent interface for performing and filtering and sorting of models, e.g. `addFieldToFlter()`, `addOrder()` and `setOrder()`.

### Accessing Resource Model Tables

This can be done using the class `Mage_Core_Model_Resource_Db_Abstract` and the methods `getMainTable()` and `getTable($entityName)`.

### Performing Joins

The following methods exist to create joins between tables on collections and on select instances.

- `Zend_Db_Select::join()`
- `Zend_Db_Select::joinInner()`
- `Zend_Db_Select::joinLeft()`
- `Zend_Db_Select::joinRight()`
- `Zend_Db_Select::joinFull()`
- `Zend_Db_Select::joinCross()`
- `Zend_Db_Select::joinNatural()`
- `Mage_Core_Model_Resource_Db_Collection_Abstract::join()`

### Supporting multiple RDBMSs

Magento abstracts datbase engine logic by using the `Varien_Db_Adapter_Interface`.  Database engine classes implement this interface, which makes it easy to replace one engine class with another without having to rewrite all models that use the database. The actual RDBMS used is defined in the connection configurtion using the `<type>` field, e.g. `<type>pdo_mysql</type>`.

### Table Name Lookups

Use `Mage::getModel('core/resource')->getTabelName($modelEntity)` to retreive the defined table for any model.  Table names in Magento are configurable to allow customising and overriding the database schema e.g. using a custom table. 

### Events fired by CRUD operations

- `model_load_before`
- `model_load_after`
- `model_save_commit_after`
- `model_save_before`
- `model_save_after`
- `model_delete_before`
- `model_delete_after`
- `model_delete_commit_after`
- `{collection_event_prefix}_load_before`
- `{collection_event_prefix}_liad_after`

### Magento Insert or Update Query

If an object does not have an ID set or the had *new* property set an `INSERT` query is chosen.  Otherwise it is updated.  There is contingency where if it not new but Magento is unable to find the row to update it will be inserted inserted.

### Filtering Flat Table Collections

Through the use of:

- `$collection->addFilter()`
- `$colletion->getSelect()->where()`

### Ordering Flat Table Collections

Through the use of:

- `$collection->setOrder()`
- `$collection->getSelect()->order()`

The first method goes through the collection interface, which could perform additional logic, while the second method operates directly on the underlying database select statement.

### Setup, Read and Write Database Resouces

Resource models request a specific type of database connection they require.  The different types are defined to allow for different permissions over the database. For example read for read-only connection, write for changing data and setup for resource intensive setup processes.  However, in practice, all of Magento's connection resources inherit from the `default_setup` resource, so they all use the same connection.

## Install and Upgrade Scripts

Magento uses Setup Resource Models to perofrm install and upgrade operations for modules.  These are executed during the application initialisation where each Setup Resource is allowed to apply any updates it requires.  This is usually done by inspecting the installed version of the module from the `core_resource` table and executing any setup scripts defined.

Setup resources are defined in `config.xml`:

{% highlight xml %}
<config>
    <global>
        <resources>
            <{resource_name}>
                <setup>
                    <module>{module_name}</module>
                    <class>{setup_resource_class}</class>
                </setup>
            </{resource_name}>
        </resources>        
    </global>
</config>
{% endhighlight %}

Then the install and upgrade scripts, which are simply PHP scripts executed by including them within the setup resource, are placed in `{module_root}/sql/{resource_name}/` for system install/upgrade scripts or `{module_root}/data/{resource_name}/` for data install/upgrade scripts.

The scripts use the following naming scheme:

- `install-{version}.php`
- `update-{from_version}-{to_version}.php`
- `data-install-{version}.php`
- `data-upgrade-{from_version}-{to_version}.php`

In older version of Magento the install/upgrade script name would be prefixed by resource type, e.g. mysql4.


If the module is not present in the database, the Setup Model will install the module by first running the latest install script and then the upgrade scripts since the install script version.

If the module version in `config.xml` is higher than the version in the database, the Setup Model performs an upgrade by running *all* upgrade scripts which have a `from-version` higher or equal to the database verison and `to-version` lower or equal to the new config version.

### Different Setup Scripts

Different Setup classes have additional methods to aid the install or upgrade procedures of their particular entities, e.g. the EAV setup resource has methods for creating entity attributes.

The base setup classes for flat tables and EAV entities are `Mage_Core_Model_Resource_Setup` and `Mage_Eav_Model_Entity_Setup`, respectively.


### Availble Setup Methods

Methods that are generally available in setup scripts are

- `getTable()`
- `setTable()`
- `updateTable()`
- `run($sql)`


### EAV Attributes Setup Methods

- `addAttribute()`
	- Handles attribute creation, including creating attribute data, adding it to groups and sets and setting attribute options. 
- `updateAttribute()`
	- Simply updates the attribute data.  It is also called by `addAttribute()` if the attribute being added already exists

### Database Rollback

Magento defines a module rollback procedure when the `config.xml` module version is lower than the database version.  However, the rollback script execution is not actually implemented.

<ul class="navigation">
    <li class="prev"><a href="/rendering.html">&larr; Rendering</a>
    <li class="next"><a href="/eav.html">EAV &rarr;</a></li>
</ul>
