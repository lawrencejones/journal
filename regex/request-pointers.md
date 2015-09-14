# Request Pointers

In reverting the revert of JSON validation pointers, we had decided to change the format
of...

```json
{
  "message": "can't be blank",
  "field": "organisation",
  "source": {
    "pointer": "/admin_access_tokens/organisation"
  }
}
```

...to a standard `request_pointer` field. Problem here is that we have 54 fixtures that
specify the pointer in the old way, with a mix of indentation and splitting the `source`
object over multiple lines.

And so here comes perl, that beautiful language...

```zsh
perl -i -p0e 's/"source"\s*:\s*{[\s\n]*"pointer"\s*:\s*"([^"]+)"[\s\n]*}(,?)/"request_pointer": "\1"/gm'
```

| Flag | Purpose |
|------|---------|
| `-p0` | Specifies no separator, so defaults to null [\0] |
| `-i` | Replaces file in place |
| `-e` | Evaluates the mini perl program |
