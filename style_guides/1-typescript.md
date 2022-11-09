# TypeScript Style Guide
This is an overview of our style guide for TypeScript files on the frontend.

The nice thing is, we can provide a `tslint` config file to be used in all projects and that will handle linting the TS files correctly.

## Linting
We use `tslint` to lint our TypeScript files.

However, due to the nature of `npm` and that our projects will end up having different node modules, we will not currently be creating a `tslint` docker image like we did for python.

## Rules
Below are all the rules we are using for TypeScript linting

### Quotes
Only single (`'`) quotes are to be used to denote strings in our TypeScript files.

### Semi Colons
We have decided to forego the use of semi colons in TypeScript.

These will be added to the transpiled JavaScript, however.

### Indentation
Indentation in TypeScript is to be 2 spaces.

### Taking Style Rules from Python 3 Style Guide
When working in Typescript, you should also follow any rules in the Python 3 style guide that are applicable, for example [multi-line collections and functions](https://gitlab.cloudcix.com/Wiki/DeveloperManual/wikis/py3_style_guide#multi-line-functions-lists-dicts-etc)

## Config File
Here is the `tslint.json` config file that we are currently using in all of the frontend projects currently using TypeScript;

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
