# Caerbannog-Lang

A simple Turing Complet language. The goal is to provide an extensible language with a few primitives and function plugability.

I this repo, we shall provide the basic for the AST, a simples frontend for the compiler (it validates said AST), an abstract machine model of execution and some reference implementations of tree-walking interpreters (interpreted in host languages), including but not restricted to:

- Java
  - pure Java
  - GraalVM Truffle
- TypeScript

Caerbannog plugins are written in the host language.

## Types

Objects can have some limited types. These atomic types:

- null
- int
- string
- boolean
- foreign

And these composite types:

- list (of valid types)
- map (string -> valid type)

"foreign" can not be created by Caerbannog, but Caerbannog functions can receive and pass it on. Plugins can create "foreign" objects and also do useful work of it.

## Functions

Functions only have named parameters. All parameters must be specified in a function call, or else it is an invalid call. All functions must declare its arguments names.

Functions can be Caerbannog native functions or plugin functions. Plugin functions can only be declared in top-level.

### Function call and variable scope

A function can only access arguments, global values, and local variables previously defined.

> Capture values are out of scope for this version of Caerbannog.

Accessing a variable name not defined as a global variable, a previously defined loal variable or an argument is considered illegal. This can be checked statically.

All parameters must be explicitly bound in a function call, or else it is an invalid call. Binding unnecessary parameters.

### Return value

All functions return a value. If a function does not return explicitly, it will return the last value computed. If it cannot be returned (ie, the last value computed was a function declaration), it should return null instead.

### Plugins

All expected plugins must be declared in a Caerbannog top level program. Plugins are registered before invoking a Caerbonnag program. One can register more plugins than it is expected, unused plugins is not an error.

Plugins are written in the host language. The SDK should allow for the plugin to return an object of the types known in [Types](#types), or else it will be considered as a "foreign" object. A way for returning a value may be through a callback, depending on SDK discretion.

Global values are not passed to plugins, except as named parameters passed explicitly.

## Object values

All objects are immutable. One can at most shadow another previously declared value.

## AST

The AST is defined as a JSON compatible notation. Bellow we have a hello world, given that the plugin `print` echoes the argument `arg`

```json
{
  "version": "0.0.1 ",
  "toplevel": [
    {
      "type": "let-plugin",
      "name": "print",
      "args": [ "s" ]
    },
    {
      "type": "let-func",
      "name": "hello",
      "args": [ "name" ],
      "body": [
        {
          "type": "let",
          "name": "who",
          "value": {
            "type": "if",
            "cond": {
              "type": "un-op",
              "operator": "ISNULL",
              "value": {
                "type": "var",
                "name": "name"
              }
            },
            "when_true": [
              {
                "type": "value",
                "objtype": "string",
                "value": "world"
              }
            ],
            "when_false": [
              {
                "type": "var",
                "name": "name"
              }
            ]
          }
        },
        {
          "type": "let",
          "name": "phrase",
          "value": {
            "type": "bin-op",
            "operator": "CONCAT",
            "lhs": {
                "type": "value",
                "objtype": "string",
                "value": "hello, "
            },
            "rhs": {
              "type": "var",
              "name": "who
            }
          }
        },
        {
          "type": "call",
          "function": "print",
          "params": {
            "s": {
              "type": "var",
              "name": "phrase"
            }
          }
        },
        {
          "type": "var",
          "name": "phrase"
        },
        {
          "type": "return"
        }
      ]
    },
    {
      "type": "entrypoint",
      "body": [
        {
          "type": "call",
          "function": "hello",
          "params": {
            "name": {
              "type": "value",
              "objtype": "null",
              "value": null
            }
          }
        },
        {
          "type": "call",
          "function": "hello",
          "params": {
            "name": {
              "type": "value",
              "objtype": "string",
              "value": "jeff"
            }
          }
        }
      ]
    }
  ]
}
```

Considering `print` plugin defined as following:

```js
print: {
  "args": [ "s" ],
  "call": (args, cb) => {
    console.log(args.s)
    cb.accept(s)
  }
}
```
