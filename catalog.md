---
layout: default
title: Catalog
chapter: 7
meta-description: Find out about categories, products, layered navigation and taxes.
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
	- No physical item required for delivery, e.g. services
- Downloadable
	- A digital rather than physical product.

Most product types are implemented as part of the `Mage_Catalog` module, apart from `Mage_Bundle` and `Mage_Downloadable`.


Grouped, Bundle and Configurable products implement a parent-child relationship where a number of other (by default, simple, virtual or downloadable) products get assigned to a main product.  This then handles the product data for the whole collection (e.g. group, bundle, or configurable product name, price and status).

Downloadable and Bundle products have extra tables in the database, meanwhile the rest are shared amongst all other product types.  Configurable products have an extra table to link to child products, `catalog_product_super_link`.

### Custom Product Type

To create a product type that extends one of the built-in product types, the corresponding product type model should be extended. Otherwise the new product type should extend the `Mage_Catalog_Model_Product_Type_Abstract` class.

An entry in the module's `config.xml` is also required:

```xml
<global>
	<catalog>
		<product>
			<type>
				<{name}>
					<label></label>
					<model></model>
					<composite></composite>
					<index_priority></index_priority>
				</{name}>
			</type>
		</product>
	</catalog>
</global>
```

More complicated products may require other customised areas such as price model and index data retriever.

### Price Calculation

When dealing with a single product, the price is always calculated on the fly.  The price EAV attribute is loaded with the product and final price is calculated by the price model, `Mage_Catalog_Model_Product_Type_Price`.

Some product types deal with it differently.  In which case they extend this class and implement their own logic.  For example, the configurable product overwrites `getFinalPrice()` and adds additional logic.  This custom model can then be specified in `config.xml` with a `<price_model>` tag.

Product collections, however, use the price index to retrieve pre-calculated prices, eliminating the need to calculate it for each product.

Final price can be adjusted by the observers of the `catalog_product_get_final_price` event.  By default, only the `Mage_CatalogRule` module observes this event.

Another method to override produce price is to simply set it on the product.  If the price is set, the product will not recalculate it.

Product tier price is separate from normal price (although taken into account when calculating price).  It's implemented as a list with a customer group and minimum quantity qualifiers for each tier.   Tier prices are displayed in a table, using the `catalog/product/view/tierprices.phtml` template.

Custom product options get processed when calculating final price.  Each option has its own price defined, which gets added to the final price.

Group, tier and special prices all get considered at the same time (`$priceModel->getBasePrice()`) and the smallest one of the three (or four if you include the regular price) is chosen as the base product price.


### Tax

The `Mage_Tax` module uses a product's tax class and whether or not the product price is inclusive or exclusive of tax in order to identify the correct rate to apply.

The following factors are used to calculate tax on products:

- Product tax class
- The amount of tax already included
- Billing and Shipping addresses
- Customer tax class
- Store settings


### Layered Navigation


The classes responsible for rendering the layered navigation are:

- `Mage_Catalog_Block_Layer_View`
	- Handles the filters and options
- `Mage_Catalog_Block_Layer_State`
	- Controls what is currently being filtered by

To implement layered navigation on attributes with custom source models the `Mage_Catalog_Model_Layer_Filter_Abstract::apply()` method would need to be overwritten to dictate how the product collection should be filtered.

Layered navigation is rendered by the `Mage_Catalog_Block_Layer_View` and `Mage_Catalog_Block_Layer_State` blocks, which use filter blocks for individual filters.

Layered Navigation uses index table for most filters, e.g. price, product attribute index, decimal product index.

## Categories

### Categories in the Database

Category hierarchy is managed by storing a category's parent id. The full hierarchy is shown in the `path` column (slash separated IDs).  There is a special category with `parent_id` of `0`. This is the true root category and each of the other *root* categories as defined in Magento use this as a shared parent.

To read and manage a category tree from the database two different classes are used depending if flat catalog is enabled, `Mage_Catalog_Model_Resource_Category_Tree` and `Mage_Catalog_Model_Resource_Category_Flat`.

The advantage of flat categories is that it is quicker to query. However, it needs to be rebuilt from the EAV tables each time there is a change.

- `getChildren()`
	- returns a comma separated string of immediate children IDs
- `getAllChildren()`
	- returns a string or array of all children IDs
- `getChildrenCategories()`
	- returns a collection of immediate children categories

N.B. If flat catalog is enabled, the only child categories returned will be ones with `include_in_menu = 1`.  In both cases, only active categories are returned.


### Catalog Price Rules

Catalog price rules apply discounts to products based on the date, product, website and customer group.

When `getFinalPrice()` is called on a product, the event `catalog_product_get_final_price` is fired. This is observed by `Mage_CatalogRule_Model_Observer` which will then look for any catalog price rule that applies to the product.  If applicable, it then looks at the database price table and writes the price back to the product model as a Varien data field `final_price`.

Within the database, the `catalogrule` table describes rules, their conditions and their actions.  `catalogrule_product` contains the matched products and some rule information.  Meanwhile `catalogrule_product_price` contains the price after the rule has been applied.

## Indexing and Flat Tables

Flat catalog tables are managed by catalog indexers. If automatic rebuilding of the indexes is enabled, the catalog indexers get rebuilt every time a product, category or any related entities are updated.  The `_afterSave()` method calls the indexer process.  Otherwise they have to be manually re-indexed through admin.

Product type affects price index and stock index where products can define their own custom indexers (in `config.xml`) to handle their data for these indexes.

The `Mage_Index` module provides the framework with which custom indexes can be created to help optimise the performance of the site.  The `Mage_Index_Model_Indexer_Abstract` class should be extended to create a new index, implementing the `_registerEvent()` and `_processEvent()` methods. Not forgetting to register it in `config.xml`:

```xml
<global>
	<index>
		<indexer>
			<{name}>{model}</{name}>
		</indexer>
	</index>
</global>
```


<ul class="navigation">
    <li class="prev"><a href="/eav.html">&larr; EAV</a></li>
    <li class="next"><a href="/checkout.html">Checkout &rarr;</a></li>
</ul>
