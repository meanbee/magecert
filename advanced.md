---
layout: default
title: Advanced
meta-description: Extra notes about API and Widgets
---

# Advanced Features

Exam proportion: 13%.

## Widgets

Widgets are customisable frontend blocks that can be included and configured in CMS blocks or pages.  The widget architecture is primarily driven by the `Mage_Widget_Model_Widget` model while the frotnend blocks used by widgets implement the `Mage_Widget_Block_Interface` interface.

Widgets are defined in a modules `widget.xml` configuration file using the following syntax:

{% highlight xml %}
<widgets>
    <{widget_identifier} type="{grouped_class_name}" translate="{fields}" module="{module_name}">
        <name>{widget_name}</name>
        <description>{widget_description}</description>
        <parameters>
            <{parameter_identifier} type="{type}" translate="{fields}">
                <visible>{0|1}</visible>
                <required>{0|1}</required>
                <label>{parameter_label}</label>
                <type>{parameter type}</type>
                <value>{value}</value>
                <values>{possible_values}</values>
                <helper_block>{helper_block_definition}</helper_block>
                <sort_order>{order}</sort_order>
            </{parameter_identifier}>
            ...
        </parameters>
    </{widget_identifier}>
</widgets>
{% endhighlight %}

Depending on the type, widget parameters can have a single value, a selection from multiple values or can include complex procedures e.g selecting a gallery image, to determine their value, driven by a helper block.

## API

Magento includes built0in SOAP, XML-RPC and REST APIs for managing the store remotely.

The SOAP API is default and the most widely used.  The WSDL for it is located in:

- `{base_url}/api/?wsdl`
- `{base_url}/api/v2.soap?wsdl`

The API resources are defined in `api.xml` or `api2.xml` configuration files.  In these files a model is specified for each resource to handle the calls. API methods are published by defining them in the `wsdl.xml` or `wsdl2.xml` configuration files.  The methods map to the methods on resource models.  API resource include ACL, so only certain users can call some API methods.

The `wsdl.xml` configuration files are loaded using `Mage_Core_Model_Config` model.  This is also responsible for Magento module configuration loading, so it works exactly like loading `config.xml` does when it comes to extending and overriding API method configuration.

The API methods are defined by API models, which are instantiated using Magento's class factory methods.  Therefore, they can be overwritten like regular Magento models to override the API methods. Additional methods can be added to overwritten classes or by adding new API models to `wsdl.xml`.


### API versus API2

The difference between the two SOAP API version is that in version 1 all operations are executed by passing them to the `call()` and `multicall()` methods, e.g.

{% highlight php %}
<?php $client->call($session, "sales_order.list"); ?>
{% endhighlight %}

Meanwhile in version 2, every operation has its own method defined, e.g.

{% highlight php %}
<?php $client->salesOrderList(); ?>
{% endhighlight %}

Internally, the version 2 API handler uses the same API method configuration as version 1.  When a version 2 API method is called e.g. `catalogProductCreate()` the handler searches for a prefix that matches the method in

{% highlight xml %}
<v2>
	<resources_function_prefix>
    </resources_function_prefix>
</v2>
{% endhighlight %}

section of the config, e.g. `catalogProduct` to find the method resource and then calls the `{resource}.{method_suffix} API version 1 method.

<ul class="navigation">
    <li class="prev"><a href="/sales.html">&larr; Sales</a>
    <li class="next"><a href="/">Home &rarr;</a>
</ul>








