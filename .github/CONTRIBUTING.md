# Contributing

Thanks for your interest in improving this Dify DSL skill! This is a small,
documentation-focused project — most contributions are reference fixes, new
examples, or validator improvements.

## Reporting issues

Use the issue templates. For bugs, include the Dify version, the DSL `version`,
and the minimal YAML that reproduces the problem.

## Pull requests

1. If you change anything under `examples/` or `scripts/validate_dsl.py`, run
   the validator locally and make sure it passes:

   ```bash
   pip install -r requirements.txt
   python scripts/validate_dsl.py examples/*.yml
   ```

2. Keep the English and Chinese READMEs (`README.md` / `README_CN.md`) in
   parity — a change to one should be reflected in the other.

3. Respect the licensing boundary (see `NOTICE`): the MIT `LICENSE` covers only
   original contributions. Do not copy additional content from the
   All-Rights-Reserved upstream base without permission.

4. Be respectful and constructive.

## License posture

Original contributions you make here are licensed under the MIT License (see
`LICENSE`). The forked base remains All Rights Reserved; see `NOTICE` for the
full provenance.
