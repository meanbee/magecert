---
layout: default
---

# EAV

## Basic Concepts

- Entity
	- Stores information about the type of the data being stored.  In the case of Magento this is customer, product, category, etc.
- Attribute
	- The individual properties of each of the entities, e.g. name, weight, email address etc.
- Value
	- The value of a given entity and attribute.  For example, we may specify the customer entity and the email attribute and then give it the value `hello@example.com`.

### Database Schema

Important to note that `_varchar` table has the type `varchar` for values even if date or integer would suit the value better.

### Models Versus Resource Models

All the models inside `Mage/Eav/Model/Resource are Mysql4 and are resource models.

In addition `Enttity/Abstract.php` and `Entity/Setup.php`.

### Flat Versus EAV

The EAV models are more complex, providing logic to save and load from multiple tables, whereas the flat models are relatively straightforward (traditional).

### EAV Resource Model Examples

There are a sizeable number of resource models that use EAV

- Customer
- Customer Address
- Customer Payment
- Order
- Order Status
- Order Address
- Order Item
- Order Payment
- Catalog Category
- Catalog Product
- Quote
- Quote Address
- Quote Address Rate
- Quote Address Item
- Quote Item
- Quote Payment
- Order Status History
- Invoice
- Invoice Item
- Invoice Shipment
- Invoice Comment
- Shipment
- Shipment Item
- Shipment Comment
- Shipment Track
- Credit Memo
- Credit Memo Item
- Credit Memo Comment

The reason that EAV is used is so that they can have an underdetermined number of properties and therefore remain flexible.

### Advantages

- Flexibility
- Database schema doesn't need to change with the model
- Quick to implement

### Disadvantages

- Inefficient
	- A query returning 20 columns normally would consist of 20 self joins in EAV.
- By default uses varchar for all values
	- Meanwhile Magento provides a different table for different value types but will still have performance overheads.
- No mechanism for relationships between subtypes
- No grouping of entity subtypes

### Website and Store Scopes

To handle website and store scope attribute values within EAV a `store_id` value exists on the entity and this is used to show scope which linke back to `core_store`. Along with the normal stores (store views) there is also a store '0' which is the global value.  When on a particular store the system will first check for an entity value on the current store and then fall back to the global entity. 

### Insert Versus Update

To determine if an update or an insert needs to be performed, a check is performed to see if the attribute exists on the original object. The original object is basically a duplicate of the data object when the entity was retrieved from the database.  If it does and it's not empty then it updates.  If it does and it is empty then it deletes. If it doesn't and it's not empty then it inserts.

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

{% highlight php %}
<?php
	public function getAllOptions();
	public function getOptionText($value);
?>
{% endhighlight %}

A frontend model does not require any methods in particular.

A backend model requires:

{% highlight php %}
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
	public function setEnttiyValidId($entity, $valudId);
?>
{% endhighlight %}

### System Configuration Source Models

Cannot be used for EAV attributes.  EAV source models implement the `getAllOptions` method while adminhtml source models implement the `toOptionArray()` method.

### Default Models

{% highlight php %}
<?php 
	const DEFAULT_BACKEND_MODEL  = 'eav/entity_attribute_backend_default';
	const DEFAULT_FRONTEND_MODEL = 'eav/entity_attribute_frontend_default';
?>
{% endhighlight %}


### Attribute Source Models

If a source model is not specified for an attribute in the database it gets a default one, which populates options from the `config.xml` or it is empty. 

To get a list of all options for an attribute, perform the following:

{% highlight php %}
<?php 
	$options = $attribute->getSource()->getAllOptions(false);

	// or for admin
	$options = $_attribute->getSource()->getAllOptions(true, true);
?>
{% endhighlight %}


### Stores and Flat Tables

There is a different flat table for each store, each one contains a different store-scoped entity attribute value. 

Multi-lingual values are managed by having different stores for each language.

