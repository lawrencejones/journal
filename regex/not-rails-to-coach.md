# NotRails -> Coach

It happened that I needed to extract a project from a rails repo, and rename along the
way. I found myself working with some peculiar search and replaces, so have recorded some
for posterity.

First, rename all requires from `require 'not_rails/*' -> 'coach/*'`...

```sh
ag -l "require.*not_rails" | xargs gsed -i 's/not_rails/coach/g'
ag -l "NotRails" | xargs gsed -i 's/NotRails/Coach/g'
```

Now, rename files...

```sh
zmv '(**)not_rails(**)' '$1coach$2'
```

Then remove all requires, as the code is now extracted as a gem and implicitly required.

```sh
ag -l "require.*coach" | xargs gsed -i '/require.*coach/d'
```

After committing, realised that some files now had blank lines where the requires had been
removed. So to neaten up...

```sh
find -name "*.rb" | xargs gsed -i '1{/^$/d}'  # remove all leading blank lines
```
