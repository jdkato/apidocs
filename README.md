# APIDocs

[Vale](https://vale.sh) styles for the prose in an
[OpenAPI](https://spec.openapis.org/oas/v3.1.0) document: the summary and
description of each operation, the descriptions of parameters, schemas, and
their properties, the API's title and description, and its tags. Everything
else in the file, the paths, keys, examples, and media types, is left alone.

Works on OpenAPI 3.0 and 3.1 documents written in YAML or JSON, including a
3.1 document's `webhooks`. A description is read as Markdown, as the
specification says it is.

## Install

Requires Vale 3.21.0 or later.

```ini
StylesPath = styles
Packages = APIDocs

[openapi.{yaml,yml,json}]
BasedOnStyles = APIDocs
View = APIDocs
```

```console
$ vale sync
```

`APIDocs` is in the [package library](https://vale.sh/explorer), so the name
is enough. To pin a version, give its release URL instead:
`https://github.com/jdkato/apidocs/releases/download/<version>/APIDocs.zip`.

The package ships a [View](https://docs.vale.sh/topics/views) named
`APIDocs`, which is what reaches inside the document. Name your OpenAPI files
in the section rather than every `*.yaml`: the View reads any file it is
given as OpenAPI, so a workflow or a Helm chart under it is linted for
nothing.

`APIDocs` is the core style and is always on. `Imperative` and `Booleans` are
opinions you can take or leave.

| Style | What it checks |
| ----- | -------------- |
| `APIDocs` | Summaries, descriptions, parameters, properties, and terms. See below. |
| `Imperative` | Summaries written as commands, "Get a pet", in place of the third person, "Gets a pet". |
| `Booleans` | A boolean's description starting with "Whether": `Whether the pet is vaccinated.` in place of `If true, ...` or `Indicates whether ...`. |

### Core rules

| Group | Rules | Examples flagged |
| ----- | ----- | ---------------- |
| Summaries | `ThirdPerson`, `SummaryVerb`, `SummaryPeriod`, `SummaryLength` | `Get a pet`, `The pet with this ID`, `List pets.`, a 16-word summary |
| Descriptions | `DescriptionPeriod`, `Restated` | `Return every pet` with no period, a description that repeats its summary |
| Either | `Filler` | `Endpoint for listing pets`, `API to add a pet`, `This endpoint returns ...` |
| Fields | `Capital`, `FieldName` | `maximum number of pets`, `name: The name.` |
| Terms | `Terms`, `Stutter` | `Id`, `url`, `Json`, `http`, `Returns returns` |

Every rule file starts with a comment explaining what it checks, and most
link to the guidance they come from. The alerts land in the source, at the
line and column of the word in the YAML or JSON, inside a block scalar too.

## Configuration

Write your summaries as commands:

```ini
[openapi.{yaml,yml,json}]
BasedOnStyles = APIDocs, Imperative
View = APIDocs
APIDocs.ThirdPerson = NO
```

Limits are parameters, and any rule can be turned off:

```ini
APIDocs.SummaryLength[max] = 8
APIDocs.Filler = NO
```

The View names each kind of prose as a scope, so a rule of your own can
target one of them:

| Scope | What it holds |
| ----- | ------------- |
| `apititle`, `apidescription` | `info.title` and `info.description` |
| `tagdescription` | Each tag's description |
| `opsummary`, `opdescription` | Each operation's summary and description |
| `param` | Each parameter's description, wherever it is declared |
| `schema` | Each schema's description under `components.schemas` |
| `property` | Each property's description, at any depth |
| `flag` | The description of a boolean parameter or property |
| `restated` | An operation description that only repeats its summary |
| `fieldname` | A parameter or property description that only repeats its name |

No description is in two scopes: a boolean field's is in `flag` and not in
`param` or `property`, so a rule that covers fields lists all three.

## What it doesn't check

Response, request body, header, and server descriptions are not read; they
are prose too, and a rule of your own can reach them through a View of your
own. A description pulled in by `$ref` is checked where it is written, not
where it is used. A boolean is one whose `type` is `boolean`; a 3.1
`type: [boolean, "null"]` is read as any other field, and so is a boolean
with an `enum`, which is a constant rather than a yes-or-no.

Two identical descriptions in different kinds of field, a parameter and a
property both described as "The pet's name.", can trade places in the
report: each alert is right about the text and may name the other's line.

## Sources

- [OpenAPI Specification 3.1.0](https://spec.openapis.org/oas/v3.1.0#operation-object):
  a summary is "a short summary of what the operation does", and a
  description "a verbose explanation". `SummaryLength`, `Restated`.
- [Google developer documentation style guide, API reference code comments](https://developers.google.com/style/api-reference-comments):
  start a method's description with a verb, don't say "this class does",
  capitalize a parameter's description. `SummaryVerb`, `Filler`, `Capital`.
- [Google AIP-192, Documentation](https://google.aip.dev/192#style): the
  first sentence is in the third-person present tense. `ThirdPerson`.
- [Microsoft Writing Style Guide, Reference documentation](https://learn.microsoft.com/en-us/style-guide/developer-content/reference-documentation#elements-of-a-reference-article):
  describe an element in a sentence or two, and "don't just repeat the words
  in the parameter name". `DescriptionPeriod`, `FieldName`.
- [Google word list, ID](https://developers.google.com/style/word-list#id)
  and the [Red Hat supplementary style guide, URL](https://redhat-documentation.github.io/supplementary-style-guide/#url).
  `Terms`.

`SummaryPeriod`, `Stutter`, `Imperative`, and `Booleans` are this package's
conventions. `Booleans` is opt-in because Google's guide describes a boolean
as "True if ...; false otherwise" instead of "Whether ...".

## Development

`./test.sh` runs each rule's own cases with `vale test --coverage`, then
checks `fixtures/openapi.yaml` and `fixtures/openapi.json` against the golden
files in `testdata/` and requires their rewrites in `fixtures/clean/` to pass
clean. `./test.sh -u` rewrites the golden files. The cases need Vale 3.24.0
or later.
