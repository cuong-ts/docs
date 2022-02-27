# Quick load .env file

## TL;DR

Load .env and ignore comment line start with `#`

```bash
$ export $(grep -v '^#' .env | xargs)
```

For unset all of the variables defined in the file , use this:

```bash
$ unset $(grep -v '^#' .env | sed -E 's/(.*)=.*/\1/' | xargs)
```



