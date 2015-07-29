# Curl from stdin

Example of reposting a captured json payload to clippy...

```sh
curl \
  -X POST \
  -d @- \
  http://gc-clippy.herokuapp.com/webhooks/github <<< $(cat ~/pr_open_payload.json)
```

The `@` notation on a `-d` flag means take data from filehandle, `-` representing stdin.
