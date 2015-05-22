#!/usr/bin/env run-md-bash
# Search for string in git log

Assuming we want to find the commit that introduced/removed a particular string
from a repo, then the command can be run like so...

```bash
git log -S "commit that introduced" --source --all
```
