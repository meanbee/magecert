---
layout: default
---

# Adminhtml

Exam proportion: 6%.

## Architecture

### Configuration

Configuration areas that are loaded for the admin area are

- `<menu>`
- `<acl>`
- `<admin>`
- `<adminhtml>`

### Adminhtml Controller

The adminhtml controller extends `Mage_Adminhtml_Controller_Action`.  Whereas the frontend controllers extend `Mage_Core_Controller_Front_Action`.

The admin controller adds some authentication on the sessions since the admin area must not be accessible by all. 

### Routing

In `Mage_Core_Controller_Varien_Front::dispatch()`, Magento loops over each of its routers and checks against the path specified by the request to see if any modules have registered against this frontname.  From there, the module spcified in its controller (defined by `config.xml`) which controller class it extends (the admin or the front one) and in each of these the area is defined.

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

In `Mage_Admin_Model_Observer` there is an observer for the `predispatch` event of an admin controller action.  In here, there is logic to see if there is a user attached to the session.  If there is refreshed the ACL and returns, otherwise it checks a list of allowed actions.  If what the user has requested is outside this then it redirects to the login page.

### Adminhtml Blocks

Adminhtml blocks extend `Mage_Adminhtml_Block_Template`.

### Adminhtml Config

The role of the adminhtml configuration is to provide a mechanism within the admin area for users to be able to set configuration options for stores.

### Cache Management

- Flush Magento Cache
	- This removes only the cache entries that are managed by Magento itself.
	- `Mage::app()->cleanCache();`
- Flush Cache Storage
	- This clears all cache but this might affect other applications using this cache storage.
	- `Mage::app()->getCacheInstance()->flush();`

### Adminhtml Blocks

A standard form within the admin area uses `Mage_Adminhtml_Block_Widget_Form` and template `app/design/adminhtml/default/default/template/widget/form.phtml`.

A form container's role (`app/design/adminhtml/default/default/template/widget/form/container.html`) is to act as a wrapper around the form.  There are several things on an adminhtml form page which are not controller by the form itself, e.g. save button URL, page heading and back button functionality.

As part of defining a form, elements need to be added to it.  Each of these are represented by a class in the backend.

{% highlight php %}
<?php
	$fieldset->addField('module[enabled]', 'select', array(
	    'label' => $this->_helper->__('Enable?'),
	    'title' => $this->_helper->__('Enable?'),
	    'name' => 'module[enabled]',
	    'value' => ($this->_input_data) ? $this->_input_data->getData("enabled") : $this->_config_model->getEnabled(),
	    'values' => Mage::getSingleton('adminhtml/system_config_source_yesno')->toOptionArray()
	));
?>
{% endhighlight %}

The second argument of `addField` maps to an element type, e.g. select and `Varien_Data_Form_Element`, by way of:

{% highlight php %}
<?php $className = 'Varien_Data_Form_Element_' . ucfirst(strtolower($type)); ?>
{% endhighlight %}

Available elements are:

- `Varien_Data_Form_Element_Button`
- `Varien_Data_Form_Element_Checkbox`
- `Varien_Data_Form_Element_Checkboxes`
- `Varien_Data_Form_Element_Collection`
- `Varien_Data_Form_Element_Column`
- `Varien_Data_Form_Element_Date`
- `Varien_Data_Form_Element_Editor`
- `Varien_Data_Form_Element_Fieldset`
- `Varien_Data_Form_Element_File`
- `Varien_Data_Form_Element_Gallery`
- `Varien_Data_Form_Element_Hidden`
- `Varien_Data_Form_Element_Image`
- `Varien_Data_Form_Element_Imagefile`
- `Varien_Data_Form_Element_Label`
- `Varien_Data_Form_Element_Link`
- `Varien_Data_Form_Element_Multiline`
- `Varien_Data_Form_Element_Multiselect`
- `Varien_Data_Form_Element_Note`
- `Varien_Data_Form_Element_Obscure`
- `Varien_Data_Form_Element_Password`
- `Varien_Data_Form_Element_Radio`
- `Varien_Data_Form_Element_Radios`
- `Varien_Data_Form_Element_Reset`
- `Varien_Data_Form_Element_Select`
- `Varien_Data_Form_Element_Submit`
- `Varien_Data_Form_Element_Text`
- `Varien_Data_Form_Element_Textarea`
- `Varien_Data_Form_Element_Time`


### Fieldsets

Fields are added to a fieldset which are in turn added to the form.

### Custom Element Templates

To add a custom element to a template a new type to the fieldset:

{% highlight php %}
<?php $fieldset->addType('colour_picker', '{namespace}_{module_name}_Varien_data_Form_Element_ColourPicker'); ?>
{% endhighlight %}

Within this custom form element class the `getElementHtml()` method can then define the custom HTML that is required.


## Grids

Grids typically extend the `Mage_Adminhtml_Block_Widget_Grid` class and by default use the `app/design/adminhtml/default/default/template/widget/grid.phtml` template.

### Filters

When a column is added to a grid a filter can optionally be added.  For example, for numbers the `adminhtml/widget_grid_column_filter_range` can be used. 

To then perform operations such as filtering, sorting and paging, a collections exists on the grid via `setCollection`.  For example, when filtering, it uses the `addFieldToFilter()` method to define parameters with which to filter the collection.


### Column Renderers

Column renderers control the logic for drawing the column on the page.  For instance, the select column renderer has logic for drawing the heading of the column in a select box. 

### Grid Javascript

Values can be set on the grid block to determine what JavaScript is used.

{% highlight php %}
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
{% endhighlight %}


## System Configuration

The system configuration is populated through the use of `system.xml` files.  The structure of rendered elements is defined by the groups, labels, types, orders.  When the configuration is saved, it is stored in the `core_config_data` table.

To use configuration elements such as selects and multiselects, a `<source_model>` needs to be defined with an implemented `toOptionArray()` method.

Rending a custom template in system configuration can be done by specifying a `frontend_model` in the `system.xml` file.  The custom frontend model class should extend `Varien_Data_Form_Element_Rendere_Interface` and implement the `render()` method.

The CSS class of an element can be changed using `<frontend_class>`.

When retrieving configuration values `Mage::getStoreConfig()` prepends `default/stores/default` to `$config->getNode()` and so therefore is a wrapper around `Mage::getConfig()->getNode()`.

### Under-the-hood

`Mage_Adminhtml_Model_Config` parses the configuration and then `Mage_Adminhtml_Block_System_Config_Form` renders the config form.

<ul class="navigation">
    <li class="prev"><a href="/eav.html">&larr; EAV</a>
    <li class="next"><a href="/catalog.html">Catalog &rarr;</a>
</ul>






















