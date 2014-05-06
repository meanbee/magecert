---
layout: default
title: Sales |
---

# Sales

Exam proportion: 11%.

## Admin Order Creation

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

## Important Classes


### `Mage_Sales_Model_Order`

Magento's order model.  It handles everything which associates to the order itself, some of the most important of which are prices, billing and shipping addresses, shipping method, discounts, status, customer and others.  The file is 2000+ lines long so you can imagine what other magic is in there.

### `Mage_Sales_Model_Quote`

This is what an order is before it is converted into an order.  It is built up over the course of a session by a customer and when it is saved it is converted to an order using the `Mage_Sales_Model_Service_Quote::submitAll()` method.  That mehtod using the `Mage_Sales_Model_Convert_Quote` model to facilitte this.

### `Mage_Sales_Model_Service_Order`

This model is used to convert `Mage_Sales_Model_Order` into things like shipment models, invoice models or back to quote items.  `Mage_Sales_Model_Convert_Quote` is used to do the heavy lifting here using *fieldsets*.

### Fieldsets

Defined in `Mage/Sales/etc/config.xml`, fieldsets define what data is copied across orders and quotes when they are converted. For example, when creating an order from a quote, the items, addresses, shipping and payment methods needs moving across.

### `Mage_Sales_Model_Order_Address`

This model represents an address. Its main role is to store all of the data that can be present in an address, e.g. name, street, region etc.  It also validates this data, e.g.

{% highlight php %}
if (!Zend_Validate::is($this->getFirstname(), 'NotEmpty')) {
    $errors[] = Mage::helper('customer')->__('Please enter the first name.');
}
{% endhighlight %}

This code validates that an address has a first name associated to it.

### `Mage_Adminhtml_controllers_Sales_Order_CreateController`

This is the place where the admin form will fire its request to.  Its job is to build up the `Mage_Adminhtml_Model_Sales_orderCreate object from the data which is inputted into the admin form.

The main action within this controller is `loadBlockAction`.  It is hit via AJAX every time a field is blurred on the admin area. 

### `Mage_Adminhtml_Model_Sales_Order_Create`

This is the admin equivalent of the `Mage_Sales_Model_Order` model.

## Calculating Price on Admin Orders

The widget is used to add products to the order.  This is then posted to the backend which is added to the order model in `Mage_Adminhtml_controllers_Sales_Order_CreatController`

{% highlight php %}
<?php
if ($this->getRequest()->has('item') && !$this->getRequest()->getPost('update_items') && !($action == 'save')) {
    $items = $this->getRequest()->getPost('item');
    $items = $this->_processFiles($items);
    $this->_getOrderCreateModel()->addProducts($items);
}
?>
{% endhighlight %}

The `addProducts()` function here will then add each of these products to the `Mage_Sales_Model_Quote` object and set a flag to indicate that the totals need to be recollected. Then, when saving the quote this flag is checked and the totals are collected.

{% highlight php %}
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
{% endhighlight %}

## Editing Orders

An order cannnot be edited, rather a new order is created with the required changes made. Therefore, editing orders are treated in much the same as creating orders.  The edit controller actually extends the create controller.  The form fileds are populated with the stored data from the saved order and it works exactly the same as the create action.


## Order States and Statusses

An order state can have one or more order statuses.  A status can only have one state. For example, the `pending_review` state has both the `fraud` and `payment_review` statuses.


## Card Operations

These are the classes and methods that are responsible for credit card operations, e.g. authorization and capturing.

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


## Pay versus Capture

*Capture* acquires the funds for an invoice, based on the specific payment method. While *Pay* is the method where the totals are updated on the invoice.

## Saving to database

### Invoices

Invoice information is stored in several tables in the database.  `additional_information` is used to store payment method specific information.

### Shipping and Tracking

The models which are used to control this are `Mage_Sales_Model_Order_Shipment` and `Mage_Sales_Model_Order_Shipment_Track`.

## Refunds

`Mage_Sales_Model_Order_Creditmemo` handles refunds on an order.  Each payment method needs to specify if it can perform refunds and provide a method to do so.  If the method allows for online refunds, the refund wil be automated and processed by the method.  Whereas offline refunds will need to be actioned outside of Magento after an order has been cancelled.

Taxes on refunds are processsed as configured in admin, i.e on origin, billing or shipping address.

## Partial Order Operations

## Cancel

An order can be cancelled when it is in the pending or processing state but not after it has been invoiced.

<ul class="navigation">
    <li class="prev"><a href="/checkout.html">&larr; Checkout</a>
    <li class="next"><a href="/advanced.html">Advanced Features &rarr;</a>
</ul>
