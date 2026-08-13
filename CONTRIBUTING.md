# Contributing

Corrections, worked examples, and reports of where this model breaks down are all welcome.

## The most useful contribution

If you implemented this and your operation does something the four axes could not express, open an issue describing it. Tell us what you were trying to represent and what you did instead. Model gaps are worth more than typo fixes.

## Adding a worked example

Examples live in `examples/` as JSON. Before opening a pull request:

1. Validate against the schema.

   ```bash
   npx ajv-cli@5 validate --spec=draft2020 -s schema/escalation-matrix.schema.json -d examples/your-example.json
   ```

2. Use a generic organization name. Do not include a real company name, real phone numbers, real customer names, or anything that identifies a specific site.
3. Explain the reasoning in `description` fields. An example that shows *what* without *why* is not much use to the next reader.

## Changing the schema

Schema changes need a rationale in the pull request describing the operational situation the current schema cannot represent. Enum additions are easier to accept than structural changes.

Breaking changes increment the major version. See section 8 of [`SPEC.md`](./SPEC.md).

## Style

Plain language. Spell out the consequence rather than the abstraction. If a rule exists because of something that went wrong in a real operation, say so.

## Not accepted

- Vendor specific configuration. The schema is platform neutral and stays that way.
- Marketing content, backlinks, or promotional additions to documentation.
- Examples containing real customer or site data.

## License

Contributions are accepted under the MIT license of this repository.
