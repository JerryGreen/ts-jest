# ts-jest

[![npm version](https://badge.fury.io/js/ts-jest.svg)](https://badge.fury.io/js/ts-jest)
[![NPM downloads](https://img.shields.io/npm/dm/ts-jest.svg?style=flat)](https://npmjs.org/package/ts-jest)
[![Greenkeeper badge](https://badges.greenkeeper.io/kulshekhar/ts-jest.svg)](https://greenkeeper.io/)

[![Build Status for linux](https://travis-ci.org/kulshekhar/ts-jest.svg?branch=master)](https://travis-ci.org/kulshekhar/ts-jest)
[![Build Status for Windows](https://ci.appveyor.com/api/projects/status/g8tt9qd7usv0tolb/branch/master?svg=true)](https://ci.appveyor.com/project/kulshekhar/ts-jest/branch/master)

> Note: Looking for collaborators. [Want to help improve ts-jest?](https://github.com/kulshekhar/ts-jest/issues/223)

ts-jest is a TypeScript preprocessor with sensible defaults, that quickly allows you to get going, testing typescript code with jest.

## Table of Contents
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Versioning](#versioning)
- [Usage](#usage)
  - [Coverage](#coverage)
  - [React Native](#react-native)
- [Options](#options)
  - [Known limitations for TS compiler options](#known-limitations-for-ts-compiler-options)
- [Tips](#tips)
  - [Importing packages written in TypeScript](#importing-packages-written-in-typescript)
- [How to Contribute](#how-to-contribute)
  - [Quickstart to run tests (only if you're working on this package)](#quickstart-to-run-tests-only-if-youre-working-on-this-package)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


    short intro
    Usage
    Sensible defaults
        sourcemap support
        Autolocates tsconfig
        Automatic transpilation of modules if enabled in tsconfig (mention skipbabel)
        Supports hoisting of jest.mock, mention skipBabel and limitations
    Configuration, for when you need that little extra
        TS_CONFIG stuff
        skipBabel flag
    Use cases
        Angular 2 (compile html stuff)
        React native
    Limitations
        Current limitations
    Tips
        same as now


## Usage

To use this in your project, run:
```sh
npm install --save-dev ts-jest
```
Modify your project's `package.json` so that the `jest` section looks something like:
```json
{
  "jest": {
    "transform": {
      ".(ts|tsx)": "<rootDir>/node_modules/ts-jest/preprocessor.js"
    },
    "testRegex": "(/__tests__/.*|\\.(test|spec))\\.(ts|tsx|js)$",
    "moduleFileExtensions": [
      "ts",
      "tsx",
      "js"
    ]
  }
}
```
This setup should allow you to write Jest tests in Typescript and be able to locate errors without any additional gymnastics.

### Versioning
From version `"jest": "17.0.0"` we are using same MAJOR.MINOR as [`Jest`](https://github.com/facebook/jest).
For `"jest": "< 17.0.0"` use `"ts-jest": "0.1.13"`. Docs for it see [here](https://github.com/kulshekhar/ts-jest/blob/e1f95e524ed62091736f70abf63530f1f107ec03/README.md).

### Coverage

Prior to version `20.0.0`, coverage reports could be obtained using the inbuilt coverage processor in `ts-jest`. Starting with version `20.0.0`, `ts-jest` delegates coverage processing to jest and no longer includes a coverage processor.

To generate coverage results, set the `mapCoverage` property in the `jest` configuration section to `true`.

> Please note that the `outDir` property in the `jest` configuration section is removed in coverage mode, due to [#201](https://github.com/kulshekhar/ts-jest/issues/201).

## Sensible Defaults
ts-jest tries to ship with sensible defaults, to get you on your feet as quickly as possible.

### Sourcemap support
Sourcemaps should work out of the box, that means your stacktraces should have the correct line numbers,
and your breakpoints should work.

### Automatically finds .tsconfig.json
ts-jest automatically finds your .tsconfig.json and uses that. If you want to run typescript with a special configuration, you [also can](#ADDLINK)//TODO

### Supports synthetic modules 
If you're on a codebase where you're using synthetic default exports, e.g. 
```javascript 1.6
//Regular default exports
import * as React from 'react';

//Synthetic default exports:
import React from 'react';
```
ts-jest tries to support that. If `allowSyntheticDefaultExports` is set to true in your tsconfig file, it uses babel
to automatically create the synthetic default exports for you - nothing else needed. 
You can opt-out of this behaviour with the [skipBabel flag](#Skipping Babel)

### Supports automatic of jest.mock() calls
[Just like Jest](https://facebook.github.io/jest/docs/manual-mocks.html#using-with-es-module-imports) ts-jest
automatically hoists your `jest.mock()` calls to the top of your file.
You can opt-out of this behaviour with the [skipBabel flag](#Skipping Babel)

## Configuration
Sometimes you just need a little extra configurability. 


### tsconfig
By default this package will try to locate `tsconfig.json` and use its compiler options for your `.ts` and `.tsx` files.

But you are able to override this behaviour and provide another path to your config for TypeScript by using `__TS_CONFIG__` option in `globals` for `jest`:
```json
{
  "jest": {
    "globals": {
      "__TS_CONFIG__": "my-tsconfig.json"
    }
  }
}
```
Or even declare options for `tsc` instead of using separate config, like this:
```json
{
  "jest": {
    "globals": {
      "__TS_CONFIG__": {
        "module": "commonjs",
        "jsx": "react"
      }
    }
  }
}
```
For all available `tsc` options see [TypeScript docs](https://www.typescriptlang.org/docs/handbook/compiler-options.html).

Note that if you haven't explicitly set the `module` property in the `__TS_CONFIG__` setting (either directly or through a separate configuration file), it will be overwritten to `commonjs` (regardless of the value in `tsconfig.json`) since that is the format Jest expects. This only happens during testing.

### Skipping Babel
If you want to skip the babel transpilation step in ts-jest, you can. This means `jest.mock()` calls will not be hoisted to the top,
and synthetic default exports will never be created.
Simply add skipBabel to your global variables under the `ts-jest` key:
```json
//This will skip babel transpilation
{
  "jest": {
    "globals": {
      "ts-jest": {
        "skipBabel": true
      }
    }
  }
}
```

## Use cases

### React Native

There is a few additional steps if you want to use it with React Native.

Install `babel-jest` and `babel-preset-react-native` modules.

```sh
npm install -D babel-jest babel-preset-react-native
```

Ensure `.babelrc` contains:

```json
{
  "presets": ["react-native"],
  "sourceMaps": "inline"
}
```

In `package.json`, inside `jest` section, the `transform` should be like this:
```json
"transform": {
  "^.+\\.js$": "<rootDir>/node_modules/babel-jest",
  ".(ts|tsx)": "<rootDir>/node_modules/ts-jest/preprocessor.js"
}
```

Fully completed jest section should look like this:

```json
"jest": {
    "preset": "react-native",
    "transform": {
      "^.+\\.js$": "<rootDir>/node_modules/babel-jest",
      ".(ts|tsx)": "<rootDir>/node_modules/ts-jest/preprocessor.js"
    },
    "testRegex": "(/__tests__/.*|\\.(test|spec))\\.(ts|tsx|js)$",
    "moduleFileExtensions": [
      "ts",
      "tsx",
      "js"
    ]
  }
```
If only testing typescript files then remove the `js` option in the testRegex.

You might want to use ES6 default imports, which will allow you to write things like
`import React from 'react';`

In that case you can add the following to your `.tsconfig`
```json
        "allowSyntheticDefaultImports": true
```

This will make ts-loader send the compiled typescript code through babel, and the above import will resolve.

This configuration will allow `debugger` statements to work properly in both WebStorm and VSCode.
Breakpoints currently only work in WebStorm.

## Angular 2
When using Jest with Angular (a.k.a Angular 2) apps you will likely need to parse HTML templates. If you're unable to add `html-loader` to webpack config (e.g. because you don't want to eject from `angular-cli`) you can do so by defining `__TRANSFORM_HTML__` key in `globals` for `jest`.

```json
{
  "jest": {
    "globals": {
      "__TRANSFORM_HTML__": true
    }
  }
}
```

You'll also need to extend your `transform` regex with `html` extension:
```json
{
  "jest": {
    "transform": {
      "^.+\\.(ts|tsx|js|html)$": "<rootDir>/node_modules/ts-jest/preprocessor.js"
    }
  }
}
```

## Tips
### Importing packages written in TypeScript

If you have dependencies on npm packages that are written in TypeScript but are
**not** published in ES5 you have to tweak your configuration. For example
you depend on a private scoped package `@foo/bar` you have to add following to
your Jest configuration:

```js
{
  // ...
  "transformIgnorePatterns": [
    "<rootDir>/node_modules/(?!@foo)"
  ]
  // ...
}
```

By default Jest ignores everything in `node_modules`. This setting prevents Jest from ignoring the package you're interested in, in this case `@foo`, while continuing to ignore everything else in `node_modules`.


## Known Limitations
### Known limitations for TS compiler options
- You can't use `"target": "ES6"` while using `node v4` in your test environment;
- You can't use `"jsx": "preserve"` for now (see [progress of this issue](https://github.com/kulshekhar/ts-jest/issues/63));
- If you use `"baseUrl": "<path_to_your_sources>"`, you also have to change `jest config` a little bit:
```json
"jest": {
  "moduleDirectories": ["node_modules", "<path_to_your_sources>"]
}
```

### Known Limitations for autohoisting
If the `jest.mock()` calls is placed after actual code, (e.g. after functions or classes) and `skipBabel` is not set,
the line numbers in stacktraces will be off.
We suggest placing the `jest.mock()` calls after the imports, but before any actual code.

## How to Contribute
If you have any suggestions/pull requests to turn this into a useful package, just open an issue and I'll be happy to work with you to improve this.

### Quickstart to run tests (only if you're working on this package)

```sh
git clone https://github.com/kulshekhar/ts-jest
cd ts-jest
npm install
npm test
```

**Note:** If you are cloning on Windows, you may have to run `git config --system core.longpaths true` for Windows to stop complaining about long filenames.

## License

Copyright (c) [Authors](AUTHORS).
This source code is licensed under the [MIT license](LICENSE).
