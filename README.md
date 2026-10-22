# JSON Structure Lab

JSON Structure describes data with explicit types: `int32`, `int64`, `uuid`, and more. Structurize converts schemas into and out of that type system. In this lab, you will import two schemas, export a JSON Structure schema, and check both a schema and the data it describes.

Five tasks. Small files. No services to deploy.

## Setup

You need Git and Python 3.12 or later. Use a terminal in the repository root for every command below.

```sh
git clone https://github.com/clemensv/structurize-lab.git
cd structurize-lab
pip install structurize
python -c "from pathlib import Path; Path('output').mkdir(exist_ok=True)"
structurize --help
```

Installing Structurize also installs its dependencies, including the JSON Structure SDK. No separate dependency installation is needed.

If your system calls Python `python3`, use that name for the Python commands in this lab.

The SDK checks schemas; `structurize validate` checks data instances. These are different questions: is the contract well formed, and does this document satisfy it?

## Your Starting Files

| File | What it contains |
| --- | --- |
| [schemas/address.schema.json](schemas/address.schema.json) | An address described with JSON Schema. |
| [schemas/address.avsc](schemas/address.avsc) | An address described with Avro. It is not an identical contract. |
| [schemas/basic-types.struct.json](schemas/basic-types.struct.json) | A native JSON Structure schema with required fields and explicit scalar types. |
| [data/basic.valid.json](data/basic.valid.json) | A document that satisfies the native schema. |
| [data/basic.invalid.json](data/basic.invalid.json) | A document with an integer outside the `int32` range. |

Generated files go into `output/`, which Git ignores. Keep the starting files unchanged unless a task explicitly asks you to edit one.

## Task 1: Import JSON Schema

Convert the JSON Schema address contract to JSON Structure:

```sh
structurize j2s schemas/address.schema.json --out output/address-from-jsonschema.struct.json
```

Open the source and result side by side. Find `properties`, `required`, `$id`, and `$schema`. Which properties are required? What happened to `dependentRequired`?

You are done when you can locate the address properties and required fields in the generated schema. A successful conversion is not proof that every source constraint survived. Inspect the result.

## Task 2: Import Avro

Convert the Avro address contract:

```sh
structurize a2s schemas/address.avsc --out output/address-from-avro.struct.json
```

Compare the result with Task 1. Avro describes records with `fields`; JSON Structure describes objects with `properties`. Find how the converter represents a field whose Avro type is `["null", "string"]`. Then find the `region` default.

You are done when you can explain one difference between the two address contracts. Nullable and optional are not the same question: one concerns a value, the other concerns whether a property must be present.

## Task 3: Export JSON Structure

Use the supplied native schema so this task does not depend on the previous two outputs. Export it to JSON Schema and Avro:

```sh
structurize s2j schemas/basic-types.struct.json --out output/basic-types.schema.json
structurize s2a schemas/basic-types.struct.json --out output/basic-types.avsc
```

Find `intField`, `longField`, and `uuidField` in both results. How does each target express those types? Compare its required or nullable fields with the source.

You are done when both files exist and you can trace `int32` into each target. Do not expect textually identical schemas after a round trip. Schema languages have different type systems and different constraints.

## Task 4: Check the Schema

A JSON document can parse correctly and still be an invalid schema. Check the native schema with the JSON Structure SDK:

```sh
python -c "import json; from json_structure import SchemaValidator; schema = json.load(open('schemas/basic-types.struct.json', encoding='utf-8')); errors = SchemaValidator().validate(schema); print(errors or 'Schema is valid'); raise SystemExit(bool(errors))"
```

Expect `Schema is valid` and exit code `0`.

Now change `"required": ["stringField", "intField"]` to `"required": "stringField"` in the native schema and run the same command. The required property names must form an array, not a string. Expect a diagnostic and a nonzero exit code. Restore the original array and confirm the check passes again.

You are done when you have seen the schema check succeed, fail, and succeed again.

## Task 5: Check the Data

Validate the valid document against the native schema:

```sh
structurize validate data/basic.valid.json --schema schemas/basic-types.struct.json
```

Expect one valid instance and exit code `0`. Now check the invalid document:

```sh
structurize validate data/basic.invalid.json --schema schemas/basic-types.struct.json
```

Expect an out-of-range diagnostic and exit code `1`. The value `2147483648` is a perfectly good JSON number. It is not a signed 32-bit integer.

Change `intField` in the invalid document to `2147483647` and rerun its validation. It should pass. Then remove `stringField` and rerun it. It should fail because a required property is missing. Restore the original document when you finish.

In PowerShell, inspect `$LASTEXITCODE` immediately after a command. In bash, use `echo $?`. Expected failures are part of the exercise.

You are done when you can distinguish an invalid schema, an invalid field value, and a missing required property.

## Where to Go Next

- [JSON Structure specification](https://json-structure.org/)
- [Structurize and Avrotize documentation](https://github.com/clemensv/avrotize)
- `structurize j2s --help`, `structurize s2j --help`, and `structurize validate --help` for command-specific options.

## Sample Provenance and License

The three schemas are staged unchanged from the [Avrotize test corpus at revision 16b9958](https://github.com/clemensv/avrotize/tree/16b995863f9aa162828d2332daca019defb6bd0e/test): `test/jsons/address.jsons`, `test/avsc/address.avsc`, and `test/struct/basic-types.struct.json`, respectively. The lab renames the first file to make its format explicit. The two data documents are lab examples.

This lab and the staged schemas are provided under the [Apache License 2.0](LICENSE), matching the source repository's license.
