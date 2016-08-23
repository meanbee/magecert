---
layout: default
title: EAV
chapter: 5
meta-description: Entity Attribute Value tables, explained.
---

# EAV

Exam proportion: 10%.

## Basic Concepts

- Entity
	- Stores information about the type of the data being stored.  In the case of Magento this is customer, product, category, etc.
- Attribute
	- The individual properties of each of the entities, e.g. name, weight, email address etc.
- Value
	- The value of a given entity and attribute.  For example, we may specify the customer entity and the email attribute and then give it the value `hello@example.com`.

### Database Schema

- `eav_entity`
	- The entity table.
- `eav_entity_attribute`
	- The attribute table.
- `eav_entity_{type}`
	- The values tables.  Types are datetime, decimals, int, text and varchar.

Important to note that `eav_entity_varchar` table has the type `varchar` for values even if date or integer would suit the value better.

### Models Versus Resource Models

All the models inside `Mage/Eav/Model/Resource` are Mysql4 and are resource models.

In addition `Entity/Abstract.php` and `Entity/Setup.php`.

### Flat Versus EAV

The EAV models are more complex, providing logic to save and load from multiple tables, whereas the flat or standard models are relatively straightforward (traditional).

Standard models mainly manage their properties with data setters and getters working with a single table.  EAV models mainly manage their attribute models.  Standard models only save their data to a table and load from it.  EAV models load all (or a specific set) attributes after loading base data and save attributes after saving data (including inserting, updating and deleting attributes).

### EAV Resource Model Examples

To find entities using the EAV storage schema searching for the string `extends Mage_Eav_Model_Entity_Abstract` can be used.  
This should reveal all resource models based on the EAV structure. However, the resulting list will include many obsolete classes from the `Mage_Sales` module that no longer are being used.  

The only modules containing entities using the EAV storage schema are `Mage_Catalog` (categories and products) and `Mage_Customer` (customers and addresses).

Customer Groups use the flat table storage schema. All the sales entities where converted to flat table entities with the release of Magento 1.4.

The reason that EAV is used is so that entities can have an undetermined number of properties and therefore remain flexible. For example, when you add a new attribute to a Customer entity (which is an EAV entity), the database table does not need to be altered for this new attribute to be added.

### Advantages

- Flexibility
- Database schema doesn't need to change with the model
- Quick to implement

### Disadvantages

- Inefficient
	- A query returning 20 columns normally would consist of 20 self joins in EAV. However, the Mage_Eav module generally does not use joins to load the attribute value data. Instead union selects are used. Joins are only used for filtering EAV collections.
- No mechanism for relationships between subtypes
- No grouping of entity subtypes

### Website and Store Scopes

To handle website and store scope attribute values within EAV, a `store_id` value exists on the catalog entity to show scope which links back to `core_store`. Along with the normal stores (store views) there is also a store '0' which is the global value.  When on a particular store, the system will first check for an entity value on the current store and then fall back to the global entity. Mage_Customer EAV entities do not have a `store_id` scope column.


### Insert, Update and Delete

To determine if an insert, update or delete needs to be performed on an attribute, a comparison is made against the original object. The original object is basically a duplicate of the data object when the entity was retrieved from the database.  
- If the attribute exist originally and its new value is not empty; it updates.  
- If the attribute exist originally but its new value is set to empty; it deletes.
- If the attribute doesn't exist originally and its new value is not empty; it inserts.

## Attribute Management

### Attribute Models

- Attribute Model
	- Represents the attribute in the database form, its logic is standard across all attributes and is difficult to change.
- Frontend Model
	- The attribute's interface to the frontend and provides any logic that the attribute requires on the frontend, e.g. the `getUrl()` method on images.
- Backend Model
	- These perform validation on the attribute before it is saved to the database.  For example, the password backend model converts the password into a hash before it is saved.  It also checks that the password and password confirmation match before saving.
- Source Models
	- Used to populate the options available for an attribute, e.g. `catalog/product_status` has *enabled* and *disabled*.

### Required Methods

A source model requires:

```php
<?php
	public function getAllOptions();
	public function getOptionText($value);
?>
```
Usually only `getAllOptions()` needs to be implemented though since an implementation for `getOptionText()` already exists in the abstract source model `Mage_Eav_Model_Entity_Attribute_Source_Abstract`.

A frontend model does not require the method `getValue()`.

A backend model requires:

```php
<?php
	public function getTable();
	public function isStatic();
	public function getType();
	public function getEntityIdField();
	public function setValueId($valueId);
	public function getValueId();
	public function afterLoad($object);
	public function beforeSave($object);
	public function afterSave($object);
	public function beforeDelete($object);
	public function afterDelete($object);
	public function getEntityValueId($entity);
	public function setEntityValidId($entity, $valueId);
?>
```
All these methods are implemented in the abstract backend model `Mage_Eav_Model_Entity_Attribute_Backend_Abstract`. For custom backend models only the methods requiring customisation need to be overridden.

### System Configuration Source Models

Cannot be used for EAV attributes.  EAV source models implement the `getAllOptions` method while adminhtml source models implement the `toOptionArray()` method.

Default system configuration source models can be found in `Mage/Adminhtml/Model/System/Config/Source/`.

### Attribute Source Models

The purpose of Attribute Source Models is to supply the list of options and values for select and multiselect attributes.  They also supply the column information to the catalog flat table indexer if required.

To get a list of all options for an attribute, perform the following:

```php
<?php
	$options = $attribute->getSource()->getAllOptions(false);

	// or for admin
	$options = $_attribute->getSource()->getAllOptions(true, true);
?>
```

### Default Attribute Models

If no class is specified as a frontend, backend or - for select or multiselect attributes - source models, a default class is used.  

The default attribute frontend model is `Mage_Eav_Model_Entity_Attribute_Frontend_Default`.  

The default attribute backend model depends on the attribute code and is determined in the method `Mage_Eav_Model_Entity_Attribute::_getDefaultBackendModel()`.

```php
<?php
    protected function _getDefaultBackendModel()
    {
        switch ($this->getAttributeCode()) {
            case 'created_at':
                return 'eav/entity_attribute_backend_time_created';

            case 'updated_at':
                return 'eav/entity_attribute_backend_time_updated';

            case 'store_id':
                return 'eav/entity_attribute_backend_store';

            case 'increment_id':
                return 'eav/entity_attribute_backend_increment';
        }

        return parent::_getDefaultBackendModel();
    }
?>
```
If the method falls through to the last line `Mage_Eav_Model_Entity_Attribute_Backend_Default` is used.  

The default source model is set in `Mage_Eav_Model_Entity_Attribute_Source_Table`. This is set in the catalog modules attribute model. The default config source model specified in the eav module is never used.



### Add Attribute

To add EAV attributes, use `Mage_Eav_Model_Entity_Setup` by extending in the setup class.

- `addAttribute()`
	- Creates the attributes, add it to groups and sets (including default) , or updates if it already exists
- `updateAttribute()`
	- Updates the attribute data only.

Custom setup classes can be used to extend these methods, adding additional data or simplifying the arguments needed.

### Flat Tables

Flat catalog attributes are managed by indexers:

- `Mage_Catalog_Model_Resource_Product_Flat_Indexer::updateAttribute()`
- `Mage_Catalog_Model_Resource_Category_Flat::synchronise()`

Product attributes get added to the flat table if they are (see `Mage_Catalog_Model_Resource_Product_Flat_Indexer::getAttributeCodes()`):

- Static (backend type)
- Filterable
- Used in product listing
- Used for promo rules
- Used for sort by
- System Attributes

There is a different flat table for each store, each one contains a different store-scoped entity attribute value. Multi-lingual values are managed by having different stores for each language.



<ul class="navigation">
    <li class="prev"><a href="/databases.html">&larr; Databases</a>
    <li class="next"><a href="/adminhtml.html">Adminhtml &rarr;</a>
</ul>

