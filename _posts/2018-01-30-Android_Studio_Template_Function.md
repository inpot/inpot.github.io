# Android Studio Template Function

### string *activityToLayout*(string)

This function converts an activity class-like identifer string, such as `FooActivity`, to a corresponding resource-friendly identifier string, such as `activity_foo`.

#### Arguments

- `activityClass`

  The activity class name, e.g. `FooActivity` to reformat.

#### See also

[`layoutToActivity`](#toc_layouttoactivity)

### string *camelCaseToUnderscore*(string)

This function converts a camel-case identifer string, such as `FooBar`, to its corresponding underscore-separated identifier string, such as `foo_bar`.

#### Arguments

- `camelStr`

  The camel-case string, e.g. `FooBar` to convert to an underscore-delimited string.

#### See also

[`underscoreToCamelCase`](#toc_underscoretocamelcase)

### string *escapeXmlAttribute*(string)

This function escapes a string, such as `Android's` such that it can be used as an XML attribute value: `Android&apos;s`. In particular, it will escape ', ", < and &.

#### Arguments

- `str`

  The string to be escaped.

#### See also

[`escapeXmlText`](#toc_escapexmltext)

[`escapeXmlString`](#toc_escapexmlstring)

### string *escapeXmlText*(string)

This function escapes a string, such as `A & B's` such that it can be used as XML text. This means it will escape < and >, but unlike [`escapeXmlAttribute`](#toc_escapexmlattribute) it will **not** escape ' and ". In the preceeding example, it will escape the string to `A &amp; B\s`. Note that if you plan to use the XML text as the value for a <string> resource value, you should consider using [`escapeXmlString`](#toc_escapexmlstring) instead, since it performs additional escapes necessary for string resources.

#### Arguments

- `str`

  The string to escape to proper XML text.

#### See also

[`escapeXmlAttribute`](#toc_escapexmlattribute)

[`escapeXmlString`](#toc_escapexmlstring)

### string *escapeXmlString*(string)

This function escapes a string, such as `A & B's` such that it is suitable to be inserted in a string resource file as XML text, such as `A &amp; B\s`. In addition to escaping XML characters like < and &, it also performs additional Android specific escapes, such as escaping apostrophes with a backslash, and so on.

#### Arguments

- `str`

  The string, e.g. `Activity's Title` to escape to a proper resource XML value.

#### See also

[`escapeXmlAttribute`](#toc_escapexmlattribute)

[`escapeXmlText`](#toc_escapexmltext)

### string *extractLetters*(string)

This function extracts all the letters from a string, effectively removing any punctuation and whitespace characters.

#### Arguments

- `str`

  The string to extract letters from

### string *classToResource*(string)

This function converts an Android class name, such as `FooActivity` or `FooFragment`, to a corresponding resource-friendly identifier string, such as `foo`, stripping the 'Activity' or 'Fragment' suffix. Currently stripped suffixes are listed below.

- Activity
- Fragment
- Provider
- Service

#### Arguments

- `className`

  The class name, e.g. `FooActivity` to reformat as an underscore-delimited string with suffixes removed.

#### See also

[`activityToLayout`](#toc_activitytolayout)

### string *layoutToActivity*(string)

This function converts a resource-friendly identifer string, such as `activity_foo`, to a corresponding Java class-friendly identifier string, such as `FooActivity`.

#### Arguments

- `resourceName`

  The resource name, e.g. `activity_foo` to reformat.

#### See also

[`activityToLayout`](#toc_activitytolayout)

### string *slashedPackageName*(string)

This function converts a full Java package name to its corresponding directory path. For example, if the given argument is `com.example.foo`, the return value will be `com/example/foo`.

#### Arguments

- `packageName`

  The package name to reformat, e.g. `com.example.foo`.

### string *underscoreToCamelCase*(string)

This function converts an underscore-delimited string, such as `foo_bar`, to its corresponding camel-case string, such as `FooBar`.

#### Arguments

- `underStr`

  The underscore-delimited string, e.g. `foo_bar` to convert to a camel-case string.

#### See also

[`camelCaseToUnderscore`](#toc_camelcasetounderscore)