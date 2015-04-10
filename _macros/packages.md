---
layout: default
class: Analyzer
title: packages 
summary: A list of package names filtered by a query language
---


The `packages` macro provides a query function over the contained packages of a bundle. A simple query language is used to query the packages and filter them.
For example if you want to export all packages that are annotated with the `@org.example.Export` annotation:

    Export-Package: ${packages;ANNOTATED;org.example.Export}

All pattern matching is based on fully qualified name and uses the globbing model.

More examples:

    # List all packages in the bundle, irrespective of how they were included
    All-Packages: ${packages}

    # List all packages, alternative syntax
    All-Packages: ${packages;ANY}

    # List packages containing the substring 'export'
    Exports: ${packages;NAMED;*export*}

The following table specifies the available query options:

<table>
<thead>
  <tr>
    <th>Query</th>
    <th>Parameter</th>
    <th>Description</th>
  </tr>
</thead>

<tr>
  <td>ANY</td>
  <td></td>
  <td>Matches any package</td>
</tr>

<tr>
  <td>ANNOTATED</td>
  <td>PATTERN</td>
  <td>The package must have an annotation that matches the pattern. The annotation must be either CLASS or RUNTIME retained, and placed on the `package-info.class` for the package.</td>
</tr>

<tr>
  <td>NAMED</td>
  <td>PATTERN</td>
  <td>The package FQN must match the given pattern.</td>
</tr>
</table>
