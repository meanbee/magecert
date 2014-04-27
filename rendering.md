---
layout: default
---

# Rendering

## Customising Core Functionality with Themes

Themes have layout files which, amongst other things, can be used to change which blocks appear on the page.  The block template can also be changed and different methods called. 

## Store-level designs

Magento's hierarchical structured themes means that a base theme can be extended and assigned on a store level.  

## Registering Custom Themes

Custom themes can be configured under `System > Design` and `System > Configuration > Design`.

## Package versus Theme

A package has multiple themes. Each of the themes in a package inherits from the default theme within a package.

## Missing files in Theme/Package

If a file is missing from a theme, it falls back and checks the default theme in a package.  If the file is also missing from this location then the `base/default` theme is checked. 

## Template Paths

Magento uses relative paths when ti comes to template and layout files.

## Defining physical files corresponding to templates and layouts.

## Additional Theme Fallbacks

To add further directories to the theme fallback mechanism `Mage_Core_Model_Design_Package::getFilename` method need to be rewritten

## Block Instance Creation

The `Mage_Core_Model_Layout::createBlock` method creates instances of blocks

## Page Blocks

The `Mage_Core_Model_Layout_Update` class considers which blocks need to be created for each page by looking at the layout handles.

## Rending the Block Tree

The tree of blocks is rendered by calling `toHtml()` on the root and cascading down.

## Create Blocks without Layout

In addition to the layout files, blocks can be created in the template files.

## Adding Block to Current Layout

It is possible to add a block to the current layout but it needs to be done before the `renderLayout()` method is called.

## Children Block Rendering

A child block will only be rendered automatically if it is of class `Mage_Core_Block_Textlist` otherwise the `getChildHtml` method needs to be called.


<ul class="navigation">
    <li class="prev"><a href="/request-flow.html">&larr; Request Flow</a>
    <li class="next"><a href="/databases.html">Databases &rarr;</a>
</ul>
