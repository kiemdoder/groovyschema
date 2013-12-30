# groovy-schema

Groovy object validation library that tries to emulate the [JSON Schema
specification](http://json-schema.org/latest/json-schema-validation.html).
It is vastly inspired by @tdegrunt's
[implementation](https://github.com/tdegrunt/jsonschema).

It is meant to be used with
[JsonSlurper](http://groovy.codehaus.org/gapi/groovy/json/JsonSlurper.html) when
validating incoming JSON content in REST API implementations.

## Usage

```groovy
def schema = [
  type: 'object',
  required: true,
  properties: [
    honorificPrefix: [enum:['Ms.', 'Mr.', 'Dr.']],
    givenName: [type:'string', required:true],
    additionalName: [type:'string']
    familyName: [type:'string', required:true],
    honorificSuffix: [enum:['Ph.D.', 'Esq.']],
    email: [format:'email', required:true],
  ]
]

def validator = new groovyschema.Validator()
def instance = new groovy.json.JsonSluper().parseText(req.body)

def validationErrors = validator.validate(instance, schema)
```

## Non type-specific validations attributes

### required

Validates that the passed-in instance is not `null`.

### type

Validates that the instance is of the specified type. Possible values are
`string`, `number`, `integer`, `boolean`, `array` (evaluates to `true` for
instances of `List`), `object` (evaluates to `true` for instances of `Map`),
`null` and `any`.

`null` instances that do not specify `null` as their type always evaluate to
`true`.

### enum

Validates that the instance is in one of the given values. Enum values can be of
any type.

## String-specific validations attributes

### pattern

Validates that the string instance matches the passed-in regular expression:

```groovy
def schema = [
  type: 'string',
  required: true,
  pattern: /^fo+$/,
]
```

### format

Same as `pattern` but references predefined regular expressions. Possible values
are `date-time`, `email`, `hostname`, `ipv4`, `ipv6`, and `uri`.

### minLength, maxLength

Validates the length of the string. By default, minimum and maximum lengths are
inclusive. Add the `exclusiveMinimum` or `exclusiveMaximum` attributes to alter
this behaviour.

```groovy
// Valid if string length in (0..30]
def schema = [
  type: 'string',
  minLenght: 0,
  maxLength: 30,
  exclusiveMinimum: true,
]
```

## Number-specific validations attributes

### minimum, maximum

By default, minimum and maximum values are inclusive. Add the
`exclusiveMinimum` or `exclusiveMaximum` attributes to alter this behaviour.

```groovy
// Valid if number in (0..100]
def schema = [
  type: 'number',
  minimum 0,
  maximum 100,
  exclusiveMinimum: true,
]
```

### divisibleBy

Validates the the instance is divisible by the attribute value.

## Map-specific validations attributes

### properties

Validates a schema for each of the instance's entries. For each entry in the
`properties` map, the key specifies a certain attribute found in the instance;
the value specifies a schema for that attribute.

The following schema indicates that its instances must be maps with two entries.
One of those must have a key called `name` and its value must be non-null and a
string. The other key is called `email` and its value must be a non-null string
that matches the predefined `email` regular expression.

```groovy
def schema = [
  type: 'object',
  properties: [
    name: [type:'string', required:true],
    email: [type:'string', format:'email', required:true],
  ]
]
```

### additionalProperties

Validates what additional properties can be part of the instance. Possible
values can be a boolean (`true` by default) a string or list of strings
representing the allowed, optional, additional properties or the schema for all
additional properties.

```groovy
def schema = [
  type: 'object',
  properties: [
    name: [type:'string', required:true],
    email: [type:'string', format:'email', required:true],
  ],
  additionalProperties: ['givenName', 'familyName']
]
```

### patternProperties

Similar to the `properties` attribute, `patternProperties` specifies regular
expressions instead of exact strings for property names. Values for properties
that match more than a single regular expression must conform to all matching
schemas.

```groovy
def schema = [
  type: 'object',
  properties: [
    /^(given|family)Name$/: [type:'string', required:true],
  ]
]
```

### dependencies

Defines dependencies between properties. Dependencies can either be
property-dependencies or schema-dependency.

Property-dependency (e.g. if property `a` is present, so also must be properties
`b` and `c`):

```groovy
def schema = [
  type: 'object',
  dependencies: [
    a: ['b', 'c']
  ]
]
```

Schema-dependency (e.g. if property `a` is present, the instance must also
conform to an additional schema):

```groovy
def schema = [
  type: 'object',
  dependency: [
    a: [
      properties: [
        b: [type:'integer', minimum:0, divisibleBy:2, required:true]
      ]
    ]
  ]
]
```

In this example, valid instances would include `[a:1, b:2]` and `[c:3]`. Invalid
instances could be `[a:1]` or `[a:1, b:1]`.

## List-specific validations attributes

### items

Validates that the items in the instance all conform to a specified schema.
Possible values for the `items` attribute are a single, common, schema for all
items in the list or an ordered list of schemas (one for each item in the list).

All items must be non-null positive integers:

```groovy
def schema = [
  type: 'array',
  items: [type:'integer', minimum:0, required:true]
]
```

The first item must be an email; the second must be en integer. Additional
items are not allowed:

```groovy
def schema = [
  type: 'array',
  items: [
    [type:'string', format:'email'],
    [type:'integer']
  ]
]
```

The first item must be an email. Additional items *are* allowed:

```groovy
def schema = [
  type: 'array',
  additionalItems: true,
  items: [
    [type:'string', format:'email']
  ]
]
```

### minItems, maxItems

By default, minItems and maxItems values are inclusive. Add the
`exclusiveMinimum` or `exclusiveMaximum` attributes to alter this behaviour.

### uniqueItems

Validates that all items are unique. Males a deep comparisons for collections
and uses the "groovy truth" for simple types.

## Multi-schema validations attributes (allOf, anyOf, oneOf, not)

Validates an instance against an array of schemas. `allOf` indicates that the
instance must conform to all given schemas; `not` indicates that the instance
must not conform to any schemas.

Match either strings or integers between 0 and 5:

```groovy
def schema = [
  anyOf: [
    [type:'string', required:true],
    [type:'integer', minimum:0, maximum:5, required:true],
  ]
]
```

## License

MIT License.