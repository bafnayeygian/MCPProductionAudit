# Contributing

```bash
python -m pip install -e .
python -m unittest discover -s tests -v
python -m compileall -q src
```

Good contributions:

- new versioned audit rules
- HAR import
- OpenTelemetry import
- Tasks extension checks
- MCP Apps checks
- EMA/CIMD checks
- richer SARIF
- rule suppressions
- fleet-level reports

Rules should cite the MCP requirement or operational rationale in code comments or documentation.

Do not add rules that claim to verify behavior that cannot be observed from the input capture.
