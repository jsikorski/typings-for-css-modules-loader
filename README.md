[![npm][npm]][npm-url]
[![build][build]][build-url]
[![deps][deps]][deps-url]

# typings-for-css-modules-loader

Webpack loader that generates TypeScript typings for CSS modules from css-loader on the fly

## Disclaimer

This repository is a fork of the unmaintained https://github.com/Jimdo/typings-for-css-modules-loader repository.

## Installation

Install via npm `npm install --save-dev @teamsupercell/typings-for-css-modules-loader`

**webpack.config.js**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          "style-loader",
          "@teamsupercell/typings-for-css-modules-loader",
          {
            loader: "css-loader",
            options: { modules: true }
          }
        ]
      }
    ]
  }
};
```

## Options

|             Name              |    Type    | Description                                                                  |
| :---------------------------: | :--------: | :--------------------------------------------------------------------------- |
|    **[`banner`](#banner)**    | `{String}` | To add a 'banner' prefix to each generated `*.d.ts` file                     |
| **[`formatter`](#formatter)** | `{String}` | Formats the generated `*.d.ts` file with specified formatter, eg. `prettier` |
|       **[`eol`](#eol)**       | `{String}` | Newline character to be used in generated `*.d.ts` files                     |

### `banner`

To add a "banner" prefix to each generated `*.d.ts` file, you can pass a string to this option as shown below. The prefix is quite literally prefixed into the generated file, so please ensure it conforms to the type definition syntax.

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          {
            loader: "@teamsupercell/typings-for-css-modules-loader",
            options: {
              banner:
                "// autogenerated by typings-for-css-modules-loader. \n// Please do not change this file!"
            }
          },
          {
            loader: "css-loader",
            options: { modules: true }
          }
        ]
      }
    ]
  }
};
```

### `formatter`

Possible options: `none` and `prettier` (requires `prettier` package to be installed). Defaults to prettier if `prettier` module can be resolved.

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          {
            loader: "@teamsupercell/typings-for-css-modules-loader",
            options: {
              formatter: "prettier"
            }
          },
          {
            loader: "css-loader",
            options: { modules: true }
          }
        ]
      }
    ]
  }
};
```

### `eol`

"Newline character to be used in generated d.ts files. By default a value from `require('os').eol` is used.
This option is ignored when [`formatter`](#formatter) `prettier` is used.

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          {
            loader: "@teamsupercell/typings-for-css-modules-loader",
            options: {
              eol: "\r\n"
            }
          },
          {
            loader: "css-loader",
            options: { modules: true }
          }
        ]
      }
    ]
  }
};
```

## Example

Imagine you have a file `~/my-project/src/component/MyComponent/myComponent.scss` in your project with the following content:

```scss
.some-class {
  // some styles
  &.someOtherClass {
    // some other styles
  }
  &-sayWhat {
    // more styles
  }
}
```

Adding the `typings-for-css-modules-loader` will generate a file `~/my-project/src/component/MyComponent/myComponent.scss.d.ts` that has the following content:

```ts
declare namespace MyComponentScssModule {
  export interface IMyComponentScss {
    "some-class": string;
    someOtherClass: string;
    "some-class-sayWhat": string;
  }
}

declare const MyComponentScssModule: MyComponentScssModule.IMyComponentScss & {
  /** WARNING: Only available when `css-loader` is used without `style-loader` or `mini-css-extract-plugin` */
  locals: MyComponentScssModule.IMyComponentScss;
};

export = MyComponentScssModule;
```

```ts
// using wildcard export when used with style-loader or mini-css-extract-plugin
// or default export only when typescript `esModuleInterop` enabled
import * as styles from "./myComponent.scss";

console.log(styles["some-class"]);
console.log(styles.someOtherClass);
```

```ts
// using locals export when used without style-loader or mini-css-extract-plugin
import { locals } from "./myComponent.scss";

console.log(locals["some-class"]);
console.log(locals.someOtherClass);
```

### Example in Visual Studio Code

![typed-css-modules](https://cloud.githubusercontent.com/assets/749171/16340497/c1cb6888-3a28-11e6-919b-f2f51a282bba.gif)

## Upgrade from v1:
- Update webpack config
  - This package no longer replaces `css-loader`, but it has to be added alongside `css-loader`:
  - `css-loader` is no longer a peer dependency due to the change above
  - `css-loader` will need to be configured to output CSS Modules (e.g. `options: { modules: true; }`)
```diff
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: [
          "style-loader",
          {
            loader: "@teamsupercell/typings-for-css-modules-loader",
            options: {
              // pass all the options for `css-loader` to `css-loader`, eg.
-             namedExport: true,
-             modules: true
            }
          },
+         {
+           loader: "css-loader",
+           options: {
+             modules: true
+           }
+         },
        ]
      }
    ]
  }
};
```

## Support

As the loader just acts as an intermediary it can handle all kind of css preprocessors (`sass`, `scss`, `stylus`, `less`, ...).
The only requirement is that those preprocessors have proper webpack loaders defined - meaning they can already be loaded by webpack anyways.

## Requirements

The loader is supposed to be used with `css-loader`(https://github.com/webpack/css-loader). Thus it is a peer-dependency and the expected loader to create CSS Modules.

## Known issues

### Webpack rebuilds / builds slow

As the loader generates typing files, it is wise to tell webpack to ignore them.
The fix is luckily very simple. Webpack ships with a "WatchIgnorePlugin" out of the box.
Simply add this to your webpack plugins:

```
plugins: [
    new webpack.WatchIgnorePlugin([
      /css\.d\.ts$/
    ]),
    ...
  ]
```

where `css` is the file extension of your style files. If you use `sass` you need to put `sass` here instead. If you use `less`, `stylus` or any other style language use their file ending.

### Typescript does not find the typings

As the webpack process is independent from your typescript "runtime" it may take a while for typescript to pick up the typings.

It is possible to write a custom webpack plugin using the `fork-ts-checker-service-before-start` hook from https://github.com/TypeStrong/fork-ts-checker-webpack-plugin#plugin-hooks to delay the start of type checking until all the `*.d.ts` files are generated. Potentially, this plugin can be included in this repository.

[npm]: https://img.shields.io/npm/v/@teamsupercell/typings-for-css-modules-loader.svg
[npm-url]: https://npmjs.com/package/@teamsupercell/typings-for-css-modules-loader
[build]: https://travis-ci.com/TeamSupercell/typings-for-css-modules-loader.svg?branch=master
[build-url]: https://travis-ci.com/TeamSupercell/typings-for-css-modules-loader
[deps]: https://david-dm.org/@teamsupercell/typings-for-css-modules-loader.svg
[deps-url]: https://david-dm.org/@teamsupercell/typings-for-css-modules-loader
