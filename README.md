# Emlang Specification

![GitHub release](https://img.shields.io/github/v/release/emlang-project/spec)
![License](https://img.shields.io/github/license/emlang-project/spec)

This repository contains the formal specification for Emlang, a YAML-based DSL for describing systems with [Event Modeling](https://eventmodeling.org) patterns.

## Contents

- [SPEC.md](SPEC.md) — The specification
- [schema.json](schema.json) — JSON Schema for validation

## Schema Usage

Use `schema.json` to validate Emlang files in your editor or CI pipeline:

```yaml
# yaml-language-server: $schema=https://raw.githubusercontent.com/emlang-project/spec/main/schema.json
slices:
  MySlice:
    - c: DoSomething
    - e: SomethingDone
```

## Links

- [Emlang website](https://emlang-project.github.io) — Documentation, examples, and guides
- [Event Modeling](https://eventmodeling.org) — The methodology behind Emlang

## License

[MIT](LICENSE)
