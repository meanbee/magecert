---
layout: default
noToc: true
---

# Study Guide Questions

## Basics

### High-Level Magento Architecture

- [Describe Magento codepools](/basics.html#code-pools)
- [Describe typical Magento module structure](/basics.html#module-structure)
- [Describe Magento templates and layout files location](/basics.html#template-layout)
- [Describe Magento skin and JavaScript files location](/basics.html#skin-javascript)
- [Identify and explain the main Magento design areas (adminhtml and frontend)](/basics.html#design-areas)
- [Explain class naming conventions and their relationship with the autoloader](/basics.html#naming-conventions)
- [Describe methods for resolving module conflicts.](/basics.html#conflict-resolution)

### Magento Configuration

- [Explain how Magento loads and manipulates configuration information](/basics.html#config-load)
- [Describe class group configuration and use in factory methods](/basics.html#factory-methods)
- [Describe the process and configuration of class overrides in Magento](/basics.html#overrides)
- [Register an Observer](/basics.html#observer)
- [Identify the function and proper use of automatically available events.](/basics.html#automatic-events)
- [Set up a cron job](/basics.html#cron-jobs)

### Internationalization

- [Describe how to plan for internationalization of a Magento site](/basics.html#internationalization)
- [Describe the use of Magento translate classes and translate files](/basics.html#internationalization)
- [Describe the advantages and disadvantages of using subdomains and subdirectories in internationalization](/basics.html#subdomains-directories)

## Request Flow

### Application initialization 

- Describe the steps for application initialization 
- Describe the role of the system entrypoint, index.php

### Front Controller 

- Describe the role of the front controller
- Identify uses for events fired in the front controller

### URL rewrites 

- Describe URL structure/processing in Magento 
- Describe the URL rewrite process

### Request routing 

- Describe request routing/request flow in Magento
- Describe how Magento determines which controller to use and how to customize route-to-controller resolution

### Module initialization 

- Describe the steps needed to create and register a new module
- Describe the effect of module dependencies
- Describe different types of configuration files and the priorities of their loading

### Design and layout initialization 

- Identify the steps in the request flow which: Design data is populated, Layout configuration files are parsed, Layout is compiled and Output is rendered

### Flushing data (output) 

- Describe how and when Magento renders content to the browser 
- Describe how and when Magento flushes output variables using the Front controller 

## Rendering

### Themes in Magento

- Define and describe the use of themes in Magento
- Define and describe the use of design packages
- Describe the process of defining template file paths

### Blocks

- Describe the programmatic structure of blocks
- Describe the relationship between templates and blocks
- Describe the stages in the life-cycle of a block
- Describe events fired in blocks
- Identify different types of blocks
- Describe block instantiation
- Explain different mechanisms for disabling block output
- Describe how a typical block is rendered

### Design layout, XML schema, and CMS content directives 

- Describe the elements of Magento's layout XML schema, including the major layout directives
- Register layout XML files
- Create and add code to pages
- Explain how variables can be passed to block instances via layout XML
- Describe various ways to add and customize JavaScript to specific request scopes

## Working with Databases in Magento

### Models, resource models, and collections 

- Describe the basic concepts of models, resource models, and collections, and the relationship they have to one another 

- Configure a database connection
- Describe how Magento works with database tables 
- Describe the load-and-save process for a regular entity
- Describe the load-and-save process for a regular entity
- Describe the role of Zend_Db_Select in Magento
- Describe the collection interface (filtering/sorting/grouping) 
- Describe the hierarchy of database-related classes in Magento 
- Describe the role and hierarchy of setup objects in Magento 

### Install/upgrade scripts 

- Describe the install/upgrade workflow
- Write install and upgrade scripts using set-up resources
- Identify how to use the DDL class in setup scripts

## Entity-Attribute-Value (EAV) Model 

### EAV model concepts 

- Define basic EAV concepts and class hierarchy 
- Describe the database schema for EAV entities
- Describe the EAV entity structure and its difference from the standard core resource model 
- Describe the EAV load-and-save process and its differences from the regular load-and-save process

### Attributes management 

- Identify the purpose of attribute frontend, source, and backend models 
- Describe how to implement the interface of attribute frontend, source, and backend models
- Describe how to create and customize attributes. 

## Adminhtml

### Common structure/architecture 

- Describe the similarities and differences between adminhtml and frontend interface and routing
- Describe the components and types of cache clearing using the adminhtml interface

### Forms in Magento 

- Define form structure, form templates, grids in Magento, and grid containers and elements

### Grids in Magento 

- Create a simple form and grid for a custom entity
- Describe how to implement advanced Adminhtml Grids and Forms, including editable cells, mass actions, totals, reports, custom filters and renderers, multiple grids on one page, combining grids with forms, and adding custom JavaScript to an admin form

### System configuration 

- Define the basic terms, elements, and structure of system configuration XML
- Describe system configuration scopes

### Access Control Lists (ACL) and permissions in Magento 

- Define/identify basic terms and elements of ACL
- Use ACL to: Set up a menu item, Create appropriate permissions for users, Check for permissions in permissions management tree structures
- Describe how to enable and configure extensions 
- Define Magento extensions and describe the different types of extension available (Community, Core, Commercial)

## Catalog

### Product Types 

- Identify and describe standard product types (simple, configurable, bundled, etc.). 
- Create custom product types from scratch or modify existing product types.
- Identify how custom product types interact with indexing, SQL, and underlying data structures. 

### Price Generation 

- Identify basic concepts of price generation in Magento
- Modify and adjust price generation for products (for example, during integration of third-party software)

### Category Structure 

- Describe the Category Hierarchy Tree Structure implementation (the internal structure inside the database)

### Catalog Price Rules 

- Identify how catalog price rules are implemented in Magento:

### Other Skills 

- Choose optimal catalog structure (EAV vs. Flat) for a given implementation
- Implement, troubleshoot, and modify Magento tax rules 
- Modify, extend, and troubleshoot the Magento layered (“filter”) navigation
- Troubleshoot and customize Magento indexes
- Describe custom product options in Magento

## Checkout

### Checkout components 

- Describe how to modify and effectively customize the quote object, the quote item object, and the address object
- Explain the database schema for total models

### Shopping Cart price rules 

- Describe how shopping cart price rules work and how they can be customized

### Shipping and payment methods in Magento 

- Describe the programmatic structure of shipping methods, how to customize existing methods, and how to implement new methods

- Describe the shipping rates calculation process:
- Describe the programmatic structure of payment methods and how to implement new methods

### Magento multishipping implementation 

- Describe how to extend the Magento multishipping implementation
- Identify limitations of the multishipping implementation

## Sales and Customers

### Sales

- Describe order creation in the admin
- Describe the differences in order creation between the frontend and the admin
- Card operations (capturing and authorization)
- Describe the order shipment structure and process
- Describe the architecture and processing of refunds
- Describe the implementation of the three partial order operations (partial invoice, partial shipping, and partial refund)
- Describe cancel operations

### Customer

- Describe the architecture of the customer module 
- Describe the role of customer addresses
- Describe how to add, modify, and display customer attributes

## Advanced features

### Widgets

- Create frontend widgets and describe widget architecture

### API

- Use the Magento API to implement third party integrations
- Extend the existing Magento API to allow for deeper integrations into third party products 
- Describe the different Web Service APIs available within the Magento Core
- Describe the advantages and disadvantages of the available Web Service APIs in Magento
- Identify the configuration files used for the v2 SOAP API
- Describe the purpose of the configuration files related to the API

### Other Skills

- Integrate Google features (Google Wallet, Checkout, AdWords, Analytics) into Magento implementation

## Enterprise Edition

- Describe how to customize, extend, and troubleshoot Enterprise Edition catalog target rules.
- Describe how to customize, extend, and troubleshoot the Enterprise Edition reward point system.
- Describe how to implement, customize, and troubleshoot Enterprise Edition website restrictions.
- Identify the elements and functioning of Enterprise Edition Full Page Cache.
- Describe the Payment Bridge
