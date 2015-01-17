---
layout: default
title: Databases
chapter: 4
meta-description: Discover models, resources models and collections.
---

# Databases

Exam proportion: 13%.

## Models, Resource Models and Collections

### Basic Concepts

A **Model** is used to store and manipulate data about a particular object. Models typically contain the business logic of the application.

A **Resource Model** is used to interact with the database on behalf of the *Model*.  The Resource Model actually performs the CRUD operations.

A **Collection Model** handles working with groups of models and performing (CRUD) operations on groups of models.  

There are two types of Magento Models: simple and EAV.  Simple Models correspond to a single table, whereas EAV correspond to multiple tables using the EAV schema design pattern.

### Database Connection

Database connections are configured in `config.xml`:

```xml
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
```

The core Magento connections (`default_setup`, `default_read`, `default_write`, `core_setup`, `core_read`, `core_write`) are configured in `app/etc/config.xml` with the database credentials stored in `app/etc/local.xml`.

### Working with Database Tables

Magento used the *Resource Models* to interact with the database tables.  When a *Model* is loaded or saved, it calls its resource model to perform the operation (executing the database queries).  Database table names are configured in `config.xml` and resource models retrieve them using look-up methods, which allows for the table names to be customised, e.g. adding a prefix to all table names.

Tables are called *entities* and are configured like so:

```xml
<config>
    <global>
        <models>
            <meanbee>
                <class>Meanbee_Module_Model</class>
                <resourceModel>meanbee_resource</resourceModel>
            </meanbee>
            <meanbee_resource>
                <class>Meanbee_Module_Model_Resource</class>
                <entities>
                    <table_one>
                        <table>meanbee_module_tableOne</table>
                    </table_one>
                </entities>
            </meanbee_resource>
        </models>
    </global>
</config>
```

This allows us to load the table via `getTable('meanbee/table_one')`.


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



### Table Name Lookups

Use `Mage::getModel('core/resource')->getTableName($modelEntity)` to retrieve the defined table for any model.  Table names in Magento are configurable to allow customising and overriding the database schema e.g. using a custom table.

Accessing resource models can be done using the class `Mage_Core_Model_Resource_Db_Abstract` and the methods `getMainTable()` and `getTable($entityName)`.


### Loading Data

The loading of a model from the database is done using the `load($id, $field = null)` method.  The field argument allows the developer to load the records from a different key.  If no field is specified, then the resource model identifies the primary key based on the parameters provided to the `_init($table, $key)` method call when the resource model was constructed.

Magento uses the Zend database abstraction classes like `Zend_Db_Select` to perform database operations.  These classes allow building and executing database queries without having to use the syntax of the specific database engine being used.

When a record is fetched with `Zend_Db_Select` it is added to the `Varien_Object` using `setData($data)`.  Some fields may be serialised in the database, so they are un-serialised before adding to the `Varien_Object`.

### Saving Data

Saving is not as trivial as loading.  The first step is to check to see if the model has any changes. Both `setData($data)` or `unsetData($key, $value)` set the `_hasDataChanges` flag on the model.  This flag can then be used to determine whether it needs to be written out to the database.

A transaction is then begun so that any changes can be rolled back in the event that something goes wrong during the saving process.

Firstly a check for uniqueness is performed.  This queries the database to check whether each of the fields marked as unique are actually unique.  If a duplicate key is found, then it is bubbled up using an exception.

The data then needs to be prepared for insertion or update. This is performed in the `prepareDataForSave()` method.  This calls `DESCRIBE` on the table to identify the column types and sanitise the input accordingly.  This description is, of course, cached.  This is the reason that cache needs to be cleared if schema changes are made.

The next problem is identifying whether an `INSERT` or an `UPDATE` statement is required.  This is usually performed by checking for the existence of the primary key by calling `$model->getId() == null`, and here is no exception.  If there is no ID set, then it is auto-incremented in the database and needs to be fetched from there.  After performing the `INSERT`, `getLastInsertId()` is called on the write adapter to add it to the model.

If there was an ID specified on the model when save was called, it's either because an update to a model is being saved or because the ID of the model is not controlled by the database with an auto-increment. If the flag `_isPkAutoIncrement` is set then an `UPDATE` can be used.  Otherwise the database needs to be checked for an existing record with this key.  An `UPDATE` occurs if there is a key, otherwise an `INSERT` is used.

The `_isPkAutoIncrement` flag is set to `true` as a default and can be overwritten by a subclass.

### Collection Interface

*Collection Models* provide a consistent interface for performing filtering and sorting of models, e.g. `addFieldToFlter()`, `addOrder()` and `setOrder()`.

### Group Save Operations

When several save operations have to be performed for an entity, Magento uses database transactions to ensure that the data stays in a consistent state in the database.

### Filtering Flat Table Collections

Through the use of:

- `$collection->addFilter()`
- `$collection->getSelect()->where()`

### Ordering Flat Table Collections

Through the use of:

- `$collection->setOrder()`
- `$collection->getSelect()->order()`

The first method goes through the collection interface, which could perform additional logic, while the second method operates directly on the underlying database select statement.

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

### Setup, Read and Write Database Resources

Resource models request the specific types of database connections they require.  The different types are defined to allow for different permissions over the database. For example, read for read-only connection, write for changing data, and setup for resource intensive setup processes.  However, in practice, all of Magento's connection resources inherit from the `default_setup` resource, so they all use the same connection.

## Install and Upgrade Scripts

Magento uses Setup Resource Models to perform install and upgrade operations for modules.  These are executed during the application initialisation where each Setup Resource is allowed to apply any updates it requires.  This is usually done by inspecting the installed version of the module from the `core_resource` table and executing any setup scripts defined.

Setup resources are defined in `config.xml`:

```xml
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
```

Then the install and upgrade scripts, which are simply PHP scripts executed by including them within the setup resource, are placed in `{module_root}/sql/{resource_name}/` for system install/upgrade scripts or `{module_root}/data/{resource_name}/` for data install/upgrade scripts.

The scripts use the following naming scheme:

- `install-{version}.php`
- `update-{from_version}-{to_version}.php`
- `data-install-{version}.php`
- `data-upgrade-{from_version}-{to_version}.php`

In older versions of Magento, prior to Magento CE 1.6 and Magento EE 1.11, the install/upgrade script name would be prefixed by resource type, e.g. mysql4.


If the module is not present in the database, the Setup Model will install the module by first running the latest install script and then the upgrade scripts since the install script version.

If the module version in `config.xml` is higher than the version in the database, the Setup Model performs an upgrade by running *all* upgrade scripts which have a `from-version` higher or equal to the database verison and `to-version` lower or equal to the new config version.

### Different Setup Scripts

Different Setup classes have additional methods to aid the install or upgrade procedures of their particular entities, e.g. the EAV setup resource has methods for creating entity attributes.

The base setup classes for flat tables and EAV entities are `Mage_Core_Model_Resource_Setup` and `Mage_Eav_Model_Entity_Setup` respectively.


### Available Setup Methods

Methods that are generally available in setup scripts are:

- `startSetup()`
- `endSetup()`
- `getTable()`
- `setTable()`
- `updateTable()`
- `run($sql)`


### EAV Attributes Setup Methods

- `addAttribute()`
	- Handles attribute creation, including creating attribute data, adding it to groups and sets and setting attribute options.
- `updateAttribute()`
	- Simply updates the attribute data.  It is also called by `addAttribute()` if the attribute being added already exists.

### Database Rollback

Magento defines a module rollback procedure when the `config.xml` module version is lower than the database version.  However, the rollback script execution is not actually implemented.

### Supporting multiple RDBMSs

Magento abstracts database engine logic by using the `Varien_Db_Adapter_Interface`.  Database engine classes implement this interface, which makes it easy to replace one engine class with another without having to rewrite all models that use the database. The actual RDBMS used is defined in the connection configuration using the `<type>` field, e.g. `<type>pdo_mysql</type>`.

<ul class="navigation">
    <li class="prev"><a href="/rendering.html">&larr; Rendering</a>
    <li class="next"><a href="/eav.html">EAV &rarr;</a></li>
</ul>
