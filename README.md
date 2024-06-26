PHPDoc support for Doxygen
==========================

This package provides [PHPDoc](https://en.wikipedia.org/wiki/PHPDoc) support for
[Doxygen](http://doxygen.nl/). It contains:

- *doxygen-phpdoc-filter.php:* An input filter for Doxygen, required to parse
  PHPDoc comments
- *Doxyfile.sample:* A sample configuration file fragment for Doxygen with the
  required options
- *doxygen-phpdoc-fixhtml.php:* A postprocessor for the generated HTML output

It works best with plain PHPDoc comments. The filter may get confused when
Doxygen elements get mixed in.


Installation
============

The package can be installed standalone or as a composer dependency for a
project.

Standalone package
------------------

Download the package and install is dependencies via
[composer](https://getcomposer.org):

    cd doxygen-phpdoc
    composer install

As a composer dependency
------------------------

To include the package in another project, add it as a dependency:

    cd MyProject
    composer require --dev hschletz/doxygen-phpdoc


Usage
=====

The package provides an input filter which must be activated in the project's
Doxygen config file. Also activate the JAVADOC_AUTOBRIEF option:

    FILTER_PATTERNS        = *.php=vendor/bin/doxygen-phpdoc-filter.php
    JAVADOC_AUTOBRIEF      = YES

Adjust file extensions and script path as necessary. After setting
project-specific options in the config file, Doxygen should be able to compile
the documentation.

The output is already usable, but with some limitations. The postprocessor can
fix some of the issues for the HTML output. Invoke it with the output directory:

    vendor/bin/doxygen-phpdoc-fixhtml.php doc/api/html


The input filter
================

Doxygen invokes the input filter, passing the source file name, and processes
its output instead of the original source. Any modifications by the input filter
are temporary and the original source is not touched.

The filter treats every occurrence of a single backslash within a documentation
block as a namespace separator and replaces it with `::`, which is required for
correct Doxygen operation. For this reason, only PHPDoc's "@"-Syntax will work
for any tags. Multiple consecutive backslashes are left untouched, and Doxygen
will interpret them as escaped backslashes.

Some PHPDoc tags, like `@var`, `@property`, `@return` or `@link`, have a
different meaning or syntax in Doxygen. These commands are rewritten in a way
that is correctly handled. In some cases, the behavior can only by emulated to
some degree. For example, there is no full equivalent for `@internal`, which
gets replaced by `@private`. The same goes for some commands which don't exist
in Doxygen, like `@property-read` (put description in a `@remark`) or `@license`
(replace with `@copyright`).

Inline tags, like `{@inheritdoc}`, are unknown to Doxygen. The curly braces are
removed to prevent them from appearing in the output.

The file header gets properly marked as such. Otherwise it would be interpreted
as documentation for the namespace, which is uncommon in PHPDoc.

Doxygen does not understand typed arrays, like `string[]` as datatypes for
parameters and return values. These get replaced by `array` and the full
typehint is added to the description.


The HTML postprocessor
======================

The postprocessor applies some fixes to the HTML output. Namespace separators
which are partially output as "::" get converted to a backslash where possible.
Some occurrences may remain because not all of them can be reliably detected.

When linking to the PHP documentation for builtin classes with the help of a
tagfile, Doxygen makes the links unusable by appending a .html suffix. These
links get fixed.


Known issues
============

- Not all PHPDoc tags are supported yet.
- The input filter only works properly on files that contain a namespace.
  Support for code without namespace may be added in the future.
- Doxygen will issue some warnings even for otherwise correct code. These
  warnings could be filtered out by a wrapper script.

warning: Detected potential recursive class relation
----------------------------------------------------
This message occurs when a class inherits (directly or indirectly) from a class
with the same name from a different namespace. This is a bug or limitation in
Doxygen.

warning: Internal inconsistency: scope for class ... not found!
---------------------------------------------------------------
This message is caused by some external documentation from a tagfile for unknown
reasons.

warning: Illegal command ... as part of a &lt;dt&gt; tag
--------------------------------------------------------
This is a known Doxygen bug which occurs with methods marked deprecated having a
namespaced class typehint for a parameter. The typehint will not show on the
"Deprecated List" page. The documentation is otherwise correct.

WARNING: could not parse ...
----------------------------
This message is generated by the postprocessor on invalid XHTML files. The
invalid markup can be caused by insufficient escaping or otherwise malformed
docblocks. Check the source files for invalid syntax.
