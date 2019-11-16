# JDDF Features and Syntax

> Learn how to understand and write your own JDDF schemas.

## Overview

Use this guide to learn how JDDF schemas work, and what they do. Once you
understand how JDDF schemas work, you can:

* [Learn how to integrate JDDF in your code][integrate], or
* [Try JDDF in your browser][playground].

[integrate]: /integrate.html
[playground]: https://play.jddf.io

## Validation

At its core, JDDF is a way to describe the shape of JSON data. With JDDF, you
describe your data using *schemas*. Schemas are just JSON objects, but you can
also write your schemas in YAML and then convert that YAML into JSON.

Here's an example schema. It accepts strings, and nothing else:

{{< example-schema >}}
```json
{ "type": "string" }
```
{{< /example-schema >}}

This accepts any string:

{{< example-ok >}}
```json
"hello, world!"
```
{{< /example-ok >}}

And rejects anything that's not a string:

{{< example-bad >}}
```json
null
```
{{< /example-bad >}}

{{< example-bad >}}
```json
123
```
{{< /example-bad >}}

{{< example-bad >}}
```json
["hello", "world", "!"]
```
{{< /example-bad >}}

In JDDF, some schema keywords take other schemas as values. So for example, we
can define a schema that accepts *arrays* of strings by doing:

{{< example-schema >}}
```json
{ "elements": { "type": "string" }}
```
{{< /example-schema >}}

That schema accepts any array of strings, including the empty array:

{{< example-ok >}}
```json
["hello", "world", "!"]
```
{{< /example-ok >}}

{{< example-ok >}}
```json
[]
```
{{< /example-ok >}}

But it rejects anything that's not an array. It also rejects any array
containing anything that's not a string:

{{< example-bad >}}
```json
null
```
{{< /example-bad >}}

{{< example-bad >}}
```json
"hello, world!"
```
{{< /example-bad >}}

{{< example-bad >}}
```json
["hello", "world", null, "!"]
```
{{< /example-bad >}}

Those are the main patterns you'll see in JDDF: some keywords, like `type`,
define very basic sort of data. Other keywords, like `elements`, build a more
complicated schema using a simpler one.

With that idea in mind, let's now dig into all of JDDF's features.

### Empty Schemas

Empty schemas are sort of trivial. They will accept *any* input. Empty schemas
will never reject anything. For example, here is an empty schema:

{{< example-schema >}}
```json
{}
```
{{< /example-schema >}}

It accepts any JSON input:

{{< example-ok >}}
```json
null
```
{{< /example-ok >}}

{{< example-ok >}}
```json
{ "foo": "bar", "asdf": [1, 2, 3]}
```
{{< /example-ok >}}

### "Type" Schemas

"Type" schemas use the `type` keyword to accept one of the primitive JSON data
types. JDDF supports these primitive types:

* Booleans
* Numbers / Integers
* Strings
* Timestamps encoded as strings

#### Booleans

Booleans are the simplest of these. You can validate for a boolean by doing:

{{< example-schema >}}
```json
{ "type": "boolean" }
```
{{< /example-schema >}}

This will accept only:

{{< example-ok >}}
```json
true
```
{{< /example-ok >}}

{{< example-ok >}}
```json
false
```
{{< /example-ok >}}

Everything else is rejected:

{{< example-bad >}}
```json
null
```
{{< /example-bad >}}

{{< example-bad >}}
```json
"true"
```
{{< /example-bad >}}

#### Numbers

If you want to validate for *any* number, whether or not it's an integer, then
use `float32` or `float64`.

> `float32` and `float64` do the same thing when it comes to validation. The
  only difference is that code generators will usually generate `float` from
  `float32`, and `double` from `float64`.

For example, these two schemas accept and reject exactly the same values:

{{< example-schema >}}
```json
{ "type": "float32" }
```
{{< /example-schema >}}

{{< example-schema >}}
```json
{ "type": "float64" }
```
{{< /example-schema >}}

They both accept:

{{< example-ok >}}
```json
0
```
{{< /example-ok >}}

{{< example-ok >}}
```json
3.14
```
{{< /example-ok >}}

{{< example-ok >}}
```json
-123e50
```
{{< /example-ok >}}

And they both reject:

{{< example-bad >}}
```json
null
```
{{< /example-bad >}}

{{< example-bad >}}
```json
"3.14"
```
{{< /example-bad >}}

#### Integers

If you're interested in validating for *integers* in particular, then you can
use one of the following types:

{{< mdc-data-table >}}
| Type     | Minimum Value Accepted | Maximum Value Accepted |
| -------- | ---------------------- | ---------------------- |
| `int8`   | -2^7                   | 2^7 - 1                |
| `uint8`  | 0                      | 2^8 - 1                |
| `int16`  | -2^15                  | 2^15 - 1               |
| `uint16` | 0                      | 2^16 - 1               |
| `int32`  | -2^31                  | 2^31 - 1               |
| `uint32` | 0                      | 2^32 - 1               |
{{< /mdc-data-table >}}

These types all work in basically the same way, just with a different range of
values. So let's just use this schema as an example:

{{< example-schema >}}
```json
{ "type": "uint8" }
```
{{< /example-schema >}}

It accepts the following values, because they're integers and they're in the
appropriate range for `uint8`, which goes from 0 to `2^8 - 1` (which is 255):

{{< example-ok >}}
```json
0
```
{{< /example-ok >}}

{{< example-ok >}}
```json
123
```
{{< /example-ok >}}

{{< example-ok >}}
```json
255
```
{{< /example-ok >}}

The following values are rejected because they're not numbers:

{{< example-bad >}}
```json
null
```
{{< /example-bad >}}

{{< example-bad >}}
```json
"0"
```
{{< /example-bad >}}

Also rejected are JSON inputs which don't represent integers:

{{< example-bad >}}
```json
3.14
```
{{< /example-bad >}}

And the following values are rejected because they're out of range:

{{< example-bad >}}
```json
-1
```
{{< /example-bad >}}

{{< example-bad >}}
```json
256
```
{{< /example-bad >}}

A more unexpected property of the integer schemas is that they're defined in
terms of the *value*, not the *syntax*, of the input. So all of the following
schemas, which are all equal to 10, are accepted:

{{< example-ok >}}
```json
10
```
{{< /example-ok >}}

{{< example-ok >}}
```json
10.0
```
{{< /example-ok >}}

{{< example-ok >}}
```json
1.0e1
```
{{< /example-ok >}}

#### Strings

The `string` type accepts any JSON string, and rejects everything else:

{{< example-schema >}}
```json
{ "type": "string" }
```
{{< /example-schema >}}

Accepts:

{{< example-ok >}}
```json
""
```
{{< /example-ok >}}

{{< example-ok >}}
```json
"hello, world!"
```
{{< /example-ok >}}

But rejects:

{{< example-bad >}}
```json
null
```
{{< /example-bad >}}

{{< example-bad >}}
```json
["hello", "world", "!"]
```
{{< /example-bad >}}

#### Timestamps

Beyond strings, a special case is made for timestamps. One of the most common
timestamp formats used in JSON-based APIs is the [RFC 3339][rfc3339] format,
which looks like this:

[rfc3339]: https://tools.ietf.org/html/rfc3339

```text
1985-04-12T23:20:50.52Z
```

Because this format is so common, JDDF has special support for it. The
`timestamp` type accepts any JSON string encoding an RFC 3339 timestamp, and
rejects everything else:

{{< example-schema >}}
```json
{ "type": "timestamp" }
```
{{< /example-schema >}}

Accepts:

{{< example-ok >}}
```json
"1985-04-12T23:20:50.52Z"
```
{{< /example-ok >}}

But rejects all of these, because they're either not strings, or not encoding
RFC 3339 timestamps:

{{< example-bad >}}
```json
null
```
{{< /example-bad >}}

{{< example-bad >}}
```json
0
```
{{< /example-bad >}}

{{< example-bad >}}
```json
"July 4, 1776"
```
{{< /example-bad >}}

### "Enum" Schemas

"Enum" schemas use the `enum` keyword to validate that a value is one of a
predefined list of strings. For example, this schema:

{{< example-schema >}}
```json
{ "enum": ["STARTING", "RUNNING", "FINISHED"] }
```
{{< /example-schema >}}

Would accept only the following values:

{{< example-ok >}}
```json
"STARTING"
```
{{< /example-ok >}}

{{< example-ok >}}
```json
"RUNNING"
```
{{< /example-ok >}}

{{< example-ok >}}
```json
"FINISHED"
```
{{< /example-ok >}}

Everything else would be rejected, including different capitalizations, or using
numbers instead of strings:

{{< example-bad >}}
```json
null
```
{{< /example-bad >}}

{{< example-bad >}}
```json
"starting"
```
{{< /example-bad >}}

{{< example-bad >}}
```json
1
```
{{< /example-bad >}}

"Enum" schemas exclusively work with arrays of strings. These are not valid
schemas:

```json
{ "enum": [0, 1, 2] }
```

```json
{ "enum": ["FOO", 1, "BAR"] }
```

### "Elements" Schemas

"Elements" schemas describe arrays. They validate that:

1. The input is a JSON array, and
2. Every element of the array is accepted by the sub-schema you provided as the
   `elements` value.

For example:

{{< example-schema >}}
```json
{ "elements": { "type": "string" }}
```
{{< /example-schema >}}

That schema accepts any array of strings, including the empty array:

{{< example-ok >}}
```json
["hello", "world", "!"]
```
{{< /example-ok >}}

{{< example-ok >}}
```json
[]
```
{{< /example-ok >}}

But it rejects anything that's not an array. It also rejects any array
containing anything that's not a string:

{{< example-bad >}}
```json
null
```
{{< /example-bad >}}

{{< example-bad >}}
```json
"hello, world!"
```
{{< /example-bad >}}

{{< example-bad >}}
```json
["hello", "world", null, "!"]
```
{{< /example-bad >}}

### "Properties" Schemas

"Properties" schemas describe `struct`-like arrays, where you know in advance
what the key names you're interested are, and you know what values they should
have. They validate that:

1. The input is a JSON object, and
2. All of the *required* properties you're interested in (using the `properties`
   keyword) are present, and satisfy their corresponding schemas, and
3. If any of the *optional* properties you're interested in (using the
   `optionalProperties` keyword) are present, that they satisfy their
   corresponding schemas, and
4. That there aren't any properties you didn't mention in `properties` or
   `optionalProperties`, unless you explicitly set `additionalProperties: true`
   on the schema.

#### Required Properties

For example, here's a simple "properties" schema:

{{< example-schema >}}
```json
{ "properties": { "id": { "type": "string" }}}
```
{{< /example-schema >}}

It accepts any object that has an `id`, and the `id`'s value is a string:

{{< example-ok >}}
```json
{ "id": "users/jdoe" }
```
{{< /example-ok >}}

But if the input isn't an object, or `id` is missing, or if `id`'s value isn't a
string, then the input is rejected:

{{< example-bad >}}
```json
null
```
{{< /example-bad >}}

{{< example-bad >}}
```json
{ "ID": "users/jdoe" }
```
{{< /example-bad >}}

{{< example-bad >}}
```json
{ "id": 123 }
```
{{< /example-bad >}}

#### Optional Properties

Here's an example that uses `properties` in combination with
`optionalProperties`:

{{< example-schema >}}
```json
{
  "properties": { "id": { "type": "string" }},
  "optionalProperties": { "age": { "type": "uint32" }},
}
```
{{< /example-schema >}}

It accepts any object that has a string `id`, and optionally a uint32 `age`:

{{< example-ok >}}
```json
{ "id": "users/jdoe" }
```
{{< /example-ok >}}

{{< example-ok >}}
```json
{ "id": "users/jdoe", "age": 42 }
```
{{< /example-ok >}}

But if `age` is present and has the wrong type, that's rejected:

{{< example-bad >}}
```json
{ "id": "users/jdoe", "age": "42" }
```
{{< /example-bad >}}

Here's a more exciting example with multiple required and optional properties.
It describes a paginated list of users:

{{< example-schema >}}
```json
{
  "properties": {
    "users": {
      "elements": {
        "properties": {
          "id": { "type": "string" },
          "name": { "type": "string" }
        },
        "optionalProperties": {
          "age": { "type": "uint32" },
          "favorite_color": { "type": "string" }
        }
      }
    },
  },
  "optionalProperties": {
    "next_page_token": { "type": "string" }
  }
}
```
{{< /example-schema >}}

Here's a valid example that uses all of the properties, both optional and
required:

{{< example-ok >}}
```json
{
  "users": [
    {
      "id": "users/jdoe",
      "name": "John Doe",
      "age": 42,
      "favorite_color": "blue"
    }
  ],
  "next_page_token": "bG92ZSBvbmUgYW5vdGhlcgo="
}
```
{{< /example-ok >}}

Here's another valid example, but with some of the optional properties left out:

{{< example-ok >}}
```json
{
  "users": [
    {
      "id": "users/jdoe",
      "name": "John Doe",
      "age": 42
    }
  ]
}
```
{{< /example-ok >}}

But if you were to remove one of the required properties, that's invalid:

{{< example-bad >}}
```json
{
  "users": [
    {
      "id": "users/jdoe",
      "age": 42
    }
  ]
}
```
{{< /example-bad >}}

#### Additional Properties

Oftentimes, you want to allow for properties in an object that you didn't
explicitly mention. This is especially useful if you're retroactively specifying
an existing message format, and you don't have all of the properties specified
yet.

By default, JDDF does *not* allow for additional properties. For example, this
schema:

{{< example-schema >}}
```json
{
  "properties": {
    "a": { "type": "string" },
    "b": { "type": "string" }
  },
  "optionalProperties": {
    "c": { "type": "string" },
    "d": { "type": "string" }
  },
}
```
{{< /example-schema >}}

Accepts this input:

{{< example-ok >}}
```json
{ "a": "a", "b": "b", "c": "c", "d": "d" }
```
{{< /example-ok >}}

But not this input, because `e` is not one of the properties described in
`properties` or `optionalProperties`:

{{< example-bad >}}
```json
{ "a": "a", "b": "b", "c": "c", "d": "d", "e": "e" }
```
{{< /example-bad >}}

If you wanted additional properties like `e` to be allowed, then you can set
`additionalProperties: true` on your schema:

{{< example-schema >}}
```json
{
  "properties": {
    "a": { "type": "string" },
    "b": { "type": "string" }
  },
  "optionalProperties": {
    "c": { "type": "string" },
    "d": { "type": "string" }
  },
  "additionalProperties": true
}
```
{{< /example-schema >}}

With that change, now both inputs are accepted:

{{< example-ok >}}
```json
{ "a": "a", "b": "b", "c": "c", "d": "d" }
```
{{< /example-ok >}}

{{< example-ok >}}
```json
{ "a": "a", "b": "b", "c": "c", "d": "d", "e": "e" }
```
{{< /example-ok >}}

### "Values" Schemas

"Properties" schemas are about describing `struct`-like JSON objects, where you
know the keys and values in advance. But there's another we commonly use JSON
objects: as dictionaries. JDDF supports dictionary-like JSON objects using the
`values` keyword.

The `values` keyword is about describing JSON objects where you don't know the
keys in advance, but you do know the type of all the values.

Here's an example of a "values" schema:

{{< example-schema >}}
```json
{ "values": { "type": "uint8" }}
```
{{< /example-schema >}}

That schema accepts all of the following inputs:

{{< example-ok >}}
```json
{}
```
{{< /example-ok >}}

{{< example-ok >}}
```json
{ "a": 1 }
```
{{< /example-ok >}}

{{< example-ok >}}
```json
{ "a": 1, "b": 5, "zzzzz": 22 }
```
{{< /example-ok >}}

But not any of these inputs:

{{< example-bad >}}
```json
null
```
{{< /example-bad >}}

{{< example-bad >}}
```json
{ "a": "1" }
```
{{< /example-bad >}}

{{< example-bad >}}
```json
{ "a": 1, "b": "5", "zzzzz": 22 }
```
{{< /example-bad >}}

### "Discriminator" Schemas

There is one last, but very important, way that we use JSON objects in practice:
as *discriminated unions*, also called *tagged unions*. In discrimniated unions,
you have a special "tag" property whose value tells you what kind of message
you're dealing with. Afer reading the "tag", you can then proceed to read the
rest of the message.

Because discriminated unions are so common, JDDF supports them directly using
the `discriminator` keyword. "Discriminator" schemas validate that:

1. The input is an object,
2. The input has a certain `tag` property,
3. Then, based on the `tag` value, the input is validated against a
   corresponding `mapping`. `mapping` is an object whose keys are `tag` values,
   and whose values are schemas.
4. If the `tag` value isn't one of the keys in `mapping`, then the input is
   invalid.

Here's an example. Let's say you're dealing with messages that always have an
`event_type` property:

* If `event_type` is `user_created`, then there should be a `user` property, an
  object.
* If `event_type` is `user_deleted`, then there should be a `user` property, but
  this time it should be a string. It might optionally also have a
  `delete_reason`.

Here's how you can express that with JDDF:

{{< example-schema >}}
```json
{
  "discriminator": {
    "tag": "event_type",
    "mapping": {
      "user_created": {
        "properties": {
          "user": {
            "properties": {
              "id": { "type": "string" },
              "name": { "type": "string" }
            }
          }
        }
      },
      "user_deleted": {
        "properties": {
          "user": { "type": "string" }
        },
        "optionalProperties": {
          "delete_reason": { "type": "string" }
        }
      }
    }
  }
}
```
{{< /example-schema >}}

That schema accepts:

{{< example-ok >}}
```json
{ "event_type": "user_created", "user": { "id": "jdoe", "name": "John" }}
```
{{< /example-ok >}}

{{< example-ok >}}
```json
{ "event_type": "user_deleted", "user": "jdoe" }
```
{{< /example-ok >}}

{{< example-ok >}}
```json
{ "event_type": "user_deleted", "user": "jdoe", "delete_reason": "gdpr" }
```
{{< /example-ok >}}

But that schema would reject any non-objects, or anything missing `event_type`,
or anything where `event_type` isn't either `user_created` or `user_deleted`:

{{< example-bad >}}
```json
null
```
{{< /example-bad >}}

{{< example-bad >}}
```json
{ "EVENT_TYPE": "user_deleted", "user": "jdoe" }
```
{{< /example-bad >}}

{{< example-bad >}}
```json
{ "event_type": "user_was_deleted", "user": "jdoe" }
```
{{< /example-bad >}}

## Avoiding Repetition with Definitions

As you begin to write larger JDDF schemas, you'll probably find yourself
re-using the same sub-schemas over and over. For example, let's say you have a
system that accepts requests for directions: both `start` and `end` are objects
with properties `lat` and `lng` (latitude and longitude).

In JDDF, you could describe that like so:

{{< example-schema >}}
```json
{
  "properties": {
    "start": {
      "properties": {
        "lat": { "type": "string" },
        "lng": { "type": "string" }
      }
    },
    "end": {
      "properties": {
        "lat": { "type": "string" },
        "lng": { "type": "string" }
      }
    }
  }
}
```
{{< /example-schema >}}

There are a couple downsides to repeating `lat` and `lng` everywhere:

1. It's more verbose,
2. It somewhat hides the insight that `start` and `end` actually have the same
   type,
3. Tools like [`jddf-codegen`](/tooling/code-generation.html) will generate
   different types for `start` and `end`, because it might just be coincidence
   that `start` and `end` are the same right now.

JDDF lets you re-use schemas, and signal to tools like `jddf-codegen` that you
want to give a particular sub-schema a meaningful name, by using the
`definitions` and `ref` keywords.

Here's how you can use `definitions` to make the above schema less repetitive:

{{< example-schema >}}
```json
{
  "definitions": {
    "latlng": {
      "properties": {
        "lat": { "type": "string" },
        "lng": { "type": "string" }
      }
    }
  },
  "properties": {
    "start": { "ref": "latlng" },
    "end": { "ref": "latlng" }
  }
}
```
{{< /example-schema >}}
