# Versioning

## Summary

Webwire interfaces use [semantic versioning](https://semver.org/).

Given a schema version number `MAJOR.MINOR.PATCH`, increment the:

1. `MAJOR` version when you make incompatible API changes,
2. `MINOR` version when you add functionality in a backwards compatible manner, and
3. `PATCH` version when you make backwards compatible bug fixes.

Structs, fields, enums and endpoints carry their own version number
`MAJOR.MINOR` which can also include deprecation information. It is not
necessary that structs, enums and endpoints use the same `MAJOR` version
as the schema version.

## Version ranges

In order to deprecate features it is possible to not only define a version
number when a feature was introduced but also when it was removed. Therefore
it is possible to write versions in the form of `LOWER..UPPER` where `LOWER`
signifies when the feature was added and `UPPER` when the feature
was removed.

e.g. `1.0..2.0` means that a given feature was introduced in version `1.0`
of the schema and removed in version `2.0` of the schema.

## Example version progression

### Version 1.0

The `Person` struct has been available since version 1.0:

```webwire
version 1.0;

struct Person {
    name: String,
}

Service {
    hello: Person -> String;
}
```

### Version 1.1

Adding an `age` field to the `Person`:

```webwire
version 1.1;

struct Person {
    name: String,
    age?: Integer (range=0..50),
}

http {
    endpoint hello (Person) -> String;
}
```

**Remember:** When adding optional fields it is fine to just increase the
`MINOR` version. When adding non-optional fields or making optional fields
mandator the `MAJOR` version MUST BE increased.

### Version 2.0

Making `age` required:

```webwire
version 2.0;

struct Person {
    name: String,
    age: Integer (range=0..50),
}

endpoint hello (Person) -> String;
```

### Version 3.0

Replace `name` by separate fields for `first_name` and `last_name`:

```webwire
version 3.0;

struct Person {
    first_name: String,
    last_name: String,
    age: Integer (range=0..)
}

endpoint hello (Person) -> String;
```

### Version 1.1 + 2.0 + 3.0

It is also possible to support multiple major versions in one webwire schema
file by adding versioning information to the structs and endpoints:

```webwire
version 3.0;

struct Person/1.1 {
    name: String,
    age?: Integer (range=0..),
}

struct Person/2.0 {
    name: String,
    age: Integer (range=0..),
}

struct Person/3.0 {
    first_name: String,
    last_name: String,
    age: Integer (range=0..),
}

service Hello {
    hello/1.0: Person/1.0 -> String,
    hello/2.0: Person/2.0 -> String,
    hello/3.0: Person/3.0 -> String
}
```
