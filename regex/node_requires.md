#!/usr/bin/env ../bin/run-md-bash
# Extract required modules

For any java/coffee-script files that run with the CommonJS module pattern, it can
be useful to extract the arguments passed to `require` in order to examine what files
have been sourced by the project.

The following regex can extract the contents of the require string...

```bash
pattern="^[^#]*require\s*\(?['\"]\K([^'\"]{1,80})"
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
      if [[ $match =~ ^\. ]];
      then
        realpath $basePath/$match | sed 's/\/\.\//\//'
      else
        echo $match
      fi
    done
  done

}

realpath() {
  [[ $1 = /* ]] && echo "$1" || echo "$PWD/${1#./}"
}

function resolveRequire {

  requireArg=$1

  coffee -e "
  console.log require.resolve(\"$requireArg\")"

}


projectBase=$1
searchIn=$2
filePattern=$3

cd $projectBase
echo "Searching for files matching $filePattern in $searchIn..."
for requireArg in $(listPaths "$searchIn" "$filePattern" | sort | uniq)
do
  resolveRequire $requireArg
done

```
