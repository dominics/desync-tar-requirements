## desync tar requirement example

This repository has a working example of what is required for `desync tar` to
be able to process a tar file. It contains:

- An example target file tree we want to archive; the two directories `a/` and `empty/`
- Two newline-separated file lists, suitable for passing to GNU tar's `--files-from` option
- Two bash scripts that run the broken and working minimal test cases (output shown below so you don't need to run them)

The test cases in each case are:

  - `broken.list`: which has two entries, but has missing intermediate directories implied by one of those paths; for those
    directories (`a/`, `a/b/`, `a/b/c/`), there's _no entry_ placed in the resulting tar.
  - `working.list`: which has two entries as well, but because it targets `a/`, the resulting tar *does* have the intermediate
    directories present

### Conclusion

It's not enough for the tar to have a common parent directory, nor enough to use desync's `--tar-add-root`.
It's also necessary that any intermediate directories appear in the tar before the files in those directories.

When passing a file list to GNU tar, this means:

- You can pass a list of directories, with `--recursive` (the default), and if this list of directories has no missing intermediate directories, it'll work (e.g. you pass `a/ b/`, or `a/ a/b/`)
- But if it has entries with missing intermediate directories (e.g. you pass `a/ b/c/`), then the resulting tar is _not suitable_ as a target for desync

Your options are:

- Include the intermediate directories in the tar
  - This can be complicated, because by adding the directories tar will, by default, recurse into them, potentially picking
    up sibling entries sibling entries that you didn't want included.)
    - e.g. consider if there was a file at `./a/b/foo` that you _didn't_ want in the resulting tar
    - You can `--exclude` specific directory contents you don't want, but this gets complicated/unwieldy if the file tree in question is large
  - Otherwise, you can pass the `--no-recursion` flag; this lets you do the recursion yourself, allowing you to control the contents of the tar much more exactly, because you can include _just_ the intermediate directories and not their contents
- Don't use CLI tar: build the tar archive yourself with your language's bindings, or try pax/cpio possibly.

## Example output

### working

```
$ ./working
./empty/
./empty/.gitignore
./a/
./a/b/
./a/b/c/
./a/b/c/d/
./a/b/c/d/1
./a/b/c/d/3
Result of desync run was: 0
.caidx produced was: -rw-r--r--  1 user  group   144B 14 Jan 10:31 working.caidx
```

### broken

```
$ ./broken
./empty/
./empty/.gitignore
./a/b/c/d/
./a/b/c/d/1
./a/b/c/d/3
Result of desync run was: 141
.caidx produced was: -rw-r--r--  1 user  group   144B 14 Jan 10:31 broken.caidx
```
