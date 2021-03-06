# Simple Neutrino Customization

No two JavaScript projects are ever the same, and as such there may be times when you will need to make modifications
to the way your Neutrino preset is building your project. By defining a configuration object within your package.json,
Neutrino will merge this information with those provided by your presets, overriding those options with your custom
configuration.

_Note: Using package.json for customization tends to be quite verbose for anything more than simple overrides. If this
is not to your liking, consider moving your overrides to their own file using
[Advanced Customization](./advanced.md)._

## Prepare package.json

First, you will need to define a `neutrino` section within your package.json. You
[may have already done this](../usage.md#using-multiple-presets) if you
specified your presets through `neutrino` as opposed to flags through `scripts`:

```json
{
  "neutrino": {
    "use": [
      "neutrino-preset-react",
      "neutrino-preset-karma"
    ]
  },
  "scripts": {
    "start": "neutrino start",
    "build": "neutrino build"
  }
}
```

## Overriding Neutrino options

Neutrino has a number of useful options for customizing its behavior, as well as the behavior of presets and middleware.
You can override these options using an object at `neutrino.options`:

### `options.root`

Set the base directory which Neutrino middleware and presets operate on. Typically this is the project directory where
the package.json would be located. If the option is not set, Neutrino defaults it to `process.cwd()`. If a relative
path is specified, it will be resolved relative to `process.cwd()`; absolute paths will be used as-is.

```js
{
  "neutrino": {
    "options": {
      // if not specified, defaults to process.cwd()
      
      // relative, resolves to process.cwd() + ./website
      "root": "./website",

      // absolute
      "root": "/code/website"
    }
  }
}
```

### `options.source`

Set the directory which contains the application source code. If the option is not set, Neutrino defaults it to `src`.
If a relative path is specified, it will be resolved relative to `options.root`; absolute paths will be used as-is.

```js
{
  "neutrino": {
    "options": {
      // if not specified, defaults to options.root + src
        
      // relative, resolves to options.root + ./lib
      "source": "./lib",
      
      // absolute
      "source": "/code/website/lib"
    }
  }
}
```

### `options.output`

Set the directory which will be the output of built assets. If the option is not set, Neutrino defaults it to `build`.
If a relative path is specified, it will be resolved relative to `options.root`; absolute paths will be used as-is.

```js
{
  "neutrino": {
    "options": {
      // if not specified, defaults to options.root + build
        
      // relative, resolves to options.root + ./dist
      "output": "./dist",
      
      // absolute
      "output": "/code/website/dist"
    }
  }
}
```

### `options.tests`

Set the directory that contains test files. If the option is not set, Neutrino defaults it to `test`.
If a relative path is specified, it will be resolved relative to `options.root`; absolute paths will be used as-is.

```js
{
  "neutrino": {
    "options": {
      // if not specified, defaults to options.root + test
        
      // relative, resolves to options.root + ./testing
      "tests": "./testing",
      
      // absolute
      "tests": "/code/website/testing"
    }
  }
}
```

### `options.entry`

Set the main entry point for the application. If the option is not set, Neutrino defaults it to `index.*` - the extension is resolved by Webpack. If a relative path is specified, it will be resolved relative to `options.source`; absolute paths will be used as-is.

The main file by default is not required to be in JavaScript format. It also potentially may be JSX, TypeScript or any other preprocessor language. These extensions should be specified in a preset in `neutrino.config.resolve.extensions`. 

```js
{
  "neutrino": {
    "options": {
      // if not specified, defaults to options.source + index
        
      // relative, resolves to options.source + ./entry.js
      "entry": "./entry.js",
      
      // absolute
      "entry": "/code/website/lib/entry.js"
    }
  }
}
```

### `options.node_modules`

Set the directory which contains the Node.js modules of the project. If the option is not set, Neutrino defaults it to
`node_modules`. If a relative path is specified, it will be resolved relative to `options.root`; absolute paths will be
used as-is.

```js
{
  "neutrino": {
    "options": {
      // if not specified, defaults to options.root + node_modules
        
      // relative, resolves to options.root + ./modules
      "node_modules": "./modules"
      
      // absolute
      "node_modules": "/code/website/modules"
    }
  }
}
```

### Middleware or preset options

Some middleware and presets also have their own custom options. Consult the documentation for the package for specific
details on its customization.

## Overriding build configuration

Add a new property to `neutrino` named `config`. This will be an object where we can provide configuration data:

```json
{
  "neutrino": {
    "use": [],
    "config": {

    }
  }
}
```

Populate this object with configuration overrides. This is not a Webpack configuration, but rather a Neutrino-compatible
object based on [webpack-chain](https://github.com/mozilla-neutrino/webpack-chain). A schematic of what this structure
looks like is located on the [webpack-chain docs](https://github.com/mozilla-neutrino/webpack-chain#merging-config).

## Usage

### Entries

Add files to named entry points, or define new entry points. This is a key named `entry`, with a value being an object.
This maps to points to enter the application. At this point the application starts executing.

_Example: Define an entry point named `vendor` that bundles React packages separately from the application code._

```json
{
  "neutrino": {
    "config": {
      "entry": {
        "vendor": [
          "react",
          "react-dom",
          "react-hot-loader",
          "react-router-dom"
        ]
      }
    }
  }
}
```

### Module

The `module` object defines how the different types of modules within a project will be treated. Any additional
properties attached to `module` not defined below will be set on the final module configuration.

#### Module Rules

Using `module.rule` creates rules that are matched to requests when modules are created. These rules can modify how the
module is created. They can apply loaders to the module, or modify the parser. 

Using `module.rule.loader` allows to you define the Webpack loader and its options for processing a particular rule.
This loader is usually a `dependency` or `devDependency` of your project. Each `loader` object can specify a property
for the string `loader` and an `options` object.

_Example: Add LESS loading to the project, by overriding the `style` rule._

```json
{
  "dependencies": {
    "less": "^2.7.2",
    "less-loader": "^2.2.3"
  },
  "neutrino": {
    "config": {
      "module": {
        "rule": {
          "style": {
            "test": "\\.less$",
            "loader": {
              "less": {
                "loader": "less-loader",
                "options": {
                  "noIeCompat": true
                }
              }
            }
          }
        }
      }
    }
  }
}
```

### Output

The `output` object contains a set of options instructing Webpack on how and where it should output your bundles,
assets, and anything else you bundle or load with Webpack. This option can be any property/value combination that
[Webpack accepts](https://webpack.js.org/configuration/output/).

_Example: Change the public path of the application._

```json
{
  "neutrino": {
    "config": {
      "output": {
        "publicPath": "https://cdn.example.com/assets/"
      }
    }
  }
}
```

### Node

Use `node` to customize the Node.js environment using polyfills or mocks:

_Example: mock the `__filename` and `__dirname` Node.js globals._

```json
{
  "neutrino": {
    "config": {
      "node": {
        "__filename": "mock",
        "__dirname": "mock"
      }
    }
  }
}
```

### DevServer

Use `devServer` to customize webpack-dev-server and change its behavior in various ways.

_Example: gzip the application when serving and listen on port 9000._

```json
{
  "neutrino": {
    "config": {
      "devServer": {
        "compress": true,
        "port": 9000
      }
    }
  }
}
```

### Resolve

Use `resolve` to change how modules are resolved. When using `resolve.extensions` and `resolve.modules`, these should be
specified as arrays, and will be merged with their respective definitions used in inherited presets. See the
webpack-chain docs for more details on this structure.

_Example: Add `.mjs` as a resolving extension and specify modules are located in a `custom_modules` directory._

```json
{
  "neutrino": {
    "config": {
      "resolve": {
        "extensions": [".mjs"],
        "modules": ["custom_modules"]
      }
    }
  }
}
```

### ResolveLoader

Use `resolveLoader` to change how loader packages are resolved. When using `resolveLoader.extensions` and
`resolveLoader.modules`, these should be specified as arrays, and will be merged with their respective definitions used
in inherited presets. See the webpack-chain docs for more details on this structure.

_Example: Add `.loader.js` as a loader extension and specify modules are located in a `web_loaders` directory._

```json
{
  "neutrino": {
    "config": {
      "resolve": {
        "extensions": [".loader.js"],
        "modules": ["web_loaders"]
      }
    }
  }
}
```

### Additional configuration

Any top-level properties you set on `neutrino.config` will be added to the configuration.

_Example: Change the Webpack performance options to error when exceeding performance budgets._

```json
{
  "neutrino": {
    "config": {
      "performance": {
        "hints": "error"
      }
    }
  }
}
```

## Advanced Configuration

With the options defined above in your package.json, you can perform a variety of build customizations on a per-project
basis. In the event that you need more customization than what is afforded through JSON, consider either switching to
[advanced configuration](./advanced.md), or [creating your own preset](../creating-presets.md).
