# webwire interface description language

The interface description language of webwire is inspired by the
[Rust](https://www.rust-lang.org/) programming language.

The syntax is specified using [Extended Backus-Naur Form
(EBNF)](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_Form):

```
|   alternation
()  grouping
[]  option (zero or one time)
{}  repetition (any number of times)
```

## Lexical elements

```
LETTER = "A" … "Z" | "a" ... "z"
DIGIT_DEC = "0" … "9"
DIGIT_HEX = DIGIT_DEC | "A" … "F" | "a" … "f"
```

## Identifier

Identifiers must start with a letter. Subsequent characters may also
include digits and the underscore `"_"` character.

```
identifier = LETTER { LETTER | DIGIT_DEC | "_" }
```

## Values

### Boolean

Booleans are either `true` or `false`.

```
boolean = "true" | "false"
```

### Integer

Integers support both decimal and hexadecimal format.

```
integer_dec = [ "+" | "-" ] DIGIT_DEC { DIGIT_DEC }
integer_hex = [ "+" | "-" ] "0" ("X" | "x") DIGIT_HEX { DIGIT_HEX }
integer = integer_dec | integer_hex
```

Examples:

- `57005`
- `+3`
- `-5`
- `0x539`
- `+0xFF`
- `-0x7FFF`

### Float

Floats must contain at least one digit before and after the decimal separator.

```
float = [ "+" | "-" ] DIGIT_DEC { DIGIT_DEC } "." DIGIT_DEC { DIGIT_DEC }
```

Examples:

- `2.56`
- `+5.3338`
- `-0.5`

### String

Strings are quoted using double quotes `'"'` and the backspace character `"\"`
is used to escape special characters.

Supported escapes are:

- `\\` → backspace `"\"`
- `\"` → double quote `'"'`
- `\n` → newline (linefeed)

```
string = '"' { char_escape | /[^\\]/ } '"'
char_escape = "\" ( "\" | '"' | "n" )
```

Examples:

- `"master"`
- `"line1\nline2\nline3"`
- `"backslash: \\"`

### Range

Ranges have an upper and lower bound which are separated via `".."`

```
range = integer ".." integer
```

Examples:

- `0..255`
- `0..0xFF`
- `1..`
- `..50`

## Types

Types can either be named or used as part of an array type or map type. The
named form also supports passing generic parameters.

```
type = (type_named | type_array | type_map) [ type_options ]
type_named = identifier ["<" type { "," type } ">"]
type_array = "[" type "]"
type_map = "{" type ":" type "}"
type_options = "(" [ type_option { "," type_option } [ "," ] ] ")"
type_option = identifier "=" value
```

Examples:

- `FooBar`
- `PaginatedResponse<Bar>`
- `[ Integer ]`
- `[ Integer ] (length=1..16)`
- `{ Integer: String }`
- `String (length=0..50)`
- `Integer (range=-0x80..0x7F)`

### Builtin types

Builtin types are reserved type names and can not be used as names for your
own types. It is however possible to use them as identifiers though this
SHOULD NOT be done on a regular basis.

Builtin types are:

- `Boolean`
- `Integer`
- `Float`
- `String`
- `Date`
- `Time`
- `DateTime`
- `UUID`
- `Nullable<T>`

`Nullable` is a special type which wraps another type internally. This
is typically used in APIs to clear values. This must not be confused
with [optional fields](#Optional_Fields) of structures.

## Generics

Structures, Enumerations and some builtin types support generics.

```
generics = "<" identifier { "," identifier } ">"
```

## Struct

Structures are a collection of fields of storing complex data
structures.

```
struct = "struct" identifier [ generics ] "{" [ struct_fields ] "}"
struct_fields = struct_field "," struct_field
struct_field = identifier [ "?" ] ":" type
```

Examples:

  - ```
    struct Pet {
        name: String,
        age?: Integer
    }
  - ```
    struct Complex {
        r: Float,
        i: Float
    }
    ```
  - ```
    struct Person {
        first_name: String (length=1..50)
    }`
  - ```
    struct PaginatedResponse<T> {
        results: T,
        page: Integer(range=0..),
        count: Integer(range=0..),
    }
    ```

### Optional fields

By appending a `?` to the field identifier a field is marked as
optional. An optional field is very different from a nullable field.
Optional means that the field can be absent from the structure while
`Nullable` means that a `Null` value can be assigned.

Some programming languages support this quite naturaly. JavaScript
for example differenciates between `undefined` and `null` for object
attributes. For languages that don't support this out of the box
optional fields should be put inside a special wrapper.

## Fieldset

Fieldset is a special kind of structure that does not define
its own fields but uses an existing structure and creates a subset
of it. This feature is mainly for keeping the repetition for
typical CRUD APIs as little as possible.

```
fieldset = "fieldset" identifier "for" identifier "{" struct_fields "}"
fieldset_fields = [ fieldset_field { "," fieldset_field } [ "," ] ]
fieldset_field = identifier [ "?" ]
```

Examples:

 -  ```
    fieldset PersonUpdate for Person {
        id,
        first_name?,
        last_name?
    }
    ```

## Enumerations

Enumerations in `webwire` fulfill two things. They can either be
used as plain enumerations like in most programming languages. An
optional type argument makes it possible to describe tagged unions.

This is especially handly for returning errors which might contain
data which depends on the actual error code.

Enumerations can also extend existing enumerations.

```
enum = "enum" identifier [ generics ] [ enum_extends ] "{" enum_variants "}"
enum_extends = "extends" identifier [ generics ]
enum_variants = [ enum_variant { "," enum_variant } [ "," ] ]
enum_variant = identifier [ "(" type ")" ]
```

Examples:

  - ```
    enum Status {
        Enabled,
        Disabled,
        Error
    }
    ```
  - ```
    enum AuthError {
        Unauthenticated,
        PermissionDenied
    }
    ```
  - ```
    enum GetError extends AuthError {
        DoesNotExist
    }
    ```
  - ```
    enum Notification {
        UserJoined(User),
        UserLeft(User),
        Message(ChatMessage),
    }
    ```

## Namespace

TODO

```
namespace = "namespace" identifier "{" namespace_parts "}"
namespace_parts = { namespace_part }
namespace_part = struct | fieldset | enum | namespace | service
```

## Service

TODO

```
service = ["async" | "sync"] "service" identifier "{" methods "}"
methods = [ method { "," method } [ "," ] ]
method = identifier ":" ( type | "None" ) [ "->" type ]
```
