---
layout: default
title: Checkout |
---

# Checkout

Exam proportion: 16%.

## Quotes

Magento uses the quote model to store information about an order before it is places.  It contains:

- Customer information
- Items to be ordered
- Billing Address
- Shipping Address
- Shipping Method
- Payment Method
- Price Totals


Billing and Shipping addresses use the same model, `Mage_Sales_Model_Quote_Address`.  The type of the address is set as a field on the model.

The shipping address (or billing address if the product is virtual or downloadable) is used to calculate the total for the order and avaialble shipping methods.

At the end of the checkout process a quote is fulfilled and converted into an order.

### Adding to Quotes

The Shopping Cart model is used for manipulating (adding, removing and updating) the itesm in a quote.  It performs additional validation, such as setting the minimum order quantity on an item when it is added to the basket.

As a product is being added to cart, the `_prepareProduct()` method gets called on it to prepare the product data for storage in the quote.  For simple products, this loads all of the product data into the model (including product options, to be stored in `sales_flat_quote_item_option` table).  Grouped product, for example, load all associated products.

For product information such as custom and configurable options, it is converted to an `Mage_Sales_Quote_Item_Option` object and stored in the database in the `sales_flat_quote_item_option` table.

### Quotes in the Database

Quotes are stored in the database.  The majority is stored in the `sales_flat_quote` table but more complex information is stored in additional tables that link back to the original quote.


## Checkout

### Checkout Flow Options

There are two checkouts by default, onepage and multishipping.  In multishipping checkout, the quote items are added on each shipping address (virtual items are added to the billing address instead) rather than the quote itself and at the end of the checkout and order is created for each address.  This is reflected in the database, where the multishipping quote items are stored in `sales_flat_quote_address_item` while the onepage quote items are in `sales_flat_quote_item`.

Multishipping checkout uses a separate controller `Mage_Checkout_MultishippingController` and a custom checkout type `Mage_Checkout_Model_Type_Mulitishipping`.  Apart from that, it reuses many of the same modesl as the regular checkout.  To cusotmise of extend the multishipping checkout those two classes should be extended and modified.

In a multishipping checkout, the virtual items are added to the billing address (instead of one of the shipping addresses), as a virtual quote would be one which has items on the billing address only.  Quotes with virtual items create an additional order from the billing address containing those items.

Grouped products get added to the cart as multiple separate products, so they can be selected to be shipped to different addresses using the regular multishipping checkout process.  However, bundled products cannot be split among multiple addresses.

### Totals

The quote items, shopping cart price rules and shipping costs contribute to the total cost of the quote.

The billing and shipping addresses also affect the total as these are used to determine the tax rates, shipping methods and payment methods to be made available.

The total models keep track of the cost of an order or quote.  Each of the totals has a code associated with it that can be used to retrieve and manipulate it.  Total modesl get passed an address to collect totals for.  Through the address the models gain access to the quote and quote items.

These models can be rewritten or new total models can be added by extending the `Mage_Sales_Quote_Address_Total_Abstract` model and registering it in `config.xml`.

{% highlight xml %}
<config>
    <global>
        <sales>
            <quote>
                <totals>
                    <{code}>
                        <class>{grouped_class_name}</class>
                        <before>{csv_other_totals}</before>
                        <after>{csv_other_totals}</after>
                    </{code}>
                </totals>
            </quote>
        </sales>
    </global>
</config>
{% endhighlight %}

The priority of a total model execution can be customise using the `<before>` and `<after>` elements in the total definition.  However the default order of execution is:

1. Nominal
2. Subtotal
3. Shipping
4. Tax
5. Grand Total

This process is managed by the `Mage_Sales_Model_Quote_Address_Total_Collection` model which is called from the Address using the `collectTotals()` method.  This method is called whenever the quote is updated, e.g. at each stage of the checkout. 

### Payment Authorisation and Capturing

Card authorisation and capture occurs when an order is placed (`$order->place()`) which occures when the order is saved. Depending on the payment method authorisation can be reserved for this time and then the capturing occurs once an invoice is created.

### Inventory Decrements

For onepage checkout, the inventory is decremented by a `Mage_CatalogInventory` observer on the `sales_model_server_quote_submit_before` event.  This event is dispatched just before the order is placed, and the `checkout_submit_all_after` event, which is fired at the end of the onepage checkout (the observer has checks to prevernt decrementing the inventory twice) and after all of the orders have been created in multishipping checkout.

## Shopping Cart Price Rules

Shopping cart price rules are used to apply discounts to a cart and may be given a set of conditions of when the apply. This functionality is provided by the `Mage_SalesRule` module.

In contrast to catalog price rules, shopping cart price rules apply price changes (discounts) based on the information in the quote, e.g. customer group, voucher code, rather than on a product or category basis.

The limitations of shopping cart rules include:

- While multiple rules can apply to the same cart, only one voucher code can be applied at once.
- Rules work independently of each other, so a rule cannot be specified to be active only if no other rules apply.
- For admin area orders, shopping cart price rules can be disabled on specific items but they are always applied on all items in the frontend.


## Shipping Methods

In Magento, the shipping methods extend the `Mage_Shipping_Model_Carrier_Abstract` model and are registered in the store configuration, with the default values (and static configuration like model names) set in `config.xml`.

{% highlight xml %}
<config>
    <default>
        <carriers>
            <{code}>
                <active>{0|1}</active>
                <sallowspecific>{0|1}</sallowspecific>
                <model>{grouped_class_name}</model>
                <name>{name}</name>
                <title>{title}</title>
                ...
            </{code}>
        </carriers>
    </default>
</config>
{% endhighlight %}

Shipping methods are called carriers, because they defined a method of shipping, e.g. Royal Mail or Fedex, and can offer multiple rates within that method, e.g. Standard or Next Day delivery.

Existing shipping methods provided by Magento can be customised by rewriting their model.

### Shipping Rates Calculation

The shipping rate calculate is invoked in the Quote Address (`requestShippingRates()`), which builds up a request containing all relevant data, such as destination address and package weight and calles the `Mage_Shipping_Model_Shipping::collectRates($request)` method.  It loops over all available carriers, validates the request and calls the `collectRates($request)` method for each one to get the rates offered.

#### TableRates

The TableRate shipping method allows creating a set of rules, specified as a csv, to set the shipping price based on the destination and either order price, weights or number of items.  The factor used is set on a store level and must be used throughout the store, e.g. price and weight rules cannot be mixed on a store level.

#### US Shipping Methods

Some shipping carriers make requests to third party service to get the rates offered. FedEX, IPS and USPS all extend the `Mage_Usa_Model_Shipping_Carrier_Abstract` model and retrieve their rates through HTTP or SOAP requests.

## Payment Methods

Payment methods in Magento, like shipping methods, extend the `Mage_Payment_Model_Method_Abstract` model and are registered using the store configuration, with the default values (and additional configuration) set in `config.xml`.

{% highlight xml %}
<config>
    <default>
        <payment>
            <{name}>
                <active>{0|1}</active>
                <model>{grouped_class_name}</model>
                <order_status>{order_state}</order_status>
                <title>{title}</title>
                <allowspecific>{0|1}</allowspecific>
                <sort_order>{sort_order}</sort_order>
                <group>{group}</group>
                <payment_action>{action}</payment_action>
                ...
            </{name}>
        </payment>
    </default>
</config>
{% endhighlight %}

Most of the payment method parameters can usually be customised in System Configuration or by manipulating the default values in `config.xml`.  For example, the place order payment action is usually defined in the `<payment_action>` configuration field. If deeper customisation is required, the payment method models can be rewritten.

Payment information gets added to methods using the `Mage_Sales_Model_Quote_Payment::importData($data)` method and is called by the `savePayment()` method in the onepage checkout.  This fires the `sales_quote_payment_import_data_before` event before called `assignData($data)` method on the payment method model.  By observing this event, the data can be modified before it gets stored in the method.

Payment methods implement the logic of a particular method of payment, e.g. processing credit carts.  Meanwhile, payment models manage payment methods within a quote or order and handle payment related operations such as capturing payment or refunds.  

### Billing Agreements

To make use of billing agreements within a payment method, the billing agreement data needs to be set on a payment model,

{% highlight php %}
<?php $payment->setBillingAgreementData($data); ?>
{% endhighlight %}

<ul class="navigation">
    <li class="prev"><a href="/catalog.html">&larr; Catalog</a>
    <li class="next"><a href="/sales.html">Sales &rarr;</a>
</ul>











