# Vault import, export secrets


```bash
./medusa export path --address="https://vault-url.com" --token="s.xxx" --format="yaml" > export.yaml
./medusa import path ./export.yaml --address="https://https://vault-url.com" --token="s.xxx"
```

REF: https://github.com/jonasvinther/medusa