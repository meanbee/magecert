---
layout: default
title: Rendering
chapter: 3
meta-description: Understand how pages are rendered through the use of themes, layouts, blocks and templates
---

# Rendering

Exam proportion: 7%.

## Customising Core Functionality with Themes

Themes have layout files which, amongst other things, can be used to change which blocks appear on the page.  The block template can also be changed and different methods called.

## Store-level designs

Magento's hierarchical structured themes means that a base theme can be extended and assigned on a store level.  

## Registering Custom Themes

Themes can be configured in three ways:

1. Per store under `System > Configuration > Design`.
2. Design change with time limits `System > Design`.
3. Theme exceptions can also be set on a category and product level.

## Package versus Theme

A package has multiple themes. Each of the themes in a package inherits from the `default` theme within a package.

## Design Fallback

The theme fallback procedure for locating templates files is:

1. `{package}/{theme}`
2. `{package}/default`
3. `base/default`

To add further directories to the theme fallback mechanism `Mage_Core_Model_Design_Package::getFilename` method need to be rewritten

For the admin area the fallback is `default/default`.

## Template and Layout Paths

Theme file paths are rendered by `Mage_Core_Model_Design_Package`.  `Mage_Core_Model_Layout_Update` requests absolute paths for layout files.  `Mage_Core_Block_Template` requests templates with relative paths.  

Magento uses relative paths when it comes to template and layout files.

## Blocks


Blocks are used for output.  The `root` block is the parent of all blocks and is of type `Mage_Page_Block_Html`.

`Mage_Core_Block_Template` blocks use template files to render content.  The template file name are set within `setTemplate()` or `addData('template')` with relative paths.

Templates are just pieces of PHP included in `Mage_Core_Block_Template`.  Therefore `$this` in a template refers to the block.

`Mage_Core_Block_Template` uses a buffer before including a template to prevent premature output.

The `Mage_Core_Model_Layout::createBlock` method creates instances of blocks.

The `Mage_Core_Model_Layout_Update` class considers which blocks need to be created for each page by looking at the layout handles.

All output blocks are rendered, e.g. by calling `toHtml()`, which in turn can choose to render their children.

`Text` and `Text_List` blocks automatically render their content.

There are two events that are fired around block rendering that can be used to modify the block before and after rendering the HTML:

1. `core_block_abstract_to_html_before`
2. `core_block_abstract_to_html_after`


A child block will only be rendered automatically if it is of class `Mage_Core_Block_Textlist` otherwise the `getChildHtml` method needs to be called.


Block instances can be accessed through the layout, e.g. `Mage::app()->getLayout()` and `$controller->getLayout()`.  Block output is controlled by the `_toHtml()` function.  

Templates are rendered by the `renderView()`/`fetchView()` methods inside a template block.  Output buffering can be disabled with `$layout->setDirectOutput`.

It is possible to add a block to the current layout but it needs to be done before the `renderLayout()` method is called.

## Layout XML

- `<reference>`
	- edit a block
- `<block>`
	- define a block
- `<action>`
	- call method on a block
- `<update>`
	- include nodes from another handle.

Layout files can be registered in `config.xml`:

```xml
<config>
	<{area}>
		<layout>
			<updates>
				<{name}>
					<file>{filepath}</file>
				</{name}>
			</updates>
		</layout>
	</{area}>
</config>
```

Page output can be customised in the following ways:

- Template changes
- Layout changes
- Overriding blocks
- Observers

Variables on blocks can be set in the following ways:

- Layout
	- Through actions or attributes
- Controller
	- `$this-getLayout()->getBlock()`
- Child blocks
	- `$this->getChild()`
- Other
	- `Mage::app()->getLayout()`

## Head Block Assets

JavaScript and CSS assets are handled in the `Mage_Page_Block_Html_head` block.  This block handles the merging of assets into a single file to minimise HTTP requests.  The merged file is based on the edit time of the source files.

When merging CSS, a callback function on `Mage_Core_Model_Design_Package` is called to update any `@import` or `url()` directives with the correct URLs.

<ul class="navigation">
    <li class="prev"><a href="/request-flow.html">&larr; Request Flow</a>
    <li class="next"><a href="/databases.html">Databases &rarr;</a>
</ul>
