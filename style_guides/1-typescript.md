# TypeScript Style Guide
This is an overview of our style guide for frontend TypeScript files.

The nice thing is, we can provide a `tslint` config file to be used in all projects and that will handle the linting of TS files correctly.

## Linting
We use `tslint` to lint our TypeScript files.

However, due to the nature of `npm` and because our projects will end up having different node modules, we will not currently be creating a `tslint` Docker image like we did for Python.

## Rules
Below are all the rules we are using for TypeScript linting

### Quotes
Only single (`'`) quotes are used to denote strings in our TypeScript files.

### Semi Colons
We have decided to forego the use of semi colons in TypeScript.

These will be added to the transpiled JavaScript, however.

### Indentation
TypeScript indentation of 2 spaces is used.

### Taking Style Rules from Python 3 Style Guide
When working in Typescript, you should also follow any rules in the Python 3 style guide that are applicable, for example [multi-line collections and functions](https://github.com/CloudCIX/developers_manual/blob/main/style_guides/application_framework/style/1-python3.md#multi-line-functions--lists--dicts--etc)

### Debug consol log statements
Without the comment before the console.log statement, the logs will not be outputted during testing.

```typescript
// eslint-disable-next-line no-console
console.log("Debug messages");
```

## Config File
Here is the `tslint.json` config file that we are use in all frontend projects, using TypeScript;

```json
{
    "defaultSeverity": "error",
    "extends": ["tslint:recommended"],
    "rules": {
        "quotemark": [true, "single"],
        "comment-format": false,
        "semicolon": [true, "never"],
        "ordered-imports": false,
        "object-literal-sort-keys": false,
        "object-literal-key-quotes": false,
        "interface-name": false,
        "indent": [true, "spaces", 2],
        "arrow-parens": false,
        "max-classes-per-file": false,
        "no-console": false,
        "eofline": false,
        "member-ordering": false,
        "no-string-literal": false,
        "only-arrow-functions": false,
        "align": false,
        "member-access": false,
        "trailing-comma": false,
        "no-var-requires": false,
        "max-line-length": {
            "severity": "warning",
            "options": [170]
        },
        "radix": false,
        "no-empty": false,
        "prefer-const": {
            "severity": "warning"
        },
        "curly": [true, "ignore-same-line"],
        "whitespace": [
            true,
            "check-decl",
            "check-operator",
            "check-module",
            "check-rest-spread",
            "check-type",
            "check-typecast",
            "check-type-operator",
            "check-preblock",
            "check-branch"
        ]
    }
}
```

You can simply copy paste this into the typescript directory as `tslint.conf`.


