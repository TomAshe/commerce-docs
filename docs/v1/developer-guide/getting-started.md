---
title: Developer guide
taxonomy:
    category: docs
---

## Introduction

Development on Drupal Commerce is a community effort, and all are invited to join in where possible.  We're bringing together people from around the world with a wide range of expertise to make this a giant leap forward in Drupal based e-commerce.

In order to dive in, you should know where the code currently stands and how to contribute to it.  To that end, we'll be adding a project roadmap in the near future that ties into both a specification and issues in our <a href="https://drupal.org/project/issues/commerce">issue tracker</a> to visualize what we have completed and what remains to be done.

When you're ready to dive in, you should review the [development standards](#development-standards) and make sure you understand the Drupal Commerce [code workflow](#code-workflow). Then go pick an issue, find other people working on the same system, and get to work!

## Development standards

The Drupal Commerce project is striving for strict adherence to the following development standards to produce the best quality code we can with a high level of maintainability. Any patch that is posted for review will be subject to these standards, which largely follow widely accepted Drupal development standards. Some of these items are simply best practices that will help us have a unified code style throughout the project.

If a patch needs work based on any of the project's development standards, please indicate the number below so the patch author can adapt his or her code style.


<ol>
<li><a href="#syntax">Code Syntax and Documentation</a></li>
<li><a href="#structure">Module File and Directory Structures</a></li>
<li><a href="#packages">Module Packages</a></li>
<li><a href="#naming">Function and Hook Naming</a></li>
<li><a href="#simpletest">Automated Testing</a></li>
<li><a href="#apis">Using Core and Contributed Module APIs</a></li>
<li><a href="#permissions">Drupal Permissions and Access Control</a></li>
<li><a href="#l10n">Localization and Variable Strings</a></li>
<li><a href="#theme">Template Files and Theme Functions</a></li>
<li><a href="#ui">Separating API from UI</a></li>
<li><a href="#performance">Performance Considerations</a></li>
<li><a href="#data">Data Storage</a></li>
</ol>

<a id="syntax"> </a>
<h3>1. Code Syntax and Documentation</h3>

We'll follow Drupal's <a href="https://drupal.org/coding-standards">coding standards</a> for code syntax and documentation. All functions should be well-documented with both doxygen comment blocks and inline comments. Patches should pass the code review provided by the <a href="https://drupal.org/project/coder">Coder</a> module.

<a id="structure"> </a>
<h3>2. Module File and Directory Structures</h3>

Modules should pattern their files and directories after CCK:

<code>
example.info
example.module
README.txt
/help
/includes
/includes/example.pages.inc
/includes/example.admin.inc
/includes/views
/includes/views/handlers
/tests
/theme
/theme/theme.inc
/theme/example.css
/theme/example-foo.tpl.php
</code>

The help directory contains files compatible with the <a href="https://drupal.org/project/advanced_help">Advanced Help</a> module. The includes directory contains files referenced in menu items and other necessary module includes. The theme directory contains a theme.inc file for theme functions and preprocess functions in addition to any CSS or module based .tpl.php files.  Theme / preprocess functions referenced by functions in *.admin.inc and *.pages.inc files can reside in the files themselves.

The tests directory contains <a href="https://drupal.org/simpletest">SimpleTest</a> include files.

<a id="packages"> </a>
<h3>3. Module Packages</h3>

Module .info files should use a strict set of package names:

<ul>
<li>Commerce - used for all modules in the core Commerce project</li>
<li>Commerce (contrib) - used for modules not in the core Commerce project regardless of who develops them</li>
<li>Commerce (_____) - projects with dependencies between the various modules in the package may consider their own package name using the same format but should otherwise default to Commerce (contrib)</li>
</ul>

As an example exception, Commerce PayPal uses the Commerce (PayPal) package name, because each PayPal payment method is represented by a separate module that depends on a core PayPal module for IPN processing.  It is helpful to the user to group these 4+ modules together with their dependency.

Contributed modules that don't conform to this standard should not be included in Features / distributions until they do. Issues should be posted on drupal.org to projects that do not conform.

<a id="naming"> </a>
<h3>4. Function and Hook Naming</h3>

Use the [module]_(object)_verb pattern for all function names:

<code>
commerce_product_load()
commerce_product_type_load()
</code>

(This list of examples needs to be fleshed out further.)

Hooks should include a soft namespace by using the commerce_ prefix to avoid conflicts with other contributed modules. For example, Ubercart's hook_order() would become hook_commerce_order() under this convention.

Only form builder functions should end in _form.

<a id="simpletest"> </a>
<h3>5. Automated Testing</h3>

We will be using Drupal 7's core <a href="https://drupal.org/simpletest">SimpleTest</a> testing framework to facilitate code development and maintainability.  While not embracing true test-driven development, we will still only be committing code that comes with appropriate function tests.

The Commerce module provides a base class used for creating Drupal Commerce tests.  The process is documented in the Specification handbook, <a href="https://drupalcommerce.org/developer-guide/utilizing-core-apis/writing-simpletests-commerce-modules">Writing SimpleTests for Commerce modules</a>.

Unit testing is under consideration for the core APIs, but a realistic target for now is to mimic Drupal's core test coverage guidelines.

<a id="apis"> </a>
<h3>6. Using Core and Contributed Module APIs</h3>

Drupal Commerce modules should follow Drupal coding best practices when implementing core APIs, including (but by no means limited to):

<ul>
<li>Using the Form API appropriately</li>
<li>Taking advantage of Drupal's alter hooks</li>
<li>Exposing their own objects for alteration via <a href="https://api.drupal.org/api/function/drupal_alter/7">drupal_alter()</a></li>
<li>Maintaining a strict separation between API functions and the user interface (i.e. not altering data directly in form submit handlers)</li>
<li>Integrating with the core Entity and Field systems</li>
</ul>

Our general rule is to integrate with major, well-maintained third party modules where possible. For example, Drupal Commerce modules should depend on Views instead of creating their own table listings. Ctools and Panels integration are under consideration, pending availability on Drupal 7.

<a id="permissions"> </a>
<h3>7. Drupal Permissions and Access Control</h3>

Our problem in the past has been too few permissions, not too many. In general, we're adopting an approach that favors finer-grained permissions over having too few. Every permission should have a description.

This same goal holds true for access control as it pertains to products, orders, customer profiles, etc.  While the core modules don't need to anticipate every possible implementation of the Commerce modules, these access systems and others like them should be extensible to allow for finer-grained access.  The goal here is to eliminate the need for core patches to allow use cases like peer-to-peer sales.

<a id="l10n"> </a>
<h3>8. Localization and Variable Strings</h3>

In Ubercart we provided numerous configuration options to override interface strings, like the text used on the Add to Cart button.  This presented a problem for multilingual sites, as Drupal's localization system can translate string literals but not variables. (For more information, see documentation on the <a href="https://api.drupal.org/api/function/t/7">t()</a> function.)

To correct this problem, the ability to override strings via custom settings forms will not be provided by the core modules. Instead, we will instruct administrators to use the <a href="https://drupal.org/project/stringoverrides">String Overrides</a> module (or whatever comparable systems exist).

Furthermore, modules should provide help 'context' to functions used in localization to facilitate accurate translation.

<a id="theme"> </a>
<h3>9. Template Files and Theme Functions</h3>

Module files and includes should not contain style tags or attributes in any HTML output. Instead, use appropriate IDs and classes with coordinating CSS in the module's .css file(s).

Template files with preprocess functions should be used for nearly all HTML output. In cases where logic cannot be easily abstracted into variables in preprocess functions or when the amount of HTML is very minor, a theme function can be used instead. Use Drupal's core theme functions whenever possible.

<a id="ui"> </a>
<h3>10. Separating API from UI</h3>

As much as possible, we want to enforce at the core level a strict separation of API components from UI components. At one level, this looks like including both a dc_product and dc_product_ui module, like the core Drupal field and field_ui modules. Since a major goal of the project is to develop a solid core and leave the door open for installation profiles and <a href="https://drupal.org/project/features">Feature</a> modules to take care of packaging components, we don't want to make it difficult for developers and site builders to replace default UI components.

To be honest, we could probably use a separate set of user interface guidelines to ensure we have a consistent feel across all the Drupal Commerce modules.

<a id="performance"> </a>
<h3>11. Performance Considerations</h3>

Code should take advantage of static caching and *_load_multiple() API functions to reduce the amount of queries per page request. The Code module can also be used to audit queries for execution speed. Code should also take advantage of Drupal's file handling abilities in various systems (like the #file property on menu items) and class autoloading capability to minimize the amount of code loaded on any given pageload.

A topic for discussion would be a custom variable storage table or system to reduce the amount of variables stored at the global level.  At the very least, modules should consider storing data in custom tables <em>before</em> storing complex module settings in Drupal's variables system.

<a id="data"> </a>
<h3>12. Data Storage</h3>

If there's anything we've learned from our past, it's that we want to leave serialized arrays of data in the database behind. We're striving for better data models, and this section should be fleshed out as the project matures.

This heading could be further expanded to address all of our underlying data models and how to best incorporate classes in our modules.

## Code Workflow

Drupal Commerce code begins at the level of discussion.  Before digging in to do serious module development, ideas are kicked around in the *#commerce* [Drupal Slack channel] for due consideration.  When the objectives are reasonably clear, a functional specification is produced to guide development, along with any necessary notes on how to implement the features or craft the APIs.  These specs are themselves turned into issues in the <a href="https://drupal.org/project/issues/commerce">issue tracker</a> where development can be monitored.

When a developer is ready to get to work, he or she should chime in on the appropriate issue on d.o and get up to speed with current development. 

Bugs should always be reported via the issue tracker on d.o, but feature requests can start here in the forums so they can be worked into the specification.  The specification here should always be an up to date representation of what we have in code.


## Using Git

This page does not necessarily need to contain a tutorial on how to use Git, as those have been done, but we should link to appropriate resources and provide at least a listing of commands you can use to fork your GitHub repository, push commits back to it, and merge changes from other forks.

For starters, here's the prototype documentation on drupal.org for using Git to maintain a Drupal module:


<ul>
<li><a href="https://drupal.org/node/711070">https://drupal.org/node/711070</a></li>
<li><a href="https://progit.org/book/">https://progit.org/book/</a></li>
</ul>

## Installation profiles

A major goal of the project has been to enable site builders and developers to create installation profiles that pre-configure Drupal Commerce for a variety of use cases.  We would love to see a whole collection of Drupal Commerce distributions available to help new users get started.  This is especially important given the complexity of the project and the decision to remove as much non-essential code from core as possible.

This section of the handbook will be devoted to resources to assist you in creating these profiles.  It will also serve as a directory of available installation profiles, highlighting those hosted on drupal.org and elsewhere that are packaged into full-blown distributions.

For now, the following list of resources must suffice:

<ul>
<li><a href="https://drupal.org/project/commerce_kickstart">Commerce Kickstart</a> - an installation profile that performs default installation and configuration of the Commerce modules so a site builder can get right to work.</li>
<li><a href="https://github.com/rszrama/commercedev">Commerce Dev</a> - a developer oriented installation profile that enables modules and prepares some dummy products and displays for quick testing</li>
</ul>

[Drupal Slack channel]: https://www.drupal.org/slack
