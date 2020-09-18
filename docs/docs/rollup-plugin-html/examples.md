---
title: Examples
eleventyNavigation:
  key: Examples
  parent: Rollup Plugin HTML
  order: 30
---

### Simple HTML page

When used without any options, the plugin will inject your rollup bundle into a basic HTML page. Useful for developing a simple application.

<details>

<summary>Show example</summary>

```js
import html from '@open-wc/rollup-plugin-html';
export default {
  input: './my-app.js',
  output: { dir: 'dist' },
  plugins: [html()],
};
```

</details>

### Input from file

During development, you will probably already have an HTML file which imports your application's modules. You can use this same file as the input of the html plugin, which will bundle any modules inside and output the same HTML minified optimized.

To do this, you can set the html file as input for rollup:

<details>

<summary>Show example</summary>

```js
import html from '@open-wc/rollup-plugin-html';
export default {
  input: 'index.html',
  output: { dir: 'dist' },
  plugins: [html()],
};
```

</details>

#### Input from multiple html files

You can also work with multiple html files. An example of this could be when you're using this plugin in combination with a static site generator. To target all your html files, you can provide a glob as input. You can specify the `flatten: false` property to retain the folder structure.

<details>

<summary>Show example</summary>

```js
import html from '@open-wc/rollup-plugin-html';
export default {
  input: '**/*.html',
  // or even
  input: ['index.html', 'pages/*.html'],
  output: { dir: 'dist' },
  plugins: [html({ flatten: false, rootDir: '_site' })],
};
```

</details>

You can also set the `files` property on the html plugin. This will take precedence over rollup's input.
Additionally, it supports an array of files/globs and can be used if you want to use multiple html plugins with different options:

<details>

<summary>Show example</summary>

```js
import html from '@open-wc/rollup-plugin-html';
export default {
  output: { dir: 'dist' },
  plugins: [
    html({
      files: ['index.html', 'docs/**/*.html'],
    }),
    html({
      files: 'pages/index.html',
      /* different options */
    }),
  ],
};
```

</details>

### Input from string

Sometimes the HTML you want to use as input is not available on the file system. With the `html` option you can provide the HTML as a string directly. This is useful for example when using rollup from javascript directly.

<details>

<summary>Show example</summary>

```js
import html from '@open-wc/rollup-plugin-html';
export default {
  output: { dir: 'dist' },
  plugins: [
    html({
      html: '<html><script type="module" src="./app.js"></script></html>',
    }),
  ],
};
```

</details>

### Input from multiple html strings

When creating multiple html files via strings the following additional options are required.

- `name`: name of your html file (incl. relative folders)
- `html`: the html as a string

<details>

<summary>Show example</summary>

```js
import html from '@open-wc/rollup-plugin-html';
export default {
  output: { dir: 'dist' },
  plugins: [
    html({
      rootDir: __dirname,
      html: [
        { name: 'index.html', html: '<html>...</html>' },
        {
          name: 'pages/page-a.html',
          html: '<html>...</html>',
        },
        {
          name: 'pages/page-b.html',
          html: '<html>...</html>',
        },
      ],
    }),
  ],
};
```

</details>

### Template

With the `template` option, you can let the plugin know where to inject the rollup build into. This option can be a string or an (async) function which returns a string.

<details>

<summary>Show example</summary>

Template as a string:

```js
import html from '@open-wc/rollup-plugin-html';
export default {
  output: { dir: 'dist' },
  plugins: [
    html({
      template: `
      <html>
        <head><title>My app</title></head>
        <body></body>
      </html>`,
    }),
  ],
};
```

Template as a function:

```js
import fs from 'fs';
import html from '@open-wc/rollup-plugin-html';

export default {
  output: { dir: 'dist' },
  plugins: [
    html({
      template() {
        return new Promise((resolve) => {
          const indexPath = path.join(__dirname, 'index.html');
          fs.readFile(indexPath, 'utf-8', (err, data) => {
            resolve(data);
          });
        });
      }
    }
  ],
};
```

</details>

### Manually inject build output

If you want to control how the build output is injected on the page, disable the `inject` option, and use the arguments provided to the template function.

<details>

<summary>Show example</summary>

With a regular template function:

```js
import html from '@open-wc/rollup-plugin-html';
export default {
  input: './app.js',
  output: { dir: 'dist' },
  plugins: [
    html({
      name: 'index.html',
      inject: false,
      template({ bundle }) {
        return `
        <html>
          <head>
            ${bundle.entrypoints.map(bundle => e =>
              `<script type="module" src="${e.importPath}"></script>`,
            )}
          </head>
        </html>
      `;
      },
    }),
  ],
};
```

When one of the input options is used, the input html is available in the template function. You can use this to inject the bundle into your existing HTML page:

```js
import html from '@open-wc/rollup-plugin-html';
export default {
  input: './app.js',
  output: { dir: 'dist' },
  plugins: [
    html({
      files: './index.html',
      inject: false,
      template({ inputHtml, bundle }) {
        return inputHtml.replace(
          '</body>',
          `<script type="module" src="${bundle[0].entrypoints[0].importPath}"></script></body>`,
        );
      },
    }),
  ],
};
```

</details>

### Transform output HTML

You can use the `transform` option to manipulate the output HTML before it's written to disk. This is useful for setting meta tags or environment variables based on input from other sources.

`transform` can be a single function or an array. This makes it easy to compose transformations.

<details>
  <summary>View example</summary>

Inject language attribute:

```js
import html from '@open-wc/rollup-plugin-html';
export default {
  output: { dir: 'dist' },
  plugins: [
    html({
      files: './index.html',
      transform: html => html.replace('<html>', '<html lang="en-GB">'),
    }),
  ],
};
```

Inject language attributes and environment variables:

```js
import html from '@open-wc/rollup-plugin-html';
import packageJson from './package.json';

const watchMode = process.env.ROLLUP_WATCH === 'true';

export default {
  output: { dir: 'dist' },
  plugins: [
    html({
      files: './index.html',
      transform: [
        html => html.replace('<html>', '<html lang="en-GB">'),
        html =>
          html.replace(
            '<head>',
            `<head>
              <script>
                window.ENVIRONMENT = "${watchMode ? 'DEVELOPMENT' : 'PRODUCTION'}";
                window.APP_VERSION = "${packageJson.version}";
              </script>`,
          ),
      ],
    }),
  ],
};
```

</details>

### Public path

By default, all imports are made relative to the HTML file and expect files to be in the rollup output directory. With the `publicPath` option you can modify where files from the HTML file are requested from.

<details>
  <summary>View example</summary>

```js
import html from '@open-wc/rollup-plugin-html';

export default {
  output: { dir: 'dist' },
  plugins: [
    html({
      files: './index.html',
      publicPath: '/static/',
    }),
  ],
};
```

</details>

### Multiple build outputs

It is possible to create multiple rollup build outputs and inject both bundles into the same HTML file. This way you can ship multiple bundles to your users, and load the most optimal version for the user's browser.

<details>
<summary>View example</summary>

When you configure rollup to generate multiple build outputs you can inject all outputs into a single HTML file.

To do this, create one parent `@open-wc/rollup-plugin-html` instance and use `addOutput` to create two child plugins for each separate rollup output.

Each output defines a unique name, this can be used to retrieve the correct bundle from `bundles` argument when creating the HTML template.

```js
import html from '@open-wc/rollup-plugin-html';

const htmlPlugin = html({
  name: 'index.html',
  inject: false,
  template({ bundles }) {
    return `
      <html>
        <body>
        ${bundles.modern.entrypoints.map(
          e => `<script type="module" src="${e.importPath}"></script>`,
        )}

        <script nomodule src="./systemjs.js"></script>
        ${bundles.legacy.entrypoints.map(
          e => `<script nomodule>System.import("${e.importPath}")</script>`,
        )}
        </body>
      </html>
    `;
  },
});

export default {
  input: './app.js',
  output: [
    {
      format: 'es',
      dir: 'dist',
      plugins: [htmlPlugin.addOutput('modern')],
    },
    {
      format: 'system',
      dir: 'dist',
      plugins: [htmlPlugin.addOutput('legacy')],
    },
  ],
  plugins: [htmlPlugin],
};
```

</details>

If your outputs use different outputs directories, you need to set the `outputBundleName` option to specify which build to use to output the HTML file.

<details>
<summary>View example</summary>

```js
import html from '@open-wc/rollup-plugin-html';

const htmlPlugin = html({
  name: 'index.html',
  inject: false,
  outputBundleName: 'modern',
  template({ bundles }) {
    return `
      ...
    `;
  },
});

export default {
  input: './app.js',
  output: [
    {
      format: 'es',
      dir: 'dist',
      plugins: [htmlPlugin.addOutput('modern')],
    },
    {
      format: 'system',
      dir: 'dist',
      plugins: [htmlPlugin.addOutput('legacy')],
    },
  ],
  plugins: [htmlPlugin],
};
```

</details>

## Details for Plugin rootDir

When keeping the folder structures it may mean that your `rootDir` is not your current working directory.
For example, imagine all the html files are generated into a `_site-in-html` folder.

```
.
├── _site-in-html
│   ├── page-a
│   │   └── index.html
│   └── index.html
├── rollup.config.js
└── package.json
```

If you provide `input: '_site-in-html/**/*.html';` it will result in files like `dist/_site-in-html/index.html`. If the dist folder gets automatically uploaded to a static hosting service then it will result in this url `https://my-domain.com/_site_in-html/index.html`.
By defining `rootDir: './_site-in-html'` and `input: '**/*.html';` we can get files like `dist/index.html` and urls like `https://my-domain.com/`.

<details>

<summary>Show example</summary>

```js
import html from '@open-wc/rollup-plugin-html';
export default {
  input: '**/*.html',
  output: { dir: 'dist' },
  plugins: [html({ flatten: false, rootDir: './_site-in-html' })],
};
```

</details>
