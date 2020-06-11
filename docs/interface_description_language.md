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
- `Partial<T>`

## Struct

TODO

```
struct = "struct" identifier "{" [ struct_fields ] "}"
struct_fields = struct_field "," struct_field
struct_field = identifier [ "?" ] ":" type
```
Examples:

- `struct Pet { name: String, age?: Integer }`
- `struct Complex { r: Float, i: Float }`
- `struct Person { first_name: String (length=1..50) }`

## Fieldset

TODO

```
fieldset = "fieldset" identifier "for" identifier "{" [ struct_fields ] "}"
fieldset_fields = struct_field "," struct_field
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

## Enum

TODO

```
enum = "enum" identifier [ generics ] "{" enum_variants "}"
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
    enum Notification {
        UserJoined(User),
        UserLeft(User),
        Message(ChatMessage),
    }
    ```
  - ```
    enum PaginatedResponse<T> {
        results: T,
        page: Integer(range=0..),
        count: Integer(range=0..),
    }
    ```

## Namespace

`TODO`

## Service

`TODO`

## Method

`TODO`
