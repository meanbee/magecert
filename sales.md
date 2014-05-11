---
layout: default
title: Sales
chapter: 9
meta-description: Order creation and management
---

# Sales

Exam proportion: 11%.

## Important Classes

### `Mage_Sales_Model_Order`

Magento's order model.  It handles everything which associates to the order itself, some of the most important of which are prices, billing and shipping addresses, shipping method, discounts, status, customer and others.  The file is 2000+ lines long so you can imagine what other magic is in there.

### `Mage_Sales_Model_Quote`

This is what an order is before it is converted into an order.  It is built up over the course of a session by a customer and when it is saved it is converted to an order using the `Mage_Sales_Model_Service_Quote::submitAll()` method.  That method using the `Mage_Sales_Model_Convert_Quote` model to facilitate this.

### `Mage_Sales_Model_Service_Order`

This model is used to convert `Mage_Sales_Model_Order` into things like shipment models, invoice models or back to quote items.  `Mage_Sales_Model_Convert_Quote` is used to do the heavy lifting here using *fieldsets*.

### Fieldsets

Defined in `Mage/Sales/etc/config.xml`, fieldsets define what data is copied across orders and quotes when they are converted. For example, when creating an order from a quote, the items, addresses, shipping and payment methods needs moving across.

### `Mage_Sales_Model_Order_Address`

This model represents an address. Its main role is to store all of the data that can be present in an address, e.g. name, street, region etc.  It also validates this data, e.g.

```php
<?php
if (!Zend_Validate::is($this->getFirstname(), 'NotEmpty')) {
    $errors[] = Mage::helper('customer')->__('Please enter the first name.');
}
?>
```

This code validates that an address has a first name associated to it.

## Admin Orders

To create an order within the admin interface, the following steps should be followed.

1. Sales > Orders > Create New Order
2. Select Customer
3. Select Store and enter details
    1. Products
    2. Billing Address
    3. Shipping Address
    4. Shipping Method
    5. Payment Method
4. Place Order


### `Mage_Adminhtml_controllers_Sales_Order_CreateController`

This is the place where the admin form will fire its request to.  Its job is to build up the `Mage_Adminhtml_Model_Sales_Order_Create` object from the data which is input into the admin form.

This controller allows any items to be added to the order (not just saleable items).

The main action within this controller is `loadBlockAction`.  It is hit via AJAX every time a field is blurred on the admin area. 

### `Mage_Adminhtml_Model_Sales_Order_Create`

This is the admin equivalent of the `Mage_Sales_Model_Order` model.  It handles items, addresses, shipping, payment info and creating new quotes from existing orders.

### Calculating Price on Admin Orders

The widget is used to add products to the order.  This is then posted to the backend which is added to the order model in `Mage_Adminhtml_controllers_Sales_Order_CreatController`

```php
<?php
if ($this->getRequest()->has('item') && !$this->getRequest()->getPost('update_items') && !($action == 'save')) {
    $items = $this->getRequest()->getPost('item');
    $items = $this->_processFiles($items);
    $this->_getOrderCreateModel()->addProducts($items);
}
?>
```

The `addProducts()` function here will then add each of these products to the `Mage_Sales_Model_Quote` object and set a flag to indicate that the totals need to be recollected. Then, when saving the quote this flag is checked and the totals are collected.

```php
<?php
public function saveQuote()
{
    if (!$this->getQuote()->getId()) {
        return $this;
    }

    // This is where the flag is checked and totals are collected if it is set
    if ($this->_needCollect) {
        $this->getQuote()->collectTotals();
    }

    $this->getQuote()->save();
    return $this;
}
?>
```

### Editing Orders

An order cannot be edited, rather a new order is created with the required changes made. Therefore, editing orders are treated in much the same as creating orders.  The edit controller actually extends the create controller.  The form fields are populated with the stored data from the saved order and it works exactly the same as the create action.

Order can also be re-ordered, which copies the information from an existing order into a new quote.


### Order States and Statuses

Magento uses order state to determine what state the order is in.  Store admins can use order statuses to give more information on the states (e.g. whether a "New" state order is prepaid or cash on delivery).  

A status can only have one state. For example, the `pending_review` state has both the `fraud` and `payment_review` statuses.  New statuses can be added in the admin interface and assigned to a state.

## Payment Operations

These are the classes and methods that are responsible for payment operations, e.g. authorization and capturing credit cards.

*Capture* acquires the funds for an invoice, based on the specific payment method. While *Pay* is the method where the totals are updated on the invoice.


### `Mage_Sales_Model_Order_Invoice`

This is the model which contains all of the order detail. It is the location where capturing and paying begins.  However, the bulk of the logic is contained in `Mage_Sales_Model_Order_Payment`.

### `Mage_Sales_Model_Order_Invoice/*`

Of interest here is:

- **Item**
	- Contains the logic for each individual item in the invoice. 
- **Comment**
	- Can be applied to an invoice
- **Totals/***
	- A number of totals which are related to an invoice.  Things like subtotal, tax, grand total, etc. 
- **Api**
	- Api Access to invoices, with the ability to create new invoices, capture invoices, cancel invoices etc.

### `Mage_Sales_Model_Order_Payment`

This is where the meat of the capturing and payment occurs.  Capturing is where the funds are actually acquired for the order and payment is updating the order to reflect this.

### `Mage_Payment_Model_Info`

The payment information model.  It is inherited by `Mage_Sale_Model_Order_Payment`.

### `Mage_Payment_Model_Method_Abstract`

The payment methods all inherit this. By default, they include Cc, Checkmo, Free and several others. Validation is performed here, which can be specific to the payment method, e.g. Cc, or handled in the abstract method, e.g. Checkmo.

## Saving to database

### Invoices

Invoice information is stored in several tables in the database.  `additional_information` is used to store payment method specific information.

### Shipping and Tracking

Shipments can be created from Admin.  It is controlled by the `Mage_Adminhtml_Sales_Order_ShipmentController`.  The models which are used to control this are `Mage_Sales_Model_Order_Shipment` and `Mage_Sales_Model_Order_Shipment_Track`.

The data is stored in the `sales_flat_shipment` and other related tables.

Multiple shipments can be created for an order and can be directed to multiple addresses but only for multi-shipping orders.

## Refunds

Order refunds are handled in the `Mage_Adminhtml_Sales_Order_CreditmemoController`, which uses `Mage_Sales_Model_Order_Creditmemo` to process refunds.  Refunds are stored in the `sales_flat_creditmemo` and related tables.

Each payment method needs to specify if it can perform refunds and provide a method to do so.  If the method allows for online refunds, the refund will be automated and processed by the method.  Whereas offline refunds will need to be actioned outside of Magento after an order has been cancelled.

Credit memo totals keep track of the amount being refunded for each part of the order (items, tax, shipping, total, etc.) The order totals get set from them.

Taxes on refunds are processed as configured in admin, i.e on origin, billing or shipping address.

## Partial Order Operations

There are three types of partial operations in Magento orders: invoice, shipping and refund.  Partial means they can affect only part of an order (e.g. two out of five items shipped).  All of these operations set the order to be in processing with the `setIsInProcess()` method.   

Before saving, an order checks its state.  If all operations have been completed fully, it marks itself complete. Otherwise, if `is_in_process` flag is set , it moves to processing state.

Each of the partial operations has its own models and tables to store the data, associating with the order through `order_id` and manipulating the order totals block.

## Cancel

Magento orders an be cancelled until all items have been invoices, e.g. during the pending or processing state.  This automatically cancels payment and order items (which just set cancelled tax amounts on the item).  Invoices can be cancelled, returning order totals to pre-invoice state.  Same goes for credit memos. Only shipments cannot be cancelled.

In most cases when operations are cancelled, tax amounts are returned to the way they were before the operation (just like other price data).  In case of an order, all the cancelled amounts are set to total invoices (whatever has not been invoiced yet is cancelled). 

Invoices and credit memos cannot be cancelled from the interface, even through the functionality was implemented.

## Customers

The `Mage_Customer` module controls handling info about store customers and the customer area., where information can be updated and customer can view their orders.

Customers can also enter addresses to their account so they can quickly check out without having to enter their details each time.

Customers are stored as EAV models, with an EAV table structure.  This means that it is easy to add new attributes, albeit theirs no backend interface for it (in Community Edition).

Customer entities use the `Mage_Customer_Model_Resource_Customer` resource model. Customer data is validated using the `validate()` method on the entity before saving it. The attributes use the standard EAV validation rules for their data.

Most emails sent to customers are managed by Magento's Transactional Emails.  There's a backend interface for creating new email templates (from existing ones, if needed) and they can be assigned to emails in system configuration.

From a customer's perspective, there's no difference between the billing and shipping addresses.  Defaults can be set for both cases to avoid having to select.  The saved addresses are displayed in a select box.

Catalog and shopping cart prices rules and tax can be set specifically for a customer group (e.g. customers in a loyalty group get a discount).   

Customer addresses are also EAV entities, providing an easy way to add custom attributes. Fieldsets are used to copy customer data from Quote to Order, so they have to be taken into account when adding a custom attribute.

Customers can only be in a single group at a time.


<ul class="navigation">
    <li class="prev"><a href="/checkout.html">&larr; Checkout</a>
    <li class="next"><a href="/advanced-features.html">Advanced Features &rarr;</a>
</ul>
