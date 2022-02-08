# Stylelint mirror for pre-commit

Mirrors all [Stylelint](https://stylelint.io/) releases for the [pre-commit](https://pre-commit.com/) hooks framework.

## Usage

Add the following to your `.pre-commit-config.yaml`:

```yaml
- repo: https://github.com/thibaudcolas/pre-commit-stylelint
  rev: v14.4.0
  hooks:
    - id: stylelint
```

Change [rev](https://pre-commit.com/#repos-rev) to the stylelint version you want to use. See all [available versions as tags](https://github.com/thibaudcolas/pre-commit-stylelint/tags).

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

## Why this mirror exists

See:

- [Document pre-commit framework integration stylelint/stylelint#5373](https://github.com/stylelint/stylelint/issues/5373)
- [Add mirror for stylelint pre-commit/pre-commit#1768](https://github.com/pre-commit/pre-commit/issues/1768)
- [Suggestion: move pre-commit support to another repo prettier/prettier#8925](https://github.com/prettier/prettier/issues/8925)
- [Chore: Add .pre-commit-hooks.yaml file eslint/eslint#13628](https://github.com/eslint/eslint/pull/13628)

## Credits

This repository is based on [mirrors-prettier](https://github.com/pre-commit/mirrors-prettier), with the hook configuration options of [awebdeveloper/pre-commit-stylelint](https://github.com/awebdeveloper/pre-commit-stylelint).
