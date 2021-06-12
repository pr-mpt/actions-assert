# Assert

A GitHub Action for asserting **actual** is **expected** in GitHub Workflows.
Designed for GitHub Action integration tests and build pipelines.

* Cast action input values from strings to `type` for type safety
* Require and distribute reusable assertions via [npm][npm]
* Write custom Javascript assertions to meet unique testing requirements
* Run tests against multiple values using `each`

```yaml
jobs:
  test-actor:
    runs-on: ubuntu-latest
    steps:
      - name: Test actor is @shrink
        uses: pr-mpt/actions-assert@v2
        with:
          assertion: npm://@assertions/is-equal
          actual: "${{ github.actor }}"
          expected: shrink
```

[Jump to examples &darr;](#examples)

## Inputs

| Name | Description | Default | Examples |
| :--- | :---------- | :------ | :------- |
| **`assertion`** | **Reference to a supported [assertion](#assertions)** in `source://name` format | | **`npm://@assertions/is-equal`**<br/>**`local://is-even`** |
| `expected` | Value the assertion is looking for | | `Hello, World!` |
| `actual` | Value the assertion will test against the expected value | | `${{steps.fields.outputs.greeting}}` |
| `type` | A supported [data type](#data-types) that `actual` and `expected` will be cast to before performing assertion | `string` | `string` `json` `number` |
| `each` | Parse multi-line `actual` into many values and test each | `false` | `true` `false` |
| `local-path` | Path to directory containing `local` assertion | `${{github.workspace}}` | `.github/workflows/assertions` |

### Assertions

An `assertion` is a Javascript function that accepts `expected` and `actual`
parameters then returns a `Result`. A `Result` has a boolean `pass` parameter
and a `message` string.

```javascript
module.exports = function (expected, actual) {
  return {
    pass: (actual === expected),
    message: `compared ${actual} to ${expected}`
  }
}
```

Assertions are resolved using `source` and `name` accepted in `source://name`
format.

| Source | Resolved To | Example |
| :--- | :---------- | :------ |
| `npm` | An [npm][npm] package with an assertion as the [main exported module][package.json/main] | `npm://@assertions/is-equal` |
| `local` | A Javascript file (on the runner's filesystem) that exports an assertion as default | `local://is-equal` |

#### `npm`

A collection of assertions is available via npm within the
[`@assertions`][npm/@assertions] organisation on npm.

| Package | Test |
| :------ | :---------- |
| [@assertions/is-equal] | `actual` is equal in value to `expected` |
| [@assertions/is-strictly-equal] | `actual` is equal in value and type to `expected` |
| [@assertions/starts-with] | `actual` starts with `expected` |

### Data Types

| Name | Description |
| :--- | :---------- |
| `string` | A Javascript [`String`][javascript/string] |
| `number` | A Javascript [`Number`][javascript/number] |
| `json` | JavaScript value or object from [`JSON.parse()`][javascript/json/parse] |

## Outputs

| Name | Description | Example |
| :--- | :---------- | :-------|
| `message` | Human readable result of the assertion | `Value is Hello, World!` |
| `pass` | Boolean describing whether the assertion passed | `true` |

## Examples

### SemVer Aliases Are Prefixed

[pr-mpt/actions-semver-aliases] generates aliases for a Semantic Version with an
optional prefix, here we test that this prefix is applied to each alias.

```yaml
jobs:
  test-aliases-are-prefixed:
    runs-on: ubuntu-latest
    steps:
      - name: Generate SemVer aliases with prefix
        id: prefixed
        uses: pr-mpt/actions-semver-aliases@v1
        with:
          version: "3.14.1"
          prefix: "v"
          major: true
          minor: false
      - name: Assert alias is prefixed
        uses: pr-mpt/actions-assert@v2
        with:
          assertion: npm://@assertions/starts-with
          each: true
          actual: "${{ steps.prefixed.outputs.list }}"
          expected: "v"
```

[javascript/string]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String
[javascript/number]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number
[javascript/json/parse]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse
[pr-mpt/actions-semver-aliases]: https://github.com/pr-mpt/actions-semver-aliases
[npm]: https://npmjs.com
[package.json/main]: https://docs.npmjs.com/cli/v7/configuring-npm/package-json#main
[@assertions/is-equal]: https://npmjs.com/package/@assertions/is-equal
[@assertions/is-strictly-equal]: https://npmjs.com/package/@assertions/is-strictly-equal
[@assertions/starts-with]: https://npmjs.com/package/@assertions/starts-with
[npm/@assertions]: https://www.npmjs.com/org/assertions
[workflows/workspace]: https://docs.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions#github-context
