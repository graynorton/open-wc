---
title: Configuration options
eleventyNavigation:
  key: Configuration options
  parent: Rollup Plugin HTML
  order: 30
---

All configuration options are optional if an option is not set the plugin will fall back to smart defaults. See below example use cases.

### name

Type: `string`

Name of the generated HTML file. If `files` is set, defaults to the `files` filename, otherwise defaults to `index.html`.

### files

Type: `string|strings[]`

Paths to the HTML file to use as input. Modules in this files are bundled and the HTML is used as the template for the generated HTML file. This may also be a glob pattern.

### rootDir

Type: `string`

Path to a directory which will serve as the starting point for all `files`. The final file tree will result in relative urls to this rootDir.

### flatten

Type: `boolean`

Whether or not the folder in `filePaths` should be stripped, and all files should be placed in a single folder. (Defaults to true)

### html

Type: `string|[{ name: string, html: string}]`

Provide the HTML directly as string. If multiple files are provided then name and html are required.

### outputBundleName

Type: `string`

When using multiple build outputs, this is the name of the build that will be used to emit the generated HTML file.

### dir

Type: `string`

The directory to output the HTML file into. This defaults to the main output directory of your rollup build. If your build has multiple outputs in different directories, this defaults to the lowest directory on the file system.

### publicPath

Type: `string`

The public path where static resources are hosted. Any file requests (CSS, js, etc.) from the index.html will be prefixed with the public path.

### inject

Type: `boolean`

Whether to inject the rollup bundle into the output HTML. If using one of the input options, only the bundled modules in the HTML file are injected. Otherwise, all rollup bundles are injected. Default true. Set this to false if you need to apply some custom logic to how the bundle is injected.

### minify

Type: `boolean | object | (html: string) => string | Promise<string>`

When false, it does not do any minification. When true, does minification with default settings. When an object, does minification with a custom config. When a function, the function is called with the html and should return the minified html. Defaults to true.

Default minification is done using [html-minifier](https://github.com/kangax/html-minifier). When passing an object, the object is given to `html-minifier` directly so you can use any of the regular minify options.

### template

Type: `string | (args: TemplateArgs) => string | Promise<string>`

Template to inject js bundle into. It can be a string or an (async) function. If an input is set, that is used as the default output template. Otherwise defaults to a simple html file.

For more info see the [configuration type definitions](#configuration-types).

### transform

Type: `TransformFunction | TransformFunction[]`

TransformFunction: `(html: string, args: TransformArgs) => string | Promise<string>`

Function or array of functions that transform the final HTML output.

For more info see the [configuration type definitions](#configuration-types).
