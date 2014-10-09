#!/usr/bin/env ../bin/run-md-bash
# Extract required modules

For any java/coffee-script files that run with the CommonJS module pattern, it can
be useful to extract the arguments passed to `require` in order to examine what files
have been sourced by the project.

The following regex can extract the contents of the require string...

```bash
pattern="^(\s|[^#])*require\s*\(?['\"]\K([^'\"]+)"
```

This can be used in combination with a few tools to produce a list of all files that
are required from a node project.

Note that this is slow as hell, totally awful and doesn't support recursive searching
of paths. Just a simple proof of concept, that will take 20s to run!

Example run: `./node_requires.md ~/Projects/node-project/app "*.coffee"`

```bash

function listPaths {

  searchIn=$1
  filePattern=$2

  for f in $(find $searchIn -name "$filePattern")
  do
    basePath=$(dirname $f)
    for match in $(ggrep -Po $pattern $f)
    do
      resolveRequire "$basePath" "$match"
    done
  done

}

function resolveRequire {

  baseDir=$1
  requireArg=$2

  coffee -e "
  path = require('path')
  requireArg = \"$requireArg\"
  requireArg = path.resolve(\"$baseDir\", \"$match\") if /^\./.test(requireArg)
  console.log(require.resolve(requireArg))"

}


projectBase=$1
searchIn=$2
filePattern=$3

cd $projectBase
echo "Searching for files matching $filePattern in $searchIn..."
listPaths "$searchIn" "$filePattern" | sort | uniq

```
