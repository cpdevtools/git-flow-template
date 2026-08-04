# Packages

Add workspace packages here. Each package should contain:

- `package.json` with `"version": "0.0.0-MAIN"` (release-group placeholder)
- `release-artifacts.yml` if the package publishes artifacts, e.g.:

```yaml
artifacts:
  - type: npm
    registries: ['github-npm']
```
