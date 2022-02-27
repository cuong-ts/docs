# Convert service account json to single line

```bash
jq '.' service-account.json | jq -sR '.'
```