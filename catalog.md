---
layout: default
title: Catalog |
---

# Catalog

Exam proportion: 10%.

## Products

There are six different product types built-in to Magento.

- Simple
	- A single stock unit
- Configurable
	- First of the composite products.  Allow customers to configure their product and add a single simple product to basket.
- Grouped
	- The second composite product, a grouped product relates simple products and provides customers with the ability to choose quantities of each item.
- Bundle
	- The third composite product type , a bundle relates simple products together to purchase as a single item.
- Virtual
	- No physical item required for delivery.
- Downloadable
	- A digital rather than physical product.

Most product types are implemmented as part of the `Mage_Catalog` module, apart from `Mage_Bundle` and `Mage_Downloadable`.

### Custom Product Type

To create an product type that extends a pre-available product type, this should be done by extending the relevant model. Otherwise the abstract product model should be inherited `Mage_Catalog_Model_Product_Type_Abstract`. 

An entry in the module's `config.xml` is also required:

{% highlight xml %}
	<catalog>
		<product>
			<type></type>
		</product>
	</catalog>
{% endhighlight %}

More complicated products may require other customised areas such as price model and index data retriever.

### Price Calculation

Price calculation is handled in `Mage_Catalog_Model_Product_Type_Price` by default.  Some types deal with it differently.  In which case they just extend this class and implement their own logic.  For example, the configurable product overwrites `getFinalPrice()` and adds additional logic.

The `Mage_Tax` module uses a product's tax class and whether or not the product price is inclusive or exclusive of tax in order to identify the correct rate to apply.

### Database Tables

Most product types share database tables apart from the product types that were shipping as separately modules, i.e. Bundle and Downloadable.

Magento can index the product EAV tables and creat flat versions of them for speed.  The indexers are run each time a product is saved and the relevant rows in the flat tables are updated.

### Layered Navigation

The classes responsible for rendering the layered navigation are:

- `Mage_Catalog_Block_Layer_View`
	- Handles the filters and options
- `Mage_Catalog_Block_Layer_State`
	- Controls what is currently being filtered by

To implement layered navigation on attributes with custom source models the `Mage_Catalog_Model_Layer_Filter_Abstract::apply()` method would need to be overwritten to dictate how the product collection should be filtered.

## Categories

## Categories in the Database

Category hierarchy is managed by storing a category's parent id. The full hierarchy is shown in the `path` column.  There is a special category with parent_id of `0`. This is the true root category and each of the other *root* categoryies as defined in Magento use this as a shared parent.

To read and manage a category tree from the database two different classes are used depending if flat catalog is enabled, `Mage_Catalog_Model_Resource_Category_Tree` and `Mage_Catalog_Model_Resource_Category_Flat`.

The advantage of flat categories is that it is quicker to query. However, it needs to be rebuilt from the EAV tables each time there is a change.

## Children Categories

There are two methods to find the children of a category

- `getChildren()`
	- Returns a list of IDs
- `getChildrenCategories()`
	- Returns `Mage_Catalog_Model_Resource_Category_Collection`

## Catalog Price Rules

When `getFinalPrice()` is called on a product an event is fired.  This event, `catalog_product_get_final_price` is observed the `Mage_CatalogRule_Model_Observer` which will then look for any catalog price rule that applies to the product.  If it needs, it then looks at the database price table and writes the price back to the product model as a Varien data field `final_price`.

Within the database, the `catalogrule` table describes rules, their conditions and their actions.  `catalogrule_product` contains the matched products and some rule information.  Meanwhile `catalogrule_product_price` contains the price after the rule has been applied.


## Indexing

The `Mage_Index` module provides a framework with which custom indexes can be created to help optimise the performance of the site.  The class that needs to be extended is `Mage_Index_Model_Resource_Abstract`.

<ul class="navigation">
    <li class="prev"><a href="/eav.html">&larr; EAV</a>
    <li class="next"><a href="/checkout.html">Checkout &rarr;</a>
</ul>





