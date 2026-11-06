# Stylelint mirror for prek and pre-commit

Mirrors all\* [Stylelint](https://stylelint.io/) releases for [prek](https://prek.j178.dev/) / pre-commmit, based on the [pre-commit](https://pre-commit.com/) hooks framework.

## Usage

Add the following to your `.pre-commit-config.yaml`:

```yaml
- repo: https://github.com/thibaudcolas/pre-commit-stylelint
  rev: v14.4.0
  hooks:
    - id: stylelint
```

Or with prek and a `prek.toml`:

```toml
[[repos]]
repo = "https://github.com/thibaudcolas/pre-commit-stylelint"
rev = "v14.4.0"
hooks = [
  { id = "stylelint" },
]
```

Change [rev](https://pre-commit.com/#repos-rev) to the stylelint version you want to use from the [available versions as tags](https://github.com/thibaudcolas/pre-commit-stylelint/tags).

### With additional dependencies

To use plugins or shared configurations with `stylelint`, declare them (and stylelint) with [`additional_dependencies`](https://pre-commit.com/#config-additional_dependencies):

```yaml
- repo: https://github.com/thibaudcolas/pre-commit-stylelint
  rev: v14.4.0
  hooks:
    - id: stylelint
      additional_dependencies:
        # stylelint itself needs to be here when using additional_dependencies.
        - stylelint@14.4.0
        - stylelint-config-standard-scss@3.0.0
        # Package names starting with `@` need to be quoted.
        - "@scope/my-awesome-plugin@0.12.0"
```

#### Stylelint 17 and later

From v17, Stylelint can’t find packages from `additional_dependencies` when your config refers to them by name in `extends`, `plugins` or `customSyntax`. Every file fails with:

```
ConfigurationError: Could not find "stylelint-config-standard-scss". Do you need to install the package or use the "configBasedir" option?
```

pre-commit and prek install `additional_dependencies` into the hook’s own environment and put its `node_modules` folder on `NODE_PATH`. Stylelint 16 searched `NODE_PATH` as a fallback, but Stylelint 17 dropped that fallback ([stylelint/stylelint#8886](https://github.com/stylelint/stylelint/pull/8886)) and only looks in `node_modules` folders above your config file.

You can work around this with a JavaScript config that resolves the package paths itself, since Node’s `require.resolve` still reads `NODE_PATH`:

```js
// stylelint.config.mjs
import fs from "node:fs";
import { createRequire } from "node:module";
import path from "node:path";

const require = createRequire(import.meta.url);

function resolve(name) {
  try {
    return require.resolve(name);
  } catch {
    // ESM-only packages such as stylelint-config-standard-scss can't be
    // resolved by require, so point Stylelint at their index.js instead.
    return (
      require.resolve
        .paths(name)
        ?.map((dir) => path.join(dir, name, "index.js"))
        .find((file) => fs.existsSync(file)) ?? name
    );
  }
}

export default {
  extends: [resolve("stylelint-config-standard-scss")],
};
```

This still uses your project’s `node_modules` first when the packages are installed there, so running `npx stylelint` and editor integrations keep working. It won’t find an ESM-only package whose entry point is somewhere other than `index.js` at the package root.

### With additional stylelint options

Use pre-commit’s [`args`](https://pre-commit.com/#config-args):

```yaml
- repo: https://github.com/thibaudcolas/pre-commit-stylelint
  rev: v14.4.0
  hooks:
    - id: stylelint
      args: [--report-needless-disables, --report-invalid-scope-disables]
```

### Automatically fix issues

Stylelint supports a [`--fix`](https://stylelint.io/user-guide/usage/cli#--fix) option that will automatically fix the code. This requires the [`args`](https://pre-commit.com/#config-args) option:

```yaml
- repo: https://github.com/thibaudcolas/pre-commit-stylelint
  rev: v14.4.0
  hooks:
    - id: stylelint
      args: [--fix]
```

### File types

By default, this hook will run stylelint for the following file extensions: `.css`, `.sass`, `.scss`. If you want to change this, use [`files`](https://pre-commit.com/#config-files):

```yaml
- repo: https://github.com/thibaudcolas/pre-commit-stylelint
  rev: v14.4.0
  hooks:
    - id: stylelint
      files: \.(scss|vue)$
```

## Migrating from `awebdeveloper/pre-commit-stylelint`

Compared to [awebdeveloper/pre-commit-stylelint](https://github.com/awebdeveloper/pre-commit-stylelint), this repository makes it possible to control the stylelint version with the `rev` property. This tiny difference removes the guesswork of trying to understand how to set up the hook.

Switching is just a matter of updating the `repo`, and setting a `rev` for the desired version:

```diff
-- repo: https://github.com/awebdeveloper/pre-commit-stylelint
-  rev: c4c991cd38b0218735858716b09924f8b20e3812
+- repo: https://github.com/thibaudcolas/pre-commit-stylelint
+  rev: v14.4.0
  hooks:
    - id: stylelint
```

### With an unavailable stylelint versions

\* some versions of stylelint may be missing, though this isn’t the case as of June 2024 (v16.6.1 and below). This repository doesn’t automatically mirror patch releases to older versions of Stylelint, when they get released after another version with a "bigger" version number. If this happens, please [open an issue](https://github.com/thibaudcolas/pre-commit-stylelint/issues/new) so we manually add the release to the mirror.

As a temporary workaround, you can configure pre-commit to install from any arbitrary version of stylelint with [`additional_dependencies`](https://pre-commit.com/#config-additional_dependencies):

```yaml
- repo: https://github.com/thibaudcolas/pre-commit-stylelint
  rev: v14.4.0
  hooks:
    - id: stylelint
      additional_dependencies:
        # v14.16.43 isn’t available as a tag, so we instead load it directly from npm:
        - stylelint@14.16.43
```

## Why this mirror exists

pre-commit itself has poor support for the Node ecosystem, preferring to install projects with git rather than using packages as published. Setting up a mirror completely sidesteps those issues, and results in [much faster installation times](https://github.com/thibaudcolas/pre-commit-stylelint/discussions/1).

See:

- [Document pre-commit framework integration stylelint/stylelint#5373](https://github.com/stylelint/stylelint/issues/5373)
- [Add mirror for stylelint pre-commit/pre-commit#1768](https://github.com/pre-commit/pre-commit/issues/1768)
- [Suggestion: move pre-commit support to another repo prettier/prettier#8925](https://github.com/prettier/prettier/issues/8925)
- [Chore: Add .pre-commit-hooks.yaml file eslint/eslint#13628](https://github.com/eslint/eslint/pull/13628)

## Credits

This repository is based on [mirrors-prettier](https://github.com/pre-commit/mirrors-prettier), with the hook configuration options of [awebdeveloper/pre-commit-stylelint](https://github.com/awebdeveloper/pre-commit-stylelint).
