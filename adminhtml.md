---
layout: default
title: Adminhtml
chapter: 6
meta-description: Manage admin area forms and grids.
---

# Adminhtml

Exam proportion: 7%.

## Architecture

### Configuration

Configuration areas that are loaded for the admin area are

- `<menu>`
- `<acl>`
- `<admin>`
- `<adminhtml>`

### Adminhtml Controller

The adminhtml controller extends `Mage_Adminhtml_Controller_Action`.  In contrast to the frontend controllers which extend `Mage_Core_Controller_Front_Action`.

This action checks if the user is allowed to access the requested page and validates form and secrete keys before dispatch.

### Routing

In `Mage_Core_Controller_Varien_Front::dispatch()`, Magento loops over each of its routers and checks against the path specified by the request to see if any modules have registered against this frontname.  From there, the module specified in its controller (defined by `config.xml`) which controller class it extends (the admin or the front one) and in each of these the area is defined.

To use the `/admin` route, either `Mage_Adminhtml will need to be overwritten or define your frontname:

{% highlight xml %}
<admin>
    <routers>
        <cloudiq>
            <use>admin</use>
            <args>
                <frontName>your_frontname</frontName>
                <module>Your_Module</module>
            </args>
        </cloudiq>
    </routers>
</admin>

<admin>
    <routers>
        <adminhtml>
            <args>
                <modules>
                    <your_module before="Mage_Adminhtml">Your_Module</your_module>
                </modules>
            </args>
        </adminhtml>
    </routers>
</admin>
{% endhighlight %}


### Authentication

The observer `Mage_Admin_Model_Observer` listens for the `predispatch` event of an admin controller action.  In here, there is logic to see the user is logged in.  If there's no used but there is login post data in the request, it logs the customer in. Otherwise, the customer is redirected to the login page. Finally it refreshes the ACL cache.


### Adminhtml Blocks

Most adminhtml blocks extend `Mage_Adminhtml_Block_Template`.

### Adminhtml Config

The role of the adminhtml configuration is to provide a mechanism within the admin area for users to be able to set configuration options for stores.

`Mage_Admin_Model_Config` uses `adminhtml.xml` to provide admin area specific configuration e.g. menu items.

`Mage_Adminhtml_Model_Config` uses `system.xml` to provide system configuration parameters.

### Cache Management

Different cache types related to different portions of the cache.  Caches can associate with a particular type, which allows clearing only certain portions of the cache.

- Flush Magento Cache
	- This removes only the cache entries that are managed by Magento itself, e.g. (`MAGE` and `CONFIG` tags)
	- `Mage::app()->cleanCache();`
- Flush Cache Storage
	- This clears all cache but this might affect other applications using this cache storage (all tags)
	- `Mage::app()->getCacheInstance()->flush();`


### Adminhtml Blocks

A standard form within the admin area uses `Mage_Adminhtml_Block_Widget_Form` and template `app/design/adminhtml/default/default/template/widget/form.phtml`.

A form container's role (`app/design/adminhtml/default/default/template/widget/form/container.html`) is to act as a wrapper around the form.  There are several things on an adminhtml form page which are not controller by the form itself, e.g. save button URL, page heading and back button functionality.

Form element objects represent the different types of form fields you can have and contain their specific logic. The full list is in `Varien_Data_Form_Element`.

Form fields are added to fieldsets, which just represent a related collection of fields.  Fieldsets are, in turn, added to forms. 

Here's an example of adding a field to a fieldset.

```php
<?php
	$fieldset->addField('module[enabled]', 'select', array(
	    'label' => $this->_helper->__('Enable?'),
	    'title' => $this->_helper->__('Enable?'),
	    'name' => 'module[enabled]',
	    'value' => ($this->_input_data) ? $this->_input_data->getData("enabled") : $this->_config_model->getEnabled(),
	    'values' => Mage::getSingleton('adminhtml/system_config_source_yesno')->toOptionArray()
	));
?>
```
The second argument of `addField` maps to an element type, e.g. select and `Varien_Data_Form_Element`, by way of:

```php
<?php $className = 'Varien_Data_Form_Element_' . ucfirst(strtolower($type)); ?>
```

### Custom Elements

To customise the way an element is rendered you could call `$element->setRenderer()` on it and specify a custom class to render it.

Alternatively, you could add a custom field type, which extends the element and then overwrites the `getHtml()`, `getDefaultHtml()` or `getElementHtml()`.

```php
<?php $fieldset->addType('colour_picker', '{namespace}_{module_name}_Varien_data_Form_Element_ColourPicker'); ?>
```

## Grids

Grids print out collection data in a table, with the ability to filter and sort.  

Grids typically extend the `Mage_Adminhtml_Block_Widget_Grid` class and by default use the `app/design/adminhtml/default/default/template/widget/grid.phtml` template.

### Filters

When a column is added to a grid, a filter can be added.  For example, for numbers the `adminhtml/widget_grid_column_filter_range` can be used.

To then perform operations such as filtering, sorting and paging, a collections exists on the grid via `setCollection`.  For example, when filtering, it uses the `addFieldToFilter()` method to define parameters with which to filter the collection.

Grid columns usually extend `Mage_Adminhtml_Block_widget_Grid_Column` which contains data about the source of the columns data and its presentation (data type, rendering, styling).

Column renderers control the logic for drawing the column on the page.  For instance, the select column renderer has logic for drawing the heading of the column in a select box. 

### Grid JavaScript

The JavaScript used by the grid can be customised by setting values on the grid, e.g. `setJsObjectName()` or rewritten by changing the grid template.

```php
<script type="text/javascript">
//<![CDATA[
    <?php echo $this->getJsObjectName() ?> = new varienGrid('<?php echo $this->getId() ?>', '<?php echo $this->getGridUrl() ?>', '<?php echo $this->getVarNamePage() ?>', '<?php echo $this->getVarNameSort() ?>', '<?php echo $this->getVarNameDir() ?>', '<?php echo $this->getVarNameFilter() ?>');
    <?php echo $this->getJsObjectName() ?>.useAjax = '<?php echo $this->getUseAjax() ?>';
    <?php if($this->getRowClickCallback()): ?>
        <?php echo $this->getJsObjectName() ?>.rowClickCallback = <?php echo $this->getRowClickCallback() ?>;
    <?php endif; ?>
    <?php if($this->getCheckboxCheckCallback()): ?>
        <?php echo $this->getJsObjectName() ?>.checkboxCheckCallback = <?php echo $this->getCheckboxCheckCallback() ?>;
    <?php endif; ?>
    <?php if($this->getRowInitCallback()): ?>
        <?php echo $this->getJsObjectName() ?>.initRowCallback = <?php echo $this->getRowInitCallback() ?>;
        <?php echo $this->getJsObjectName() ?>.initGridRows();
    <?php endif; ?>
    <?php if($this->getMassactionBlock()->isAvailable()): ?>
    <?php echo $this->getMassactionBlock()->getJavaScript() ?>
    <?php endif ?>
    <?php echo $this->getAdditionalJavaScript(); ?>
//]]>
</script>
```

### Grid Containers

Grid containers work in the same way as form containers work, by integrating the grid into the admin page (setting headings, buttons, surrounding HTML).

Mass actions allow performing an action to selected grid rows.  They work by calling a controller action with all the selected row IDs in a parameter as an array.


## System Configuration

The system configuration is populated through the use of `system.xml` files.  The configuration is structured into: 

- Sections
    - Appear as left menu items, further categories into "tabs" separately
- Groups
    - Fieldsets of configuration options with all groups of a section appearing on the same page.
- Fields
    - Individual options

Fields have:

- `frontend_type` and `frontend_model`
    - Defines how a field is rendered and what type of data it stores.  To render a custom template `Varien_Data_Form_Element_Renderer_Interface` class should be extended and implement the `render()` method.
- `backend_model`
    - Defines how a field is represented in the backend
- `source_model`
    Defines field options, e.g. for a select.  Options are provided by way of `topOptionArray()` method.
- `frontend_class`
    - Can be used to customise the CSS class of the field.

The configuration is parsed by `Mage_Adminhtml_Model_Config` and rendered in the admin area by the `Mage_Adminhtml_Block_System_Config_Form` block.

When the configuration is saved, it is stored in the `core_config_data` table.  The config can have multiple values (rows), one for each scope (e.g. default, website, store view)
 
To retrieve configuration values `Mage::getStoreConfig()` can be used.  It prepends `default/stores/default` to `$config->getNode()` and so therefore is a wrapper around `Mage::getConfig()->getNode()`.


## Access Control Lists (ACL)

The ACL manages user access to different areas of the administration area and is configured through `adminhtml.xml` files. The keywords here are:

- Resources
    - Individual sections of the system
- Roles
    - Collections of Resources accessible to a particular group of users
- Users
    - Individual users of the system, who get assigned Roles.  In turn, these roles allowing access to Resources.

To add a menu item:

```xml
<menu>
    <{top_menu}>
        <{sub_menu}>
            <{sub_sub_menu}>
                <title></title>
                <action></action>
            </{sub_sub_menu}>
        <{sub_menu}>
    <{top_menu}>
</menu>
```

A menu item has to have a corresponding resource and a user has to have access to that resource for the item to render.

To add a resource:

```xml
<acl>
    <{resource}>
        <children>
            <{child}>
                <title></title>
                <sort_order></sort_order>
                <children></children>
            </{child}>
        </children>
    </{resource>
</acl>
```

Administrator roles have access to special `<all>` resource which grants all permissions.  Resources can also be created for sections, e.g. system configuration checks that the users is able to access each configuration section individually.  

Configuration resources are added in:

```xml
<acl>
    <{admin}>
        <children>
            <system>
                <children>
                    <config>
                        <children>
                            <{section_name}>
                            </{section_name}>
                        </children>
                    </config>
                </children>
            </system>
        </children>
    </{admin}>
```

Permissions are checked by controllers, using the `_isAllowed()` method which gets called on admin `preDispatch()`.

`adminhtml.xml` gets parsed by `Mage_Admin_Model_Config`.  The ACL configuration is managed by `Mage_Admin_Model_Acl` and is checked in most controllers using the following:

```php 
<?php Mage::getSingleton('admin/session')->isAllowed(); ?>
```

ACL data is stored in the database in the `admin_user` (users), `admin_role` (role names), `admin_rule` (permissions) tables.

Magento ACL is an extension of Zend_ACL. 

## Extensions

Extensions are installed as Magento modules.  The standard module considerations apply:

- Definition in `app/etc/modules`
- `app/code/{pool}/{namespace}/{module}/etc/config.xml

Module dependencies are specified in the definition XML with a `<depends>` tag.

Downloader can be used to download extension packages from Magento Connect and install (extract) them.

As covered in [the basics](/basics.html), there are three code pools

1. Core
    - Main Magento modules (First Party)
2. Community
    - Release extensions (Third Party)
3. Local
    - Local customisations.

<ul class="navigation">
    <li class="prev"><a href="/eav.html">&larr; EAV</a>
    <li class="next"><a href="/catalog.html">Catalog &rarr;</a>
</ul>
