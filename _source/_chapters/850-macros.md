---
title: Macro Reference
layout: default
---


A simple macro processor is added to the header processing. Variables allow a single definition of a value, and the use of derivations. Each header is a macro that can be expanded. Notice that headers that do not start with an upper case character will not be copied to the manifest, so they can be used as working variables. Variables are expanded by enclosing the name of the variable in `${<name>}` (curly braces) or `$(<name>)` (parenthesis). Additionally, square brackets \[\], angled brackets <>, double guillemets «», and single guillemets ‹› are also allowed for brackets. If brackets are nested, that is $[replace;acaca;a(.*)a;[$1]] will return `[cac]`.

There are also a number of macros that perform basic functions. All these functions have the following basic syntax:

     macro ::= '${' function '}' 
         | '$\[' function '\]'
         | '$(' function ')'
         | '$<' function '>'

     function ::= name ( ';' argument ) *

For example:

    version=1.23.87.200109111023542
    Bundle-Version= ${version}
    Bundle-Description= This bundle has version ${version}

## Arguments
@Since("2.3") Macros can contain arguments. These arguments are available in the expansion as ${0} to ${9}. ${0} is the name of the macro, so the actual arguments start at 1. The name is also available as ${@}. The arguments as an array (without the name of the macro) is available as ${#}. The more traditional * could not be used because it clashes with wildcard keys, it means ALL values. 

For example:

    foo: Hello ${1} -> ${foo;Peter} -> "Hello Peter"
    
## Wildcarded Keys
Keys can be wildcarded. For example, if you want to set -plugin from different places, then you can set the `plugin.xxx` properties in different places and combine them with `-plugins= ${plugins.*}`.


## Types
@TODO


## Reference

<div>
<dl class="property-index">

{% for macro in site.macros %}<dt><a href="{{macro.url}}">{{macro.title}}</a></dt><dd>{{macro.summary}}</dd>
{% endfor %}

</dl>
</div>

